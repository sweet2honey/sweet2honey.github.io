---
title: "[Video] Comparing C to machine language"
mathjax: false
toc: true
categories:
  - Reading
tags:
  - cpp
date: 2025-04-28 11:22:39
description:
---
# Conclusion

[Comparing C to machine language](https://www.youtube.com/watch?v=yOyaJXpAYZQ)

- 简单理解 `cmpl` 和 `jl` 如何判断大小

  - 有兴趣可以再去了解下这几个标志位，以及对于有无符号整数，是怎么利用这几个标志位来做大小、溢出比较的。

  - 关键标志位

    -   `cmpl` 会设置以下标志位（用 `result = dest - src` 解释）：

    - | 标志位 | 名称     | 触发条件（何时为 1）               | 用途                   |
      | ------ | -------- | ---------------------------------- | ---------------------- |
      | **ZF** | 零标志   | `result == 0`（两数相等）          | 判断是否相等           |
      | **SF** | 符号标志 | `result < 0`（结果为负）           | 判断有符号数的符号     |
      | **OF** | 溢出标志 | 结果溢出（有符号数的正负意外翻转） | 辅助判断有符号数的比较 |
      | **CF** | 进位标志 | `dest < src`（无符号比较）         | 判断无符号数的大小     |

  - 对比其他跳转指令

    - | 指令 | 含义            | 判断条件        | 比较类型 |
      | ---- | --------------- | --------------- | -------- |
      | `jl` | Jump if Less    | `SF != OF`      | 有符号   |
      | `jg` | Jump if Greater | `ZF=0 && SF=OF` | 有符号   |
      | `jb` | Jump if Below   | `CF=1`          | 无符号   |
      | `ja` | Jump if Above   | `CF=0 && ZF=0`  | 无符号   |

  - 一句话总结

    -   **`cmpl A, B`**：计算 `B - A`，设标志位。
    -   **`jl`**：若 `B < A`（有符号数），则跳转。（实际是检查 `SF != OF`，涵盖溢出情况。）

- ABI 参数传递规则
  -  x86-64 System V ABI 的参数传递规则：
  -  整数/指针参数：前 6 个参数依次通过 `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9` 传递。
  -  浮点参数：前 8 个参数通过 `%xmm0` ~ `%xmm7` 传递。
  -  多余参数：从第 7 个参数开始，从右到左压栈（即最后一个参数先入栈）。

- ABI 规则和编译器优化
  -  编译器在生成汇编时，并不严格按源码顺序填充寄存器；
  -  关键点：ABI 只要求**函数调用时**寄存器和栈中的参数值是正确的，不关心指令顺序。



# Analysis

```C++
#include <stdio.h>

int main(void) {
    int x, y, z;
    while (1) {
        x = 0;
        y = 1;
        do {
            printf("%d\n", x);
            z = x + y;
            x = y;
            y = z;
        } while (x < 255);
    }
}
```

```Assembly
Disassembly of section .text:
0000000000001140 <main>:
    # set up
    1140:       55                      push   %rbp
    1141:       48 89 e5                mov    %rsp,%rbp
    1144:       48 83 ec 10             sub    $0x10,%rsp
    1148:       c7 45 fc 00 00 00 00    movl   $0x0,-0x4(%rbp)
    # variable init
    114f:       c7 45 f8 00 00 00 00    movl   $0x0,-0x8(%rbp) # x
    1156:       c7 45 f4 01 00 00 00    movl   $0x1,-0xc(%rbp) # y
    # printf
    115d:       8b 75 f8                mov    -0x8(%rbp),%esi # x
    1160:       48 8d 3d 9d 0e 00 00    lea    0xe9d(%rip),%rdi        # 2004 <_IO_stdin_used+0x4>
    1167:       b0 00                   mov    $0x0,%al
    1169:       e8 c2 fe ff ff          call   1030 <printf@plt>
    # z = x + y
    116e:       8b 45 f8                mov    -0x8(%rbp),%eax # x -> %eax
    1171:       03 45 f4                add    -0xc(%rbp),%eax # add y -> %eax
    1174:       89 45 f0                mov    %eax,-0x10(%rbp) # %eax -> z
    # x = y
    1177:       8b 45 f4                mov    -0xc(%rbp),%eax
    117a:       89 45 f8                mov    %eax,-0x8(%rbp)
    # y = z
    117d:       8b 45 f0                mov    -0x10(%rbp),%eax
    1180:       89 45 f4                mov    %eax,-0xc(%rbp)
    # while condition
    1183:       81 7d f8 ff 00 00 00    cmpl   $0xff,-0x8(%rbp) # 255 ? x 
    118a:       0f 8c cd ff ff ff       jl     115d <main+0x1d> # jump if less than ->  # printf
    1190:       e9 ba ff ff ff          jmp    114f <main+0xf> # jump -> # variable init

Disassembly of section .plt:
0000000000001030 <printf@plt>:
    1030:       ff 25 e2 2f 00 00       jmp    *0x2fe2(%rip)        # 4018 <printf@GLIBC_2.2.5>
    1036:       68 00 00 00 00          push   $0x0
    103b:       e9 e0 ff ff ff          jmp    1020 <_init+0x20>

Disassembly of section .rodata:
0000000000002000 <_IO_stdin_used>:
    2000:       01 00                   add    %eax,(%rax)
    2002:       02 00                   add    (%rax),%al
    2004:       25                      .byte 0x25
    2005:       64 0a 00                or     %fs:(%rax),%al
```