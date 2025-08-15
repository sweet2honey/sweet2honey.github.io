---
title: Lambda vs Bind
mathjax: false
categories:
    - programming
tags:
    - cpp
    - lambda
date: 2025-04-16 01:00
---

# Source Code
```cpp
// https://godbolt.org/z/9xnnqPTKo
#include <cstdint>
#include <functional>
#include <iostream>

#define FORWARD_TO_MEMBER_FUNC(func_name) \
    [this](auto&&... args) { this->func_name(std::forward<decltype(args)>(args)...); }

using CallbackT = std::function<void(int32_t, int32_t, int32_t)>;

void Register(CallbackT callback) {
    std::cout << "Callback registered." << std::endl;
}

class Binder {
public:
    // 经典：auto 写得爽，但是可能会有隐式转换、拷贝的问题，需要比对 CallbackT 才能获得准确的信息
    void InitLambda() {
        Register([this](auto i, auto j, auto k) { this->Process(i, j, k); });
    }

    // 稳定：CallbackT 修改后，只需要更新 Process，不需要修改 lambda 的参数类型
    // 高效：使用 std::forward 进行完美转发，避免了隐式转换和拷贝的问题
    // 高效：生成的汇编跟手动写的 lambda 一模一样
    void InitLambdaForward() {
        Register([this](auto&&... args) { this->Process(std::forward<decltype(args)>(args)...); });
    }

    // 懒就是好
    void InitLambdaMacro() {
        Register(FORWARD_TO_MEMBER_FUNC(Process));
    }

    // 稳定：同上
    // 简洁：阅读顺序很舒服，没有 lambda 那么多的括号
    // 低效：通常认为 bind 就是比 lambda 性能差；看汇编比 lambda 方案多用一个 ptr 大小（函数指针？）
    void InitBind() {
        Register(std::bind_front(&Binder::Process, this));
    }

private:
    void Process(int32_t, int32_t, int32_t) {}
};

int main() {
    Binder{}.InitLambda();
    Binder{}.InitLambdaForward();
    Binder{}.InitLambdaMacro();
    Binder{}.InitBind();
}
```

# Assembly Analysis of `InitBind()`

The assembly code for `InitBind()` reveals why it's generally considered less efficient than the lambda approaches. Here's what's happening:

## Stack Setup and Function Preparation

```assembly
push    rbx                      ; Save rbx register for later use
sub     rsp, 80                  ; Allocate 80 bytes on the stack
lea     rax, [rip + Binder::Process(int, int, int)] ; Load address of Process method
mov     qword ptr [rsp + 8], rax ; Store function pointer on stack
mov     qword ptr [rsp + 16], 0  ; Initialize stack memory
mov     qword ptr [rsp], rdi     ; Store 'this' pointer on stack
```

## Creating the Bound Function Object

```assembly
lea     rbx, [rsp + 56]          ; Store destination address in rbx
lea     rsi, [rsp + 8]           ; Load address of function pointer
mov     rdx, rsp                 ; Load address of 'this' pointer
mov     rdi, rbx                 ; Set first parameter (destination)
call    std::_Bind_front<...>    ; Call bind_front implementation
```

## Converting to std::function and Registering

```assembly
lea     rdi, [rsp + 24]          ; Set destination for std::function
mov     rsi, rbx                 ; Load the bound function object
call    std::function<...>::function<...> ; Convert bind_front to std::function
call    Register(std::function<void (int, int, int)>) ; Call Register
```

## Cleanup

```assembly
lea     rdi, [rsp + 24]          ; Load address of std::function
call    std::_Function_base::~_Function_base() ; Call destructor
add     rsp, 80                  ; Free stack space
pop     rbx                      ; Restore rbx
ret                              ; Return
```

## Performance Implications

1. **Multiple Function Calls**: The code makes several function calls:
   - To `std::bind_front`
   - To `std::function` constructor
   - To the `Register` function
   - To destructors

2. **Object Construction**: It constructs intermediate objects:
   - A `_Bind_front` object
   - A `std::function` object

3. **Memory Operations**: There are many memory operations:
   - Stack allocations (80 bytes)
   - Multiple pointer manipulations
   - Storing/loading function pointers

4. **Indirection**: The use of function pointers adds indirection

Lambda approaches avoid most of this overhead by creating simpler, more direct function objects that the compiler can often inline more effectively.

# Comparing Assembly: `InitLambda()` vs `InitBind()`

Looking at the assembly for both functions, there are clear differences that explain the performance characteristics:

## InitLambda() Assembly Overview

```assembly
push    rbx                      ; Save rbx register
sub     rsp, 48                  ; Allocate 48 bytes on stack (vs 80 bytes for InitBind)
mov     qword ptr [rsp + 8], rdi ; Store 'this' pointer
lea     rdi, [rsp + 16]          ; Destination for std::function
lea     rsi, [rsp + 8]           ; Source (this pointer)
call    std::function<...>::function<Binder::InitLambda()::'lambda'...> ; Create function
call    Register(std::function<...>) ; Call Register
lea     rdi, [rsp + 16]          ; Load address of std::function
call    std::_Function_base::~_Function_base() ; Destructor
add     rsp, 48                  ; Free stack space
pop     rbx                      ; Restore rbx
ret                              ; Return
```

## Key Differences

1. **Stack Usage**: 
   - `InitLambda()`: Uses only 48 bytes on the stack
   - `InitBind()`: Uses 80 bytes on the stack

2. **Function Calls**:
   - `InitLambda()`: Has only 3 function calls
   - `InitBind()`: Has 4 function calls including the extra `std::bind_front` call

3. **Complexity**:
   - `InitLambda()`: The assembly is simpler with fewer instructions
   - `InitBind()`: More instructions and memory operations

4. **Object Construction**:
   - `InitLambda()`: Directly constructs a `std::function` from the lambda
   - `InitBind()`: Creates an intermediate `_Bind_front` object first, then converts it to a `std::function`

## Performance Implications

1. **Memory Efficiency**: `InitLambda()` uses less stack space and has fewer memory operations

2. **Call Overhead**: `InitLambda()` has fewer function calls, reducing call overhead

3. **Inlining Potential**: Lambda code is more likely to be inlined by the compiler since it's direct

4. **Simplicity**: `InitLambda()` generates simpler code that's easier for the processor to execute efficiently

The comparison confirms that lambda-based approaches generally produce more efficient code than `std::bind_front`, as they avoid the additional indirection and object creation overhead.