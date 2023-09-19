---
title: "bRPC源码解析·ExecutionQueue"
linkTitle: "bRPC源码解析·ExecutionQueue"
weight: 3
date: 2023-08-18
---
(作者简介：KIDGINBROOK，在昆仑芯参与训练框架开发工作)

## 简介
ExecutionQueue是一个无锁的mpsc队列，主要逻辑其实就是brpc的client端发送数据时多线程向同一个fd写入数据，后来单独抽出来成为ExecutionQueue，基本功能如下：

- 异步有序执行: 任务在另外一个单独的线程中执行, 并且执行顺序严格和提交顺序一致，任务提交是wait-free的
- Multi Producer: 多个线程可以同时向一个ExecutionQueue提交任务
- 支持cancel一个已经提交的任务
- 支持stop
- 支持高优任务插队，且执行顺序也会严格按照提交顺序

## 实例
首先看下使用ExecutionQueue的例子，定义执行函数consume和执行任务DemoTask，consume函数中就是遍历所有task，然后执行每个task的run；然后定义一个ExecutionQueue，使用execution_queue_start启动，使用execution_queue_execute提交一个新的任务。
这里consume为什么使用for循环的原因后面会讲。

```c++
class DemoTask {
public:
    void run();
};

int consume(void* meta, TaskIterator<DemoTask*>& iter) {
    if (iter.is_queue_stopped()) {
        return 0;
    }   
    for (; iter; ++iter) {
        DemoTask* task = *iter;
        task->run();
    }   
    return 0;
}

ExecutionQueueId<DemoTask*> exe_queue;
int ret = execution_queue_start(&exe_queue, nullptr, consume, nullptr);

DemoTask* task = new DemoTask();
ret = execution_queue_execute(exe_queue, task);
```

## ExecutionQueue的创建
然后看下这个样例背后发生了什么，首先启动一个ExecutionQueue，调用链如下
```c++
template <typename T>
struct ExecutionQueueId {
    uint64_t value;
};

inline int execution_queue_start(
        ExecutionQueueId<T>* id, 
        const ExecutionQueueOptions* options,
        int (*execute)(void* meta, TaskIterator<T>&),
        void* meta) {
   return ExecutionQueue<T>::create(id, options, execute, meta);
}


```
id为64位类型, 相当于ExecutionQueue实例的一个弱引用, 可以wait-free的在O(1)时间内定位一个ExecutionQueue，option和meta我们传的都是null，所以先不关注，execute即刚刚定义的consume函数。

```c++
inline static int create(id_t* id, const ExecutionQueueOptions* options,
                             execute_func_t execute_func, void* meta) {
        return Base::create(&id->value, options, execute_task, 
                            clear_task_mem, meta, (void*)execute_func);
}

int ExecutionQueueBase::create(uint64_t* id, const ExecutionQueueOptions* options,
                               execute_func_t execute_func,
                               clear_task_mem clear_func,
                               void* meta, void* type_specific_function) {
    ...
    slot_id_t slot;
    ExecutionQueueBase* const m = butil::get_resource(&slot, Forbidden());
    if (BAIDU_LIKELY(m != NULL)) {
        m->_execute_func = execute_func;
        m->_clear_func = clear_func;
        m->_meta = meta;
        m->_type_specific_function = type_specific_function;
        CHECK(m->_head.load(butil::memory_order_relaxed) == NULL);
        CHECK_EQ(0, m->_high_priority_tasks.load(butil::memory_order_relaxed));
        ExecutionQueueOptions opt;
        if (options != NULL) {
            opt = *options;   
        }
        m->_options = opt;
        m->_stopped.store(false, butil::memory_order_relaxed);
        m->_this_id = make_id(
                _version_of_vref(m->_versioned_ref.fetch_add(
                                    1, butil::memory_order_release)), slot);
        *id = m->_this_id;
        get_execq_vars()->execq_count << 1;
        return 0;
    }
    return ENOMEM;
}

static int execute_task(void* meta, void* specific_function,
                            TaskIteratorBase& it) {
        execute_func_t f = (execute_func_t)specific_function;
        return f(meta, static_cast<iterator&>(it));
}
```
然后设置该ExecutionQueue的各个成员，注意这里有两个函数，一个是_type_specific_function即用户自定义的consume函数，另一个_execute_func为execute_task，其实就是调用用户自定义的consume函数；然后生成id返回。

