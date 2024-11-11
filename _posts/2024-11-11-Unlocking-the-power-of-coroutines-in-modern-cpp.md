---
title: Unlocking the Power of Coroutines in Modern C++
date: 2024-11-11 19:02:00 +0200
categories: [C++]
tags: [coroutines]     # TAG names should always be lowercase
toc: true
---

# Introduction

The release of C++20 brought a plethora of new features to the language, but perhaps none as transformative as coroutines. Coroutines introduce a powerful abstraction for asynchronous programming, enabling developers to write code that is both efficient and easy to understand. In this blog post, we’ll dive deep into the world of coroutines, exploring how they work under the hood and how you can leverage them in your projects.

# What Are Coroutines?

At their core, coroutines are generalizations of subroutines (functions). While a regular function runs to completion once called, a coroutine can suspend execution to be resumed later, maintaining its state between suspensions. This capability makes coroutines exceptionally useful for tasks like asynchronous I/O, concurrency, and stateful generators.

# The Anatomy of a Coroutine

To understand coroutines in C++, we need to look at several key components:
 1. Coroutine Functions: These are functions that use the co_keywords (`co_await`, `co_yield`, `co_return`).
 2. Promise Types: Objects that manage the state and lifetime of a coroutine.
 3. Awaitables and Awaiters: Types that can be `co_awaited`, defining how suspension and resumption occur.
 4. Coroutine Handles: Objects that provide low-level control over coroutine execution.

# A Simple Coroutine Example

Let’s start with a minimal coroutine:

```C++
# include <coroutine>
# include <iostream>

struct MyCoroutine {
    struct promise_type {
        MyCoroutine get_return_object() { return {}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() {}
    };
};

MyCoroutine simple_coroutine() {
    std::cout << "Hello from coroutine!" << std::endl;
    co_return;
}

int main() {
    simple_coroutine();
    return 0;
}
```

In this example, `simple_coroutine` is a coroutine function because it uses `co_return`. The `promise_type` defines how the coroutine behaves.

# Diving Deeper: Understanding Promise Types

The promise type is a user-defined type that controls the behavior of a coroutine. When a coroutine is called, it creates a promise object that manages its execution.

Key members of a promise type include:

* `get_return_object()`: Returns the object that the coroutine function call will result in.
* `initial_suspend()`: Determines whether the coroutine suspends immediately after being called.
* `final_suspend()`: Determines what happens when the coroutine finishes execution.
* `return_void()` or `return_value()`: Defines what happens when co_return is called.
* `unhandled_exception()`: Handles exceptions thrown within the coroutine.

## Customizing Suspension Behavior

By returning different types from `initial_suspend()` and `final_suspend()`, you can control when the coroutine suspends. Common return types include:

* `std::suspend_always`: The coroutine always suspends at this point.
* `std::suspend_never`: The coroutine never suspends at this point.

## Awaitables and Awaiters

The co_await operator is used to suspend a coroutine until a given awaitable is ready. An awaitable is any type that can be awaited, and it must provide an awaiter that defines the suspension behavior.

An awaiter must have the following methods:

* `bool await_ready()`: Returns `true` if the coroutine should resume immediately without suspending.
* `void await_suspend(std::coroutine_handle<> h)`: Called when the coroutine suspends.
* `auto await_resume()`: Called when the coroutine resumes; returns the result of the `co_await` expression.

## Creating a Custom Awaitable

Let’s create a simple awaitable that suspends the coroutine for a fixed number of times:

```C++
struct SuspendFixed {
    int count;
    bool await_ready() { return count <= 0; }
    void await_suspend(std::coroutine_handle<> h) {
        --count;
        h.resume();
    }
    void await_resume() {}
};
```

Usage in a coroutine:

```C++
MyCoroutine suspend_example() {
    co_await SuspendFixed{3};
    std::cout << "Resumed after suspending 3 times." << std::endl;
    co_return;
}
```

# Real-World Application: Asynchronous I/O

One of the most powerful applications of coroutines is asynchronous I/O. By leveraging coroutines, you can write code that looks synchronous but runs asynchronously.

## Example with `std::future`

```C++
# include <future>

std::future<void> async_task() {
    std::cout << "Starting async task..." << std::endl;
    co_await std::async([] {
        std::this_thread::sleep_for(std::chrono::seconds(2));
        std::cout << "Async operation completed." << std::endl;
    });
    std::cout << "Resuming after async operation." << std::endl;
}
```

In this example, `async_task` is a coroutine that waits for an asynchronous operation to complete before resuming.

# Best Practices and Considerations

* Understand the Lifespan: Be cautious with the lifetime of variables captured by the coroutine. Since execution is suspended and resumed, you must ensure that any referenced data remains valid.
* Exception Handling: Always implement unhandled_exception() in your promise type to handle exceptions within the coroutine.
* Performance: While coroutines offer performance benefits over traditional threading models, misuse can lead to overhead. Profile your application to understand the impact.

# Conclusion

Coroutines in C++20 are a powerful tool that can simplify asynchronous programming and improve performance. By understanding the underlying mechanics—such as promise types, awaitables, and coroutine handles—you can harness the full potential of coroutines in your applications.

As with any advanced feature, coroutines come with a learning curve. However, the investment pays off by enabling you to write cleaner, more maintainable code for complex asynchronous tasks.

References

* [C++20 Coroutines Proposal (P0912R5)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0912r5.pdf)
* [cppreference.com: Coroutines](https://en.cppreference.com/w/cpp/language/coroutines)
* [ISO C++ Committee: Coroutines TS](https://isocpp.org/files/papers/N4680.pdf)
