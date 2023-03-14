## 背景
由于brpc中引入了bthread，如果在bthread中使用了mutex等pthread同步原语，那么将会挂起当前pthread，导致该bthread_worker（pthread）将无法执行其他bthread，因此类似pthread和futex的关系，brpc引入butex来实现bthread粒度的挂起和唤醒以提高性能。
## 实现思路
类比futex，futex主要由两部分组成，一个是用户态中的标记（是一个int），另一个是内核态中的等待队列，当执行futex_wait的时候，首先尝试原子修改标记，如果没有竞争的话，整个流程将不会进入内核态；如果有其他线程占用了futex，那么会将当前线程加入到内核等待队列。
## 实现细节
### 主要数据结构
#### Butex
``` c++
struct BAIDU_CACHELINE_ALIGNMENT Butex {
    Butex() {}
    ~Butex() {}

    butil::atomic<int> value;
    ButexWaiterList waiters;
    internal::FastPthreadMutex waiter_lock;
};
```
Butex的实现类似futex。value即上述的标记，表示当前Butex的状态，ButexWaiterList是一个链表，保存了在该butex上挂起的bthread或pthread。
waiter_lock即上述的锁。因为butex_wait时会比较atomic是否为expect_value，如果相等，那么将会挂起当前bthread至等待队列。如果判断是否相等和挂起这两个操作不在同一个临界区，那么有可能在判断之后但是挂起前已经执行过了butex_wake，将出现信号丢失问题，因此这里需要使用互斥锁。
#### ButexWaiter
```c++
struct ButexWaiter : public butil::LinkNode<ButexWaiter> {
    // tids of pthreads are 0
    bthread_t tid;

    // Erasing node from middle of LinkedList is thread-unsafe, we need
    // to hold its container's lock.
    butil::atomic<Butex*> container;
};

typedef butil::LinkedList<ButexWaiter> ButexWaiterList;

struct ButexBthreadWaiter : public ButexWaiter {
    TaskMeta* task_meta;
    TimerThread::TaskId sleep_id;
    WaiterState waiter_state;
    int expected_value;
    Butex* initial_butex;
    TaskControl* control;
    const timespec* abstime;
};

// pthread_task or main_task allocates this structure on stack and queue it
// in Butex::waiters.
struct ButexPthreadWaiter : public ButexWaiter {
    butil::atomic<int> sig;
};
```
Butex的等待队列为一个链表，链表的元素为ButexWaiter，由于Butex需要兼容pthread，因此有ButexBthreadWaiter和ButexPthreadWaiter两种waiter，后续通过bthread的逻辑介绍Butex。
#### FastPthreadMutex
如果不开启编译选项BTHREAD_USE_FAST_PTHREAD_MUTEX，那么FastPthreadMutex就是pthread_mutex_t，否则为：
```c++
class FastPthreadMutex {
public:
    FastPthreadMutex() : _futex(0) {}
    ~FastPthreadMutex() {}
    void lock();
    void unlock();
    bool try_lock();
private:
    DISALLOW_COPY_AND_ASSIGN(FastPthreadMutex);
    int lock_contended();
    unsigned _futex;
};
```
这里基于futex实现了互斥锁，_futex表示锁的状态，从这里可以看到futex的用法，通过Butex实现bthread_mutex也是类似的逻辑。
```c++
struct MutexInternal {
    butil::static_atomic<unsigned char> locked;
    butil::static_atomic<unsigned char> contended;
    unsigned short padding;
};

const MutexInternal MUTEX_CONTENDED_RAW = {{1},{1},0};
const MutexInternal MUTEX_LOCKED_RAW = {{1},{0},0};
// Define as macros rather than constants which can't be put in read-only
// section and affected by initialization-order fiasco.
#define BTHREAD_MUTEX_CONTENDED (*(const unsigned*)&bthread::MUTEX_CONTENDED_RAW)
#define BTHREAD_MUTEX_LOCKED (*(const unsigned*)&bthread::MUTEX_LOCKED_RAW)
```
FastPthreadMutex用MutexInternal表示一个锁的状态，如果锁没有人占用，那么locked和contended均为0，如果有一个线程占用了这个锁，那么locked为1，contended为0，如果这个时候又有线程来尝试占用锁，那么locked为1，contended为1，后边来的线程都会在这个锁上wait。
```c++
void FastPthreadMutex::lock() {
    bthread::MutexInternal* split = (bthread::MutexInternal*)&_futex;
    if (split->locked.exchange(1, butil::memory_order_acquire)) {
        (void)lock_contended();
    }
}
```
lock的过程首先尝试首先尝试修改locked这个atomic，如果发现锁没被占用（locked为0），那么直接返回，否则调用lock_contended方法。注意这里使用了memory_order_acquire的memory order，和unlock中的release形成syncwith关系，保证了当前线程获得锁之后能看到上个线程在释放锁之前对内存的修改。
```c++
int FastPthreadMutex::lock_contended() {
    butil::atomic<unsigned>* whole = (butil::atomic<unsigned>*)&_futex;
    while (whole->exchange(BTHREAD_MUTEX_CONTENDED) & BTHREAD_MUTEX_LOCKED) {
        if (futex_wait_private(whole, BTHREAD_MUTEX_CONTENDED, NULL) < 0
            && errno != EWOULDBLOCK) {
            return errno;
        }
    }
    return 0;
}
```
这里将锁的状态MutexInternal原子修改为BTHREAD_MUTEX_CONTENDED，如果锁仍被占用着，那么通过系统调用futex_wait_private将当前线程挂起到whole这个atomic对应的队列中。
```c++
void FastPthreadMutex::unlock() {
    butil::atomic<unsigned>* whole = (butil::atomic<unsigned>*)&_futex;
    const unsigned prev = whole->exchange(0, butil::memory_order_release);
    // CAUTION: the mutex may be destroyed, check comments before butex_create
    if (prev != BTHREAD_MUTEX_LOCKED) {
        futex_wake_private(whole, 1);
    }
}
```
unlock方法将锁的状态原子改为0，如果之前的状态不是BTHREAD_MUTEX_LOCKED，那说明有线程阻塞在了这个锁上，因此需要通过futex_wake_private唤醒whole对应等待队列中的一个pthread，如上所述，这里使用release保证当前线程对内存的修改能被后续竞争到锁的线程看到。
### 主要接口
#### butex_create_checked
template <typename T> T* butex_create_checked() {
    BAIDU_CASSERT(sizeof(T) == sizeof(int), sizeof_T_must_equal_int);
    return static_cast<T*>(butex_create());
}