## 执行一个task
然后看下执行一个任务，其中butil::add_const_reference<T>::type就是const T&，首先会通过id address到ExecutionQueue，然后调用execute，在示例的场景下option和handle均为null。
```c++
template <typename T>
inline int execution_queue_execute(ExecutionQueueId<T> id, 
                       typename butil::add_const_reference<T>::type task,
                       const TaskOptions* options,
                       TaskHandle* handle) {
    typename ExecutionQueue<T>::scoped_ptr_t 
        ptr = ExecutionQueue<T>::address(id);
    if (ptr != NULL) {
        return ptr->execute(task, options, handle);
    } else {
        return EINVAL;
    }   
}

int execute(typename butil::add_const_reference<T>::type task,
                const TaskOptions* options, TaskHandle* handle) {
        if (stopped()) {
            return EINVAL;
        }   
        TaskNode* node = allocate_node();
        if (BAIDU_UNLIKELY(node == NULL)) {
            return ENOMEM;
        }   
        void* const mem = allocator::allocate(node);
        if (BAIDU_UNLIKELY(!mem)) {
            return_task_node(node);
            return ENOMEM;
        }   
        new (mem) T(task);
        ...
}
```
首先申请一个TaskNode，TaskNode就是execution_queue中的节点，启动的任务task会存在节点TaskNode中，TaskNode主要结构如下，其中若task的结构小于56字节，则直接存储在static_task_mem中，否则存储在dynamic_task_mem中。
```c++
struct BAIDU_CACHELINE_ALIGNMENT TaskNode {
    ...
    butil::Mutex mutex;  // to guard version and status
    int64_t version;
    uint8_t status;
    bool stop_task;
    bool iterated;
    bool high_priority;
    bool in_place;
    TaskNode* next;
    ExecutionQueueBase* q;
    union {
        char static_task_mem[56];  // Make sizeof TaskNode exactly 128 bytes
        char* dynamic_task_mem;
    };
    ...
    static TaskNode* const UNCONNECTED;
};
```

ExecutionQueue中有一个结构为TaskAllocator<T>，会根据static_task_mem能否存下T来决定使用哪个特化版本，若small_object为true，则allocate直接返回static_task_mem，否则使用malloc来分配内存，在示例用法中，T为指针，即DemoTask*，所以使用的是static_task_mem。
```c++
template <size_t size, bool small_object> struct TaskAllocatorBase {
};

template <size_t size>
struct TaskAllocatorBase<size, true> {
    inline static void* allocate(TaskNode* node)
    { return node->static_task_mem; }
    inline static void* get_allocated_mem(TaskNode* node)
    { return node->static_task_mem; }
    inline static void deallocate(TaskNode*) {}
};

template<size_t size>
struct TaskAllocatorBase<size, false> {
    inline static void* allocate(TaskNode* node) {
        node->dynamic_task_mem = (char*)malloc(size);
        return node->dynamic_task_mem;
    }

    inline static void* get_allocated_mem(TaskNode* node)
    { return node->dynamic_task_mem; }

    inline static void deallocate(TaskNode* node) {
        free(node->dynamic_task_mem);
    }
};

template <typename T>
struct TaskAllocator : public TaskAllocatorBase<
               sizeof(T), sizeof(T) <= sizeof(TaskNode().static_task_mem)>
{};
```

然后调用allocator的allocate，如上所述，这里直接返回node的static_task_mem，然后在这块内存上调用placement_new，所以DemoTask*便赋值到了static_task_mem上。
然后设置优先级等，因为传入的TaskOptions为null，所以不是高优，然后执行start_execute。
```c++
int execute(typename butil::add_const_reference<T>::type task,
                const TaskOptions* options, TaskHandle* handle) {
        ...
        node->stop_task = false;
        TaskOptions opt;
        if (options) {
            opt = *options;
        }
        node->high_priority = opt.high_priority;
        node->in_place = opt.in_place_if_possible;
        if (handle) {
            handle->node = node;
            handle->version = node->version;
        }
        start_execute(node);
        return 0;
    }
```

