---
title: linkage example
mathjax: false
date: 2024-12-03 16:49:42
categories:
    - programming
tags:
    - compiler
    - linkage
---
Compile with:<br/>
`clang++ var.cpp main.cpp`

var.cpp
```cpp
// external linkage
int i = 0;

// internal linkage
// const int i = 0;

// external linkage
// extern const int i = 0;

// internal linkage
// static int i = 0;
```

main.cpp
```cpp
#include <iostream>

// refer to `i`
extern int i;
// may use
// extern const int i;

int main()
{
    std::cout << i;
}
```