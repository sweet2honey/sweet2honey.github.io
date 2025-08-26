---
title: Mock Rust's enum in C++
mathjax: false
toc: true
categories:
  - Programming
tags:
  - cpp
  - rust
date: 2025-08-26 10:57:35
description:
---

# Mock Rust's enum in C++

用 `std::variant` 和 `std::visit` 模拟 Rust 的 enum 和 match 语法糖。
`Overloaded` 老员工了。注意 C++ 17 里面需要一个 deduction guide。

通过 `[](const auto&) {}` 这种 catch-all 的方式来忽略不关心的分支，对应 Rust 里的 `_ => {}`。

还有个用宏的，也可一看[^1]。

# Code

下面的代码还是得用 C++20，因为用了 designated initializers。
```cpp
#include <cstdint>
#include <iostream>
#include <string>
#include <variant>

struct UserEvent {
    struct Login {
        std::string username;
    };
    struct Signoff {};
    struct Query {
        uint32_t item_id{};
    };
    struct Click {
        int32_t x{};
        int32_t y{};
    };

    std::variant<std::monostate, Login, Signoff, Query, Click> event;
};

template <typename... Ts>
struct Overloaded : Ts... {
    using Ts::operator()...;
};

//* For C++17, we need a deduction guide
template <typename... Ts>
Overloaded(Ts...) -> Overloaded<Ts...>;

template <typename E, typename O>
constexpr auto visit(const E& e, O&& o) {
    return std::visit(std::forward<O>(o), e.event);
}

template <typename E, typename... Ts>
constexpr auto match(const E& e, Ts&&... ts) {
    return std::visit(Overloaded<std::decay_t<Ts>...>{std::forward<Ts>(ts)...}, e.event);
}

int main() {
    //* use visit
    auto visitor =
        Overloaded{[](const UserEvent::Login& login) { std::cout << "User logged in: " << login.username << "\n"; },  //
                   [](const UserEvent::Signoff&) { std::cout << "User signed off\n"; },                               //
                   [](const auto&) {}};
    visit(UserEvent{UserEvent::Login{"Alice"}}, visitor);
    visit(UserEvent{UserEvent::Signoff{}}, visitor);

    //* use match
    auto event = UserEvent{UserEvent::Click{.x = 10, .y = 20}};
    match(
        event,                                                                                                 //
        [](const UserEvent::Click& click) { std::cout << "Clicked " << click.x << ", " << click.y << "\n"; },  //
        [](const auto&) {});
}
```

Outputs:
```shell
User logged in: Alice
User signed off
Clicked 10, 20
```

# References
[Rust enums in c++17](https://rucadi.eu/rust-enums-in-c-17.html)
[^1]: [Rust enums in Modern C++ – Match Pattern](https://thatonegamedev.com/cpp/rust-enums-in-modern-cpp-match-pattern/)