首先设置node的next为UNCONNECTED，UNCONNECTED为-1，表示当前节点还没有链入到链表中，_head为当前execution_queue的链表头节点，然后原子指令exchange后，链表头节点成为node，node的next为UNCONNECTED，此时链表是断链的，prev_head为链表之前的头结点，如果prev_head不为null，那么说明之前已经启动过消费bthread了，因此只需设置头节点的next为prev_head，然后直接return即可，此时node才真正的链入了链表；如果prev_head为null，则需要启动消费bthread。这里exchange使用release，是为了让消费bthread看到对node的修改。
```c++
void ExecutionQueueBase::start_execute(TaskNode* node) {
    node->next = TaskNode::UNCONNECTED;
    node->status = UNEXECUTED;
    node->iterated = false;
    if (node->high_priority) {
        // Add _high_priority_tasks before pushing this task into queue to
        // make sure that _execute_tasks sees the newest number when this 
        // task is in the queue. Although there might be some useless for 
        // loops in _execute_tasks if this thread is scheduled out at this 
        // point, we think it's just fine.
        _high_priority_tasks.fetch_add(1, butil::memory_order_relaxed);
    }
    TaskNode* const prev_head = _head.exchange(node, butil::memory_order_release);
    if (prev_head != NULL) {
        node->next = prev_head;
        return;
    }
    ...
}
```
然后设置next为null，因为默认情况下in_place为false，executor为null，所以会直接启动一个bthread后台执行_execute_tasks。如果使用了in_place则会立即执行_execute_tasks，在无竞争的场景中可以省去一次线程调度和cache同步的开销，不过谨慎使用，需要检查会不会发生死锁等情况。

然后结合示意图看下之后会发生什么，假设此时时间点t1，现在队列里只有一个节点
![图 1](/images/docs/execution_queue_1.png)

_execute_tasks中设置cur_tail为null，然后进入for循环，初始时head中的iterated为false，也没有高优任务，因此直接执行m->_execute()
```c++
void* ExecutionQueueBase::_execute_tasks(void* arg) {
    ExecutionQueueVars* vars = get_execq_vars();
    TaskNode* head = (TaskNode*)arg;
    ExecutionQueueBase* m = (ExecutionQueueBase*)head->q;
    TaskNode* cur_tail = NULL;
    bool destroy_queue = false;
    for (;;) {
        if (head->iterated) {
            CHECK(head->next != NULL);
            TaskNode* saved_head = head;
            head = head->next;
            m->return_task_node(saved_head);
        }
        int rc = 0;
        if (m->_high_priority_tasks.load(butil::memory_order_relaxed) > 0) {
            int nexecuted = 0;
            // Don't care the return value
            rc = m->_execute(head, true, &nexecuted);
            m->_high_priority_tasks.fetch_sub(
                    nexecuted, butil::memory_order_relaxed);
            if (nexecuted == 0) {
                // Some high_priority tasks are not in queue
                sched_yield();
            }
        } else {
            rc = m->_execute(head, false, NULL);
        }
        ...
    }
    vars->execq_active_count << -1;
    return NULL;
}
```
_execute中会生成迭代器，然后调用_execute_func，这个上文有提到，就是执行用户指定的执行函数，即示例中的consume
```c++
int ExecutionQueueBase::_execute(TaskNode* head, bool high_priority, int* niterated) {
    ...
    TaskIteratorBase iter(head, this, false, high_priority);
    if (iter) {
        _execute_func(_meta, _type_specific_function, iter);
    }
    ...
}
```

## 迭代器
然后看下TaskIteratorBase，主要成员为_cur_node，表示当前遍历到了哪个节点；_head表示当前执行队列的head；_high_priorty表示该iterator的优先级，而且低优迭代器只会遍历低优任务，高优迭代器只会遍历高优任务。

```c++
class TaskIteratorBase {
    ...
    TaskNode*               _cur_node;
    TaskNode*               _head;
    ExecutionQueueBase*     _q;
    bool                    _is_stopped;
    bool                    _high_priority;
    bool                    _should_break;
    int                     _num_iterated;
};

template <typename T>
class TaskIterator : public TaskIteratorBase {
    TaskIterator();
public:
    typedef T*          pointer;
    typedef T&          reference;

    reference operator*() const;
    pointer operator->() const { return &(operator*()); }
    TaskIterator& operator++();
    void operator++(int);
};
```
在demo的consume函数中，通过对TaskIter解引用得到了DemoTask*，这块逻辑如下，上文中说到 DemoTask* 存在TaskNode的static_task_mem中，这里get_allocated_mem则是直接返回static_task_mem，因此便拿到了加到队列中的DemoTask*。
```c++
inline typename TaskIterator<T>::reference
TaskIterator<T>::operator*() const {
    T* const ptr = (T* const)TaskAllocator<T>::get_allocated_mem(cur_node());
    return *ptr;
}
```
然后看下自增操作，主要逻辑就是将_cur_node挪到下一个和当前iterator优先级一致的，并且没有被遍历过的节点。
具体的，先判断当前节点是否遍历过，在示例中节点1的iterated为false，所以直接往下进入while循环，因为当前生成的是低优先级的iter，node也是低优先级，所以进入if，在第二个if中，iterated为false，peek_to_execute是判断当前节点状态是否为UNEXECUTED，因此也进入第二个if，将当前节点iterated置为true直接返回。注意上面TaskIter的构造函数会执行一次operator++，所以就会将节点1的iterated置为true，且_cur_node指向1。
```c++
void TaskIteratorBase::operator++() {
    if (!(*this)) {
        return;
    }
    if (_cur_node->iterated) {
        _cur_node = _cur_node->next;
    }
    if (should_break_for_high_priority_tasks()) {
        return;
    }  // else the next high_priority_task would be delayed for at most one task

    while (_cur_node && !_cur_node->stop_task) {
        if (_high_priority == _cur_node->high_priority) {
            if (!_cur_node->iterated && _cur_node->peek_to_execute()) {
                ++_num_iterated;
                _cur_node->iterated = true;
                return;
            }
            _num_iterated += !_cur_node->iterated;
            _cur_node->iterated = true;
        }
        _cur_node = _cur_node->next;
    }
    return;
}
```

