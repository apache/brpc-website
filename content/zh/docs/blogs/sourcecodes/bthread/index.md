---
title: "bRPC源码解析·bthread机制"
linkTitle: "bRPC源码解析·bthread机制"
weight: 2
date: 2022-06-16
---
## bthread简介
官方文档：https://brpc.apache.org/zh/docs/bthread/bthread/  

bthread是bRPC使用的M:N线程库，类似协程，即用户态线程，也因此bthread的切换不会陷入内核，不会进行一系列内存同步等耗时操作，从bthread_benchmark中可以看到bthread的创建时间和调度时间相较pthread有着数量级的提升，将大量的bthread映射至少量的内核线程pthread上执行，降低内核上下文切换开销，在充分利用多核的同时，具有更好的cache locality。

为了实现协程需要协程栈、协程的初始化，以及协程间的切换，接下来逐一分析一下这几个过程。在分析之前需要首先补充一点汇编知识以方便我们阅读其中的汇编代码。

## 基础汇编知识
首先语法习惯，代码中的是AT&T风格的汇编语言，gdb看反汇编默认的风格也是AT&T。

### AT&T风格的汇编语言
* 立即数：$ 开头
* 寄存器：% 开头
* 取地址里面的值：偏移量(%寄存器）
* 整形操作通用后缀：[B] Byte、[W] Word 2Byte、[L] Long 4Byte、[Q] QuadWord 8Byte
* 浮点操作通用后缀：[S] Singled 4、[D] Double 8、[T] Extended 16（修饰精度: precision）

### 函数的调用约定
* 整型参数依次存放在 %rdi、%rsi、%rdx、%rcx、%8、%9
* 浮点参数依次存放在%xmm0 - %xmm7中
* 寄存器不够用时，参数放到栈中
* 被调用的函数可以使用任何寄存器，但它必须保证%rbx、%rbp、%rsp和%12-%15恢复到原来的值
* 返回值存放在%rax中

### 调用函数前
* 调用方要将参数放到寄存器中
* 然后把%10、%11的值保存到栈中
* 然后调用call 跳转到函数执行
* 返回后，恢复%10、%11
* 从%eax中取出返回值

### x86_64含有16个64为整数寄存器
* %rsi、%rdi 用于字符串处理
* %rsp、%rbp 栈相关，栈从高地址到低地址 %rsp-->栈顶，push和pop会改变 %rbp-->栈基址
* %8 ~ %15

|寄存器|解释|
|:---|:---|
|// 创建部分||
|%rax|临时寄存器；参数可变时传递关于 SSE 寄存器用量的信息；第 1 个返回值寄存器|
|%rdi|用来给函数传递第 1 个参数|
|%rsi|用来给函数传递第 2 个参数|
|%rdx|用来给函数传递第 3 个整数参数|
|%rcx|用来给函数传递第 4个整数参数|
|%rsp|堆栈顶指针|
|%rbp|堆栈基指针|
|%rip|指令寄存器|
|// 切换部分||
|%rbx|被调者保存的寄存器；或用作基指针|
|%r12-%r15|被调者保存的寄存器|

## 协程栈的结构
### 协程栈
首先看下协程栈的结构：context指向协程栈顶，stacktype表示栈的类型(大小)，storage为栈空间。
``` c++
/* bthread/bthread/stack.h */
struct ContextualStack {
    bthread_fcontext_t context;
    StackType stacktype;
    StackStorage storage;
};

enum StackType {
    STACK_TYPE_MAIN = 0,
    STACK_TYPE_PTHREAD = BTHREAD_STACKTYPE_PTHREAD,
    STACK_TYPE_SMALL = BTHREAD_STACKTYPE_SMALL,
    STACK_TYPE_NORMAL = BTHREAD_STACKTYPE_NORMAL,
    STACK_TYPE_LARGE = BTHREAD_STACKTYPE_LARGE
};

struct StackStorage {
    int stacksize;
    int guardsize;
    // Assume stack grows upwards.
    // http://www.boost.org/doc/libs/1_55_0/libs/context/doc/html/context/stack.html
    void* bottom;
    unsigned valgrind_stack_id;

    // Clears all members.
    void zeroize() {
        stacksize = 0;
        guardsize = 0;
        bottom = NULL;
        valgrind_stack_id = 0;
    }
};
```

### 协程的初始化
栈分配时（allocate_stack_storage(StackStorage* s, int stacksize_in, int guardsize_in)）会通过mmap匿名映射一段空间，然后将高地址位赋值给bottom。
``` c++
/* bthread/bthread/stack_inl.h */
template <typename StackClass> struct StackFactory {
    struct Wrapper : public ContextualStack {
        explicit Wrapper(void (*entry)(intptr_t)) {
            if (allocate_stack_storage(&storage, *StackClass::stack_size_flag,
                                       FLAGS_guard_page_size) != 0) {
                storage.zeroize();
                context = NULL;
                return;
            }
```
然后创建bthread协程栈，返回值为协程栈顶context，函数入参分别为协程栈底，栈大小，以及这个bthread要执行的函数entry。
``` c++
            context = bthread_make_fcontext(storage.bottom, storage.stacksize, entry);
            stacktype = (StackType)StackClass::stacktype;
        }
        ...
    };
    ...
};
```
``` c++
#if defined(BTHREAD_CONTEXT_PLATFORM_linux_x86_64) && defined(BTHREAD_CONTEXT_COMPILER_gcc)
__asm (
".text\n"
".globl bthread_make_fcontext\n"
".type bthread_make_fcontext,@function\n"
".align 16\n"
"bthread_make_fcontext:\n"
"    movq  %rdi, %rax\n"
"    andq  $-16, %rax\n"
"    leaq  -0x48(%rax), %rax\n"
"    movq  %rdx, 0x38(%rax)\n"
"    stmxcsr  (%rax)\n"
"    fnstcw   0x4(%rax)\n"
"    leaq  finish(%rip), %rcx\n"
"    movq  %rcx, 0x40(%rax)\n"
"    ret \n"
"finish:\n"
"    xorq  %rdi, %rdi\n"
"    call  _exit@PLT\n"
"    hlt\n"
".size bthread_make_fcontext,.-bthread_make_fcontext\n"
".section .note.GNU-stack,\"\",%progbits\n"
);

#endif
```

#### 代码中的汇编指令解释
MXCSR状态管理指令（State Management Instructions），LDMXCSR与STMXCSR，用于控制MXCSR寄存器状态。
* LDMXCSR指令从存储器中加载MXCSR寄存器状态
* STMXCSR指令将MXCSR寄存器状态保存到存储器中

|汇编指令|解释|
|:---|:---|
|movq  %rdi, %rax|"movq" is a move of a quadword (64-bit value)，copy %src %dst，将第一个参数‘storage.bottom’拷贝到%rax寄存器|
|andq  $-16, %rax|与&操作，将%rax的值，即高位地址‘bottom’ & -16（补码:0xfffffff0），将地址取为16的整数倍，进行16字节对齐。|
|leaq  -0x48(%rax), %rax|leaq 取64位地址指令，将%rax中保存的高位地址向下偏移72个字节后，再保存到%rax寄存器，即留出 0x48 个字节用于存放上下文 Context Data|
|movq  %rdx, 0x38(%rax)|将rdx保存至rax + 56，rdx为第三个参数，即函数fn的入口地址entry|
|stmxcsr  (%rax)|存储 MMX 控制字和状态字，将MXCSR寄存器状态保存到rax所在位置|
|fnstcw   0x4(%rax)|存储 x87 控制字，将 FPU 控制字的当前值存储到%rax + 4|
|leaq  finish(%rip), %rcx|计算finish标志的绝对地址，将其保存至%rcx寄存器中|
|movq  %rcx, 0x40(%rax)|将rcx保存到rax + 64，finish刚好位于启动函数上方 ---> 启动函数执行完以后就会执行finish处的代码，而finish会call _exit结束进程。|
|ret|rax就是上述的基点（栈顶），返回的类型为fcontext_t|
|finish:||
|xorq  %rdi, %rdi|退出码是0|
|call  _exit@PLT|call _exit结束进程|

初始化过程可由下图直观表示：
![bthread_make_fcontext](/images/docs/bthread_make_fcontext.png)

### 协程的切换部分
实现协程上下文切换有很多种方法，本质要做的都是保存和恢复寄存器和栈信息。
``` c++
inline void jump_stack(ContextualStack* from, ContextualStack* to) {
    bthread_jump_fcontext(&from->context, to->context, 0/*not skip remained*/);
}
```
调用时会将当前的上下文保存到ofc中，并切换到目标上下文nfc进行执行
``` c++
intptr_t BTHREAD_CONTEXT_CALL_CONVENTION
bthread_jump_fcontext(bthread_fcontext_t * ofc, bthread_fcontext_t nfc,
                      intptr_t vp, bool preserve_fpu = false);
#if defined(BTHREAD_CONTEXT_PLATFORM_linux_x86_64) && defined(BTHREAD_CONTEXT_COMPILER_gcc)
__asm (
".text\n"
".globl bthread_jump_fcontext\n"
".type bthread_jump_fcontext,@function\n"
".align 16\n"
"bthread_jump_fcontext:\n"
"    pushq  %rbp  \n"
"    pushq  %rbx  \n"
"    pushq  %r15  \n"
"    pushq  %r14  \n"
"    pushq  %r13  \n"
"    pushq  %r12  \n"
"    leaq  -0x8(%rsp), %rsp\n"
"    cmp  $0, %rcx\n"
"    je  1f\n"
"    stmxcsr  (%rsp)\n"
"    fnstcw   0x4(%rsp)\n"
"1:\n"
"    movq  %rsp, (%rdi)\n"
"    movq  %rsi, %rsp\n"
"    cmp  $0, %rcx\n"
"    je  2f\n"
"    ldmxcsr  (%rsp)\n"
"    fldcw  0x4(%rsp)\n"
"2:\n"
"    leaq  0x8(%rsp), %rsp\n"
"    popq  %r12  \n"
"    popq  %r13  \n"
"    popq  %r14  \n"
"    popq  %r15  \n"
"    popq  %rbx  \n"
"    popq  %rbp  \n"
"    popq  %r8\n"
"    movq  %rdx, %rax\n"
"    movq  %rdx, %rdi\n"
"    jmp  *%r8\n"
".size bthread_jump_fcontext,.-bthread_jump_fcontext\n"
".section .note.GNU-stack,\"\",%progbits\n"
);

#endif
```
#### 代码中的汇编指令解释
|汇编指令|解释|
|:---|:---|
|pushq  %rbp|将寄存器rbp-r12依此push到当前协程栈中保存|
|pushq  %rbx|-|
|pushq  %r15|-|
|pushq  %r14|-|
|pushq  %r13|-|
|pushq  %r12|-|
|leaq  -0x8(%rsp), %rsp|rsp栈顶下移8字节 --->prepare stack for FPU 浮点运算寄存器|
|cmp  $0, %rcx|比较rcx和0，因为rcx为0，所以zf为1，rcx：第四个参数 preserve_fpu|
|je  1f|因为zf为1，所以跳转|
|stmxcsr  (%rsp)|存储 MMX 控制字和状态字，将MXCSR寄存器状态保存到rsp所在位置|
|fnstcw   0x4(%rsp)|存储 x87 控制字，将 FPU 控制字的当前值存储到rsp + 4|
|1f:||
|movq  %rsp, (%rdi)|将rsp保存至rdi中，rsp指向当前协程栈栈顶（原上下文），rdi为第一个入参，即ofc|
|movq  %rsi, %rsp|将rsi保存到rsp中，rsi为第二个参数，即nfc（目标上下文），此时栈顶指针rsp指向了新的协程栈|
|cmp  $0, %rcx|比较rcx和0，因为rcx为0，所以zf为1|
|je  2f|因为zf为1，所以跳转|
|ldmxcsr  (%rsp)||
|fldcw  0x4(%rsp)||
|2f:||
|leaq  0x8(%rsp), %rsp\n"|将rsp上移8字节|
|popq  %r12|将协程栈中r12-rbp依次pop到对应寄存器|
|popq  %r13|-|
|popq  %r14|-|
|popq  %r15|-|
|popq  %rbx|-|
|popq  %rbp|-|
|popq  %r8|-|
|movq  %rdx, %rax|将rdx保存到rax，rdx为第三个参数，rax为返回值|
|movq  %rdx, %rdi|将rdx保存到rdi，rdi为第一个入参，因此将作为新协程运行的入参|
|jmp  *%r8|跳转到r8对应的寄存器运行|

在协程切换过程中有两种情况，第一种为新协程是通过bthread_make_fcontext函数刚刚创建的栈，另一种是已经运行过的栈，这两种过程分别如下图所示：
![bthread_jump_fcontext](/images/docs/bthread_jump_fcontext.png)

参考资料
1. https://zhuanlan.zhihu.com/p/148314164
2. http://jinke.me/2018-09-14-coroutine-context-switch/
3. http://cons.mit.edu/fa18/x86-64-architecture-guide.html
4. https://blog.csdn.net/thisinnocence/article/details/50936470