void* butex_create() {
    Butex* b = butil::get_object<Butex>();
    if (b) {
        return &b->value;
    }   
    return NULL;
}
butex_create_checked通过get_object拿到了一个Butex，然后将Butex中的value返回给用户，用户通过value进行操作
#### butex_wait
```c++
int butex_wait(void* arg, int expected_value, const timespec* abstime) {
    Butex* b = container_of(static_cast<butil::atomic<int>*>(arg), Butex, value);
    if (b->value.load(butil::memory_order_relaxed) != expected_value) {
        errno = EWOULDBLOCK;
        // Sometimes we may take actions immediately after unmatched butex,
        // this fence makes sure that we see changes before changing butex.
        butil::atomic_thread_fence(butil::memory_order_acquire);
        return -1;
    }
   ...
}
```
arg即butex_create_checked中返回的Butex中的value，通过value获取到对应的Butex，判断value是否等于expected_value，如果不等于则直接返回。
```c++
int butex_wait(void* arg, int expected_value, const timespec* abstime) {
    ...
    TaskGroup* g = tls_task_group;
    if (NULL == g || g->is_current_pthread_task()) {
        return butex_wait_from_pthread(g, b, expected_value, abstime);
    }
    ButexBthreadWaiter bbw;
    // tid is 0 iff the thread is non-bthread
    bbw.tid = g->current_tid();
    bbw.container.store(NULL, butil::memory_order_relaxed);
    bbw.task_meta = g->current_task();
    bbw.sleep_id = 0;
    bbw.waiter_state = WAITER_STATE_READY;
    bbw.expected_value = expected_value;
    bbw.initial_butex = b;
    bbw.control = g->control();
    bbw.abstime = abstime;

    ...
}
```
如果value和expected_value，则尝试挂起当前bthread或者pthread，bthread是由bthread_worker进行调度执行的，bthread_worker所在的pthread有设置线程局部变量tls_task_group，所以如果tls_task_group为NULL，则当前是bthread，否则为pthread，假设当前是bthread，那么创建一个ButexBthreadWaiter，设置其中的变量，tid为当前bthread的id，initial_butex为当前Butex。
```c++
int butex_wait(void* arg, int expected_value, const timespec* abstime) {
    ...
    g->set_remained(wait_for_butex, &bbw);
    TaskGroup::sched(&g);
    ...
}
```
然后通过set_reamined设置remain，remain即bthread_worker执行下一个bthread之前需要做的事情，设置完成后执行sched切换到其他bthread上继续执行。然后看下remain，即wait_for_butex中做的工作。
```c++
static void wait_for_butex(void* arg) {
    ButexBthreadWaiter* const bw = static_cast<ButexBthreadWaiter*>(arg);
    Butex* const b = bw->initial_butex;
    
    {
        BAIDU_SCOPED_LOCK(b->waiter_lock);
        if (b->value.load(butil::memory_order_relaxed) != bw->expected_value) {
            bw->waiter_state = WAITER_STATE_UNMATCHEDVALUE;
        } else if (bw->waiter_state == WAITER_STATE_READY/*1*/ &&
                   !bw->task_meta->interrupted) {
            b->waiters.Append(bw);
            bw->container.store(b, butil::memory_order_relaxed);
            if (bw->abstime != NULL) {
                bw->sleep_id = get_global_timer_thread()->schedule(
                    erase_from_butex_and_wakeup, bw, *bw->abstime);
                if (!bw->sleep_id) {  // TimerThread stopped.
                    errno = ESTOP;
                    erase_from_butex_and_wakeup(bw);
                }
            }
            return;
        }
    }
      
    tls_task_group->ready_to_run(bw->tid);
}
```
获取到Butex，由于判断是否相等和挂起这两个操作需要在同一个临界区，所以需要上锁，如果Butex中的value不等于expected_value，那么通过ready_to_run将执行butex_wait的bthread重新假如到执行队列等待调度。否则将bw加入到Butex的等待队列中。
#### butex_wake
```c++
int butex_wake(void* arg, bool nosignal) {
    Butex* b = container_of(static_cast<butil::atomic<int>*>(arg), Butex, value);
    ButexWaiter* front = NULL;
    {
        BAIDU_SCOPED_LOCK(b->waiter_lock);
        if (b->waiters.empty()) {
            return 0;
        }
        front = b->waiters.head()->value();
        front->RemoveFromList();
        front->container.store(NULL, butil::memory_order_relaxed);
    }
    if (front->tid == 0) {
        wakeup_pthread(static_cast<ButexPthreadWaiter*>(front));
        return 1;
    }
    ButexBthreadWaiter* bbw = static_cast<ButexBthreadWaiter*>(front);
    unsleep_if_necessary(bbw, get_global_timer_thread());
    TaskGroup* g = get_task_group(bbw->control, nosignal);
    if (g == tls_task_group) {
        run_in_local_task_group(g, bbw->tid, nosignal);
    } else {
        g->ready_to_run_remote(bbw->tid, nosignal);
    }
    return 1;
}
```
在该butex的等待队列中唤醒第一个ButexWaiter front，将front从waiters链表里删除，如果front是pthread（tid == 0的位pthread），那么调用wakeup_pthread。
如果front为bthread，如果执行butex_wake的是bthread_worker，即另一个bthread执行的，那么直接让出当前bthread，直接执行被唤醒的bthread，如果执行butex_wake的pthread不是bthread_worker，那么就把当前bthread加入到某个task_group的remote_rq中等待调度执行。