在Iter的析构中，会将从_head到_cur_node区间所有相同优先级节点设置为EXECUTED。
```c++
TaskIteratorBase::~TaskIteratorBase() {
    // Set the iterated tasks as EXECUTED here instead of waiting them to be
    // returned in _start_execute as the high_priority_task might be in the
    // middle of the linked list and is not going to be returned soon
    if (_is_stopped) {
        return;
    }
    while (_head != _cur_node) {
        if (_head->iterated && _head->high_priority == _high_priority) {
            _head->set_executed();
        }
        _head = _head->next;
    }
    if (_should_break && _cur_node != NULL 
            && _cur_node->high_priority == _high_priority && _cur_node->iterated) {
        _cur_node->set_executed();
    }
}
```

## 队列调整

然后回到_execute_tasks函数的_execute之后，head即节点1被执行结束了，head的next为null，cur_tail为null，所以cur_tail被置为了head。_execute结束后，生成的Iter被析构，如上所述，Iter析构会设置1的状态为EXECUTED。
```c++
void* ExecutionQueueBase::_execute_tasks(void* arg) {
    ExecutionQueueVars* vars = get_execq_vars();
    TaskNode* head = (TaskNode*)arg;
    ExecutionQueueBase* m = (ExecutionQueueBase*)head->q;
    TaskNode* cur_tail = NULL;
    bool destroy_queue = false;
    for (;;) {
        ...
        // Release TaskNode until uniterated task or last task
        while (head->next != NULL && head->iterated) {
            TaskNode* saved_head = head;
            head = head->next;
            m->return_task_node(saved_head);
        }
        if (cur_tail == NULL) {
            for (cur_tail = head; cur_tail->next != NULL; 
                    cur_tail = cur_tail->next) {}
        }
        // break when no more tasks and head has been executed
        if (!m->_more_tasks(cur_tail, &cur_tail, !head->iterated)) {
            CHECK_EQ(cur_tail, head);
            CHECK(head->iterated);
            m->return_task_node(head);
            break;
        }
    }
    ...
    return NULL;
}
```
然后执行_more_tasks，假设此时为t2，又入队了两个新的节点，如下图所示

![图 2](/images/docs/execution_queue_2.png)

此时old_head指向1，*new_tail为1，new_head指向1，desired为null，return_when_no_more为false，然后通过cas操作，如果_head还是指向1，说明队列中没有新加的节点，那么_head被置为null，返回false，这里使用acquire是和入队进行配对，保证看到对node的修改；在上图这个例子中，新加了2,3两个节点，此时_head指向3，所以new_head被设置为3。
```c++
inline bool ExecutionQueueBase::_more_tasks(
        TaskNode* old_head, TaskNode** new_tail, 
        bool has_uniterated) {

    CHECK(old_head->next == NULL);
    // Try to set _head to NULL to mark that the execute is done.
    TaskNode* new_head = old_head;
    TaskNode* desired = NULL;
    bool return_when_no_more = false;
    if (has_uniterated) {
        desired = old_head;
        return_when_no_more = true;
    }
    if (_head.compare_exchange_strong(
                new_head, desired, butil::memory_order_acquire)) {
        // No one added new tasks.
        return return_when_no_more;
    }
    ...
}
```

假设执行到此时为t3，又新加了两个节点，如下图
![图 3](/images/docs/execution_queue_3.png)

然后开始反转链表的new_head到old_head区间，*new_tail指向3，注意在反转前会判断new_head的next是否为UNCONNECTED，如上文所述，在将一个节点加入到链表的过程中有一段时间是断链的，这种情况下就调用sched_yield将执行权从当前bthread切换到其他bthread，直到链表链接起来。

