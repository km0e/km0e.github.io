+++
date = 2025-09-15T23:27:44Z
update-date = 2025-09-14T15:01:26Z
description = "讨论XSL 协程的 Cancel 机制"
draft = true
title = "XSL 协程 Cancel机制"

[extra]
toc = true

[taxonomies]
tags = ["XSL"]
+++

## 简介

XSL 协程支持取消（Cancel）机制，允许在协程执行过程中中断其操作。这对于处理长时间运行的任务或需要响应外部事件的场景非常有用。本文主要探讨 XSL 协程的 Cancel 机制及其实现方式。

## 目标

- 允许协程（准确说是协程内的异步IO操作）在执行过程中被取消。
- 具有主动探测取消请求的能力，减少资源浪费。
- 确保多层取消机制的正确性。

## Cancel 机制设计

实际上`Cancel`机制的实现主要靠在异步任务中插入取消检查点来实现。每当协程执行到这些检查点时，它会检查是否有取消请求。如果有，协程会立即停止执行并返回取消状态。

在这里我们主要讨论多线程环境下的协程取消机制。

Note:

- 协程状态存储于协程上下文中。

### 方案一

实现伪代码:

```cpp
enum class CancelState {
  Pending,    // 正在准备取消
  Running,    // 正在运行
  Canceled    // 已取消
};
struct CoroContext {
  std::size_t version{0}; // 版本号
  std::atomic<CancelState> state{CancelState::Pending}; // 取消状态
  std::atomic<std::function<void()>*> cc; // 取消回调
  bool cancellable; // 是否可取消
    // 其他协程上下文信息
};
struct CancellableAwaitable {
  CoroContext* ctx;
  bool await_ready() {
    return ctx->cc.load() != nullptr&&this->_await_ready(); // 已经在取消中
  }
  void await_suspend(std::coroutine_handle<> h) {
    auto cc = new std::function<void()>([h, this]() mutable {
      this->resume();
    });
    if (!ctx->cc.compare_exchange_strong(nullptr, cc)) {
      delete cc;
      return false; // 已经在取消中
    }
    return this->_await_suspend(h);
  }
  void await_resume() {

  }
};
struct CancelStarter {
  CoroContext* ctx;
  size_t version;
  void init(CoroContext* c) {
    if (ctx->cancellable) {
      if (ctx->cc.load() != nullptr) {
        return; // 已经在取消中
      }
      std::atomic_thread_fence(std::memory_order_acquire);
      this->version = ctx->version;
    }
    ctx->cancellable = true;
    this->version = ctx->version;
  }
  void operator()() {
    auto cc = ctx->cc.exchange((std::function<void()>*)-1);
    if (cc != nullptr && cc != (std::function<void()>*)-1) {
      (*cc)();
      delete cc;
    }
  }
};
```