```c++
inline bool ExecutionQueueBase::_more_tasks(
        TaskNode* old_head, TaskNode** new_tail, 
        bool has_uniterated) {

    ...
    TaskNode* tail = NULL;
    if (new_tail) {
        *new_tail = new_head;
    }
    TaskNode* p = new_head;
    do {
        while (p->next == TaskNode::UNCONNECTED) {
            // TODO(gejun): elaborate this
            sched_yield();
        }
        TaskNode* const saved_next = p->next;
        p->next = tail;
        tail = p;
        p = saved_next;
        CHECK(p != NULL);
    } while (p != old_head);

    // Link old list with new list.
    old_head->next = tail;
    return true;
}
```

此时整个执行队列如下图所示，此时节点1已被执行过，但仍在队列中，回到上文的_execute_tasks，下次循环时，首先head是否被遍历过，如果遍历过，则将该节点释放；
![图 4](/images/docs/execution_queue_4.png)

然后执行_execute，_execute中会执行2,3，接着释放节点2，继续链表反转，此时队列结构如下图，后面的过程则和上文类似不再赘述。
![图 5](/images/docs/execution_queue_5.png)

然后看下本文最开始的问题，consume中为什么要写成for循环的方式，只run一个是否可以，这里其实是为了性能考虑，只run一个也是可以的，不过run完之后要执行more_task等等一系列操作，而写成for循环方式的话只有run到null之后才会执行more_task等操作，所以性能会好一些。

## 高优任务
最后看下当提交一个高优任务时会发生什么

假设t3时刻加入的5是high_priority，那么执行完_more_task第二次循环时，摘掉已执行过的节点1之后的队列情况如下图
![图 6](/images/docs/execution_queue_6.png)

因为在start_execute的时候，_high_priority_tasks会加一，所以在_execute_tasks的第二次循环中，会发现_high_priority_tasks不为0，执行_execute的时候会将high_priority置为true，_execute所做的事情为生成一个iter，然后执行用户自定义函数consume，因为此时是high_priority，因此生成的iterator也是high。
```c++

void* ExecutionQueueBase::_execute_tasks(void* arg) {
    ...
    for (;;) {
        ...
        if (m->_high_priority_tasks.load(butil::memory_order_relaxed) > 0) {
            int nexecuted = 0;
            // Don't care the return value
            rc = m->_execute(head, true, &nexecuted);
            m->_high_priority_tasks.fetch_sub(
                    nexecuted, butil::memory_order_relaxed);
            if (nexecuted == 0) {
                // Some high_priority tasks are not in queue
                sched_yield();
            }
        } else {
            rc = m->_execute(head, false, NULL);
        }
        ...
    }
    vars->execq_active_count << -1;
    return NULL;
}
```
在_execute中生成iter，iter构造函数中执行++，会遍历2，3，null，因为iter为null，所以_execute直接返回，并且如上文所述该iter析构并不会设置低优task的执行状态；此时nexecuted为0，所以调用sched_yield切出去一会，这里的原因其实是因为在新增高优任务的时候是先增加高优任务的计数器，然后再将高优任务加到队列中，所以如果遍历了一遍队列发现没高优任务就切出去，等待高优任务的入队。


接着调用_more_task，cur_tail指向节点3，经过反转链表后如下图所示
![图 7](/images/docs/execution_queue_7.png)
重新执行_execute，生成高优iter，执行++，遍历到5的时候设置5的iterated为true，然后return，调用用户自定义执行函数consume执行了5这个高优task，consume中的++又会跳过所有低优任务到达null，此时再执行_more_task时，因为head节点iterated为false，所以has_uniterated为true，此时desired指向5，return_when_no_more为true，此时再经过cas时会直接返回true；下一轮循环中会生成低优迭代器执行队列中2,3,4，并在回收内存时将5的内存一并回收。
```c++
inline bool ExecutionQueueBase::_more_tasks(
        TaskNode* old_head, TaskNode** new_tail, 
        bool has_uniterated) {

    CHECK(old_head->next == NULL);
    // Try to set _head to NULL to mark that the execute is done.
    TaskNode* new_head = old_head;
    TaskNode* desired = NULL;
    bool return_when_no_more = false;
    if (has_uniterated) {
        desired = old_head;
        return_when_no_more = true;
    }
    if (_head.compare_exchange_strong(
                new_head, desired, butil::memory_order_acquire)) {
        // No one added new tasks.
        return return_when_no_more;
    }
    ...
}
```
