---
title: 【译】SE-0300 Continuation -- 执行同步代码的异步任务接口
date: 2021-03-31
---

> 原文链接：[SE-0300 Continuations for interfacing async tasks with synchronous code](https://github.com/apple/swift-evolution/blob/main/proposals/0300-continuation.md)

* Proposal: [SE-0300](https://github.com/apple/swift-evolution/blob/main/proposals/0300-continuation.md)
* Authors: [John McCall](https://github.com/rjmccall), [Joe Groff](https://github.com/jckarter), [Doug Gregor](https://github.com/DougGregor), [Konrad Malawski](https://github.com/ktoso)
* Review Manager: [Ben Cohen](https://github.com/airspeedswift)
* Status: **Implemented**
* Previous Revisions: [1](https://github.com/apple/swift-evolution/blob/5f79481244329ec2860951c0c49c101aef5069e7/proposals/0300-continuation.md), [2](https://github.com/apple/swift-evolution/blob/61c788cdb9674c99fc8731b49056cebcb5497edd/proposals/0300-continuation.md)

## 简介

<!--
Asynchronous Swift code needs to be able to work with existing synchronous code that uses techniques such as completion callbacks and delegate methods to respond to events. Asynchronous tasks can suspend themselves on **continuations** which synchronous code can then capture and invoke to resume the task in response to an event.
-->

异步 Swift代 码需要能够与现有的同步代码一起工作，这些代码使用回调和 delegate 等方式来响应事件。异步任务可以在 **continuations** 上暂停自己，然后同步代码可以捕获并调用它来恢复任务以响应事件。

Swift-evolution thread:

- [Structured concurrency](https://forums.swift.org/t/concurrency-structured-concurrency/41622)
- [Continuations for interfacing async tasks with synchronous code](https://forums.swift.org/t/concurrency-continuations-for-interfacing-async-tasks-with-synchronous-code/43619)

<!--more-->

## 动机

<!--
Swift APIs often provide asynchronous code execution by way of a callback. This may occur either because the code itself was written prior to the introduction of async/await, or (more interestingly in the long term) because it ties in with some other system that is primarily event-driven. In such cases, one may want to provide an async interface to clients while using callbacks internally.  In these cases, the calling async task needs to be able to suspend itself, while providing a mechanism for the event-driven synchronous system to resume it in response to an event.
-->

Swift 的 API 经常会通过回调的方式提供异步执行的功能。这种情况可能是因为代码本身是在引入 async/await 之前编写的，或者（更有趣的是，从长远来看）是因为它与其它一些（主要是）事件驱动的系统联系在一起。在这种情况下，人们可能希望向客户端提供一个异步接口，同时在内部使用回调。在这些情况下，调用异步任务需要能够暂停自己，同时提供一个机制让事件驱动的同步系统在响应事件时恢复它。

## 解决方案

<!--
The library will provide APIs to get a **continuation** for the current asynchronous task. Getting the task's continuation suspends the task, and produces a value that synchronous code can then use a handle to resume the task. Given a completion callback based API like:
-->

本提案将提供 API 来获取当前异步任务的 **continuation**。获取任务的 continuation 会暂停任务，并产生一个值，同步的代码可以使用 handle 来恢复任务。给定一个基于 completionHandler 的 API，例如：

```swift
func beginOperation(completion: (OperationResult) -> Void)
```

<!--
we can turn it into an `async` interface by suspending the task and using its continuation to resume it when the callback is invoked, turning the argument passed into the callback into the normal return value of the async function:
-->

我们可以把它变成一个 `async` 接口，先将当前的任务暂停，然后把它的 continuation 传入闭包中用来恢复任务，把传入回调的参数作为 async 函数的返回值：

```swift
func operation() async -> OperationResult {
  // 暂停当前任务，立即执行这个闭包，并且将它的 contunation 传进去
  return await withUnsafeContinuation { continuation in
    // 执行基于 callback 的同步 API
    beginOperation(completion: { result in
      // 当回调执行时恢复 contiuation
      continuation.resume(returning: result)
    }) 
  }
}
```

## 具体设计

### 原始的 unsafe continuations

<!--
The library provides two functions, `withUnsafeContinuation` and `withUnsafeThrowingContinuation`, that allow one to call into a callback-based API from inside async code. Each function takes an *operation* closure, which is expected to call into the callback-based API. The closure receives a continuation instance that must be resumed by the callback, either to provide the result value or (in the throwing variant) the thrown error that becomes the result of the `withUnsafeContinuation` call when the async task resumes:
-->

本提案提供了两个函数，`withUnsafeContinuation` 和 `withUnsafeThrowingContinuation`，允许用户在异步代码内部使用基于回调的 API。这两个函数都需要传入一个 *operation* 闭包，它通常会在基于回调的 API 里调用。闭包里接收一个必须由回调恢复的 continuation 实例，用于提供结果值或（在 throw 的版本中）抛出错误，当异步任务恢复时，成为 `withUnsafeContinuation` 调用的结果：

```swift
struct UnsafeContinuation<T, E: Error> {
  func resume(returning: T)
  func resume(throwing: E)
  func resume(with result: Result<T, E>)
}

extension UnsafeContinuation where T == Void {
  func resume() { resume(returning: ()) }
}

extension UnsafeContinuation where E == Error {
  // 允许 Result 版本使用更严格的 Error 类型
  func resume<ResultError: Error>(with result: Result<T, ResultError>)
}

func withUnsafeContinuation<T>(
    _ operation: (UnsafeContinuation<T, Never>) -> ()
) async -> T

func withUnsafeThrowingContinuation<T>(
    _ operation: (UnsafeContinuation<T, Error>) throws -> ()
) async throws -> T
```

<!--
`withUnsafe*Continuation` will run its `operation` argument immediately in the task's current context, passing in a *continuation* value that can be used to resume the task. The `operation` function must arrange for the continuation to be resumed at some point in the future; after the `operation` function returns, the task is suspended. The task must then be brought out of the suspended state by invoking one of the continuation's `resume` methods.  Note that `resume` immediately returns control to the caller after transitioning the task out of its suspended state; the task itself does not actually resume execution until its executor reschedules it. The argument to `resume(returning:)` becomes the return value of `withUnsafe*Continuation` when the task resumes execution.  `resume(throwing:)` can be used instead to make the task resume by propagating the given error. As a convenience, given a `Result`, `resume(with:)` can be used to resume the task by returning normally or raising an error according to the state of the `Result`. If the `operation` raises an uncaught error before returning, this behaves as if the operation had invoked `resume(throwing:)` with the error.
-->

`withUnsafe*Continuation` 将在任务的当前上下文中立即执行它的 `operation` 参数，传入一个可用于恢复任务的 *continuation* 值。`operation` 函数必须安排在未来的某个时刻恢复 continuation；在 `operation` 函数返回后，任务就会被暂停。然后，必须通过调用 continuation 任意一个 `resume` 方法使任务脱离暂停状态。注意，`resume` 在将任务从暂停状态过渡出来后，会立即将控制权返回给调用者；任务本身实际上并不会恢复执行，而是等到它的执行者再次调度它。当任务恢复执行时，`resume(returning:)` 的参数会成为 `withUnsafe*Continuation` 的返回值。`resume(throwing:)` 可以通过传入给定的 Error 使任务恢复。为了方便起见，可以传入一个 `Result`，`resume(with:)` 会根据 `Result` 的状态，通过正常返回或抛出错误来恢复任务。如果 `operation` 在返回前引发了一个未捕获的错误，就会跟调用了 `resume(throwing:)` 一样。

<!--
If the return type of `withUnsafe*Continuation` is `Void`, one must specify a value of `()` when calling `resume(returning:)`. Doing so produces some unsightly code, so `Unsafe*Continuation<Void>` has an extra member `resume()` that makes the function call easier to read.
-->

如果 `withUnsafe*Continuation` 的返回类型是 `Void`，那么在调用 `resume(returning:)` 时必须传入 `()`。这样做会产生一些丑陋的代码，所以 `Unsafe*Continuation<Void>` 有一个额外的方法 `resume()`，使调用代码更容易阅读。

<!--
After invoking `withUnsafeContinuation`, exactly one `resume` method must be called *exactly-once* on every execution path through the program.  `Unsafe*Continuation` is an unsafe interface, so it is undefined behavior if a `resume` method is invoked on the same continuation more than once. The task remains in the suspended state until it is resumed; if the continuation is discarded and never resumed, then the task will be left suspended until the process ends, leaking any resources it holds.  Wrappers can provide checking for these misuses of continuations, and the library will provide one such wrapper, discussed below.
-->

在调用 `withUnsafeContinuation` 后，每个分支上都必须**调用一次且仅一次** `resume` 方法。`Unsafe*Continuation` 是一个不安全的接口，所以同一个 continuation 多次调用 `resume` 方法属于未定义的行为。在任务被恢复之前，它会一直处于暂停状态；如果 continuation 被释放掉了，并且从未被恢复，那么任务将一直处于暂停状态，直到进程结束，它所拥有的任何资源都会泄漏。我们可以提供一层封装捕获这些错误的使用，本提案也打算引入这样的一个 Wrapper，下面将详细讨论。

<!--
Using the `Unsafe*Continuation` API, one may for example wrap such (purposefully convoluted for the sake of demonstrating the flexibility of the continuation API) function:
-->

例如，使用 `Unsafe*Continuation` API，可以封装这样的函数（这里为了展示 continuation API 的灵活性而故意弄得很复杂）：

```swift
func buyVegetables(
  shoppingList: [String],
  // a) if all veggies were in store, this is invoked *exactly-once*
  onGotAllVegetables: ([Vegetable]) -> (),

  // b) if not all veggies were in store, invoked one by one *one or more times*
  onGotVegetable: (Vegetable) -> (),
  // b) if at least one onGotVegetable was called *exactly-once*
  //    this is invoked once no more veggies will be emitted
  onNoMoreVegetables: () -> (),
  
  // c) if no veggies _at all_ were available, this is invoked *exactly once*
  onNoVegetablesInStore: (Error) -> ()
)
// returns 1 or more vegetables or throws an error
func buyVegetables(shoppingList: [String]) async throws -> [Vegetable] {
  try await withUnsafeThrowingContinuation { continuation in
    var veggies: [Vegetable] = []

    buyVegetables(
      shoppingList: shoppingList,
      onGotAllVegetables: { veggies in continuation.resume(returning: veggies) },
      onGotVegetable: { v in veggies.append(v) },
      onNoMoreVegetables: { continuation.resume(returning: veggies) },
      onNoVegetablesInStore: { error in continuation.resume(throwing: error) },
    )
  }
}

let veggies = try await buyVegetables(shoppingList: ["onion", "bell pepper"])
```

<!--
Thanks to weaving the right continuation resume calls into the complex callbacks of the `buyVegetables` function, we were able to offer a much nicer overload of this function, allowing async code to interact with this function in a more natural straight-line way.
-->

由于在 `buyVegetables` 函数的复杂回调里正确地对 continuation resume 进行了调用，我们能够为这个函数提供一个更好的重载，让异步代码以更自然的方式与这个函数交互。

### Checked continuations

<!--
`Unsafe*Continuation` provides a lightweight mechanism for interfacing sync and async code, but it is easy to misuse, and misuse can corrupt the process state in dangerous ways. In order to provide additional safety and guidance when developing interfaces between sync and async code, the library will also provide a wrapper which checks for invalid use of the continuation:
-->

`Unsafe*Continuation` 为同步和异步代码的接口提供了一个轻量的机制，但它很容易用错，并且会以危险的方式破坏进程状态。为了提供额外的安全性和指导（在开发同步和异步代码交互的接口时），本提案还将提供一个 Wrapper，用于检查对 continuation 的非法使用：

```swift
struct CheckedContinuation<T, E: Error> {
  func resume(returning: T)
  func resume(throwing: E)
  func resume(with result: Result<T, E>)
}

extension CheckedContinuation where T == Void {
  func resume()
}

extension CheckedContinuation where E == Error {
  // Allow covariant use of a `Result` with a stricter error type than
  // the continuation:
  func resume<ResultError: Error>(with result: Result<T, ResultError>)
}

func withCheckedContinuation<T>(
    _ operation: (CheckedContinuation<T, Never>) -> ()
) async -> T

func withCheckedThrowingContinuation<T>(
  _ operation: (CheckedContinuation<T, Error>) throws -> ()
) async throws -> T
```

<!--
The API is intentionally identical to the `Unsafe` variants, so that code can switch easily between the checked and unchecked variants. For instance, the `buyVegetables` example above can opt into checking merely by turning its call of `withUnsafeThrowingContinuation` into one of `withCheckedThrowingContinuation`:
-->

这里的 API 特意与 `Unsafe` 的 API 保持一致，因此代码可以很容易地在 checked 和 unsafe 的版本之间切换。例如，上面的 `buyVegetables` 例子只需将 `withUnsafeThrowingContinuation` 的调用变成 `withCheckedThrowingContinuation` 的调用，就可以提供运行时的检查：

```swift
// returns 1 or more vegetables or throws an error
func buyVegetables(shoppingList: [String]) async throws -> [Vegetable] {
  try await withCheckedThrowingContinuation { continuation in
    var veggies: [Vegetable] = []

    buyVegetables(
      shoppingList: shoppingList,
      onGotAllVegetables: { veggies in continuation.resume(returning: veggies) },
      onGotVegetable: { v in veggies.append(v) },
      onNoMoreVegetables: { continuation.resume(returning: veggies) },
      onNoVegetablesInStore: { error in continuation.resume(throwing: error) },
    )
  }
}
```

<!--
Instead of leading to undefined behavior, `CheckedContinuation` will instead trap if the program attempts to resume the continuation multiple times.  `CheckedContinuation` will also log a warning if the continuation is discarded without ever resuming the task, which leaves the task stuck in its suspended state, leaking any resources it holds. These checks happen regardless of the optimization level of the program.
-->

`CheckedContinuation` 不会导致未定义的行为，相反，如果程序试图多次 resume continuation 的话，`CheckedContinuation` 就会捕获到这种情况并且触发 trap。如果在没有恢复任务的情况下释放了 continuation，这使任务停留在它的暂停状态，泄露它所拥有的任何资源，并且 `CheckedContinuation` 会打印一个警告。无论程序采用了哪种优化级别，这些检查都会执行。

## 更多例子

<!--
Continuations can be used to interface with more complex event-driven interfaces than callbacks as well. As long as the entirety of the process follows the requirement that the continuation be resumed exactly once, there are no other restrictions on where the continuation can be resumed. For instance, an `Operation` implementation can trigger resumption of a continuation when the operation completes:
-->

Continuation 也可以用来跟（比回调）更复杂的事件驱动接口对接。只要整个过程都符合要求（continuation 只被恢复一次），那么在哪里恢复 continuation 就没有其它限制。例如，一个 `Operation` 可以在操作完成时触发 continuation 的恢复：

```swift
class MyOperation: Operation {
  let continuation: UnsafeContinuation<OperationResult, Never>
  var result: OperationResult?

  init(continuation: UnsafeContinuation<OperationResult, Never>) {
    self.continuation = continuation
  }

  /* rest of operation populates `result`... */

  override func finish() {
    continuation.resume(returning: result!)
  }
}

func doOperation() async -> OperationResult {
  return await withUnsafeContinuation { continuation in
    MyOperation(continuation: continuation).start()
  }
}
```

<!--
Using APIs from the [structured concurrency proposal](https://github.com/apple/swift-evolution/blob/main/proposals/0304-structured-concurrency.md), one can wrap up a `URLSession` in a task, allowing the task's cancellation to control cancellation of the session, and using a continuation to respond to data and error events fired by the network activity:
-->

使用[结构化并发提案](https://github.com/apple/swift-evolution/blob/main/proposals/0304-structured-concurrency.md)中的 API，可以将一个 `URLSession` 包裹在一个任务中，让任务的取消来控制 session 的取消，并使用 continuation 来响应网络请求接收的数据和抛出的错误：

```swift
func download(url: URL) async throws -> Data? {
  var urlSessionTask: URLSessionTask?

  return try Task.withCancellationHandler {
    urlSessionTask?.cancel()
  } operation: {
    let result: Data? = try await withUnsafeThrowingContinuation { continuation in
      urlSessionTask = URLSession.shared.dataTask(with: url) { data, _, error in
        if case (let cancelled as NSURLErrorCancelled)? = error {
          continuation.resume(returning: nil)
        } else if let error = error {
          continuation.resume(throwing: error)
        } else {
          continuation.resume(returning: data)
        }
      }
      urlSessionTask?.resume()
    }
    if let result = result {
      return result
    } else {
      Task.cancel()
      return nil
    }
  }
}
```

<!--
It is also possible for wrappers around callback based APIs to respect their parent/current tasks's cancellation, as follows:
-->

围绕基于回调的 API 的 Wrapper 也可以跟随其父级/当前任务的取消，如下所示。

```swift
func fetch(items: Int) async throws -> [Items] {
  let worker = ... 
  return try Task.withCancellationHandler(
    handler: { worker?.cancel() }
  ) { 
    return try await withUnsafeThrowingContinuation { c in 
      worker.work(
        onNext: { value in c.resume(returning: value) },
        onCancelled: { value in c.resume(throwing: CancellationError()) },
      )
    } 
  }
}
```

<!--
If tasks were allowed to have instances, which is under discussion in the structured concurrency proposal, it would also be possible to obtain the task in which the fetch(items:) function was invoked and call isCanceled on it whenever the insides of the withUnsafeThrowingContinuation would deem it worthwhile to do so.
-->

如果允许任务拥有实例（这是在结构化并发提案里讨论的内容），那么也可以获得调用 `fetch(items:)` 函数的任务，并在 `withUnsafeThrowingContinuation` 内部认为需要时，调用它的 `isCanceled`。

## 其它方案

### 命名缩短为 `Continuation`（而不是 `CheckedContinuation`）

<!--
We could position `CheckedContinuation` as the "default" API for doing sync/async interfacing by leaving the `Checked` word out of the name. This would certainly be in line with the general philosophy of Swift that safe interfaces are preferred, and unsafe ones used selectively where performance is an overriding concern. There are a couple of reasons to hesitate at doing this here, though:
-->

我们可以将 `CheckedContinuation` 定位为做同步/异步接口"默认"的 API，把 `Checked` 这个词从名字中去掉。这符合 Swift 的理念，首选安全的接口，而不安全的接口是性能成为首要考虑因素时才会选择性地使用。不过，这里有几个理由让我们很犹豫：

<!--
- Although the consequences of misusing `CheckedContinuation` are not as severe as `UnsafeContinuation`, it still only does a best effort at checking for some common misuse patterns, and it does not render the consequences of continuation misuse entirely moot: dropping a continuation without resuming it will still leak the un-resumed task, and attempting to resume a continuation multiple times will still cause the information passed through the continuation to be lost. It is still a serious programming error if a `with*Continuation` operation misuses the continuation; `CheckedContinuation` only helps make the error more apparent.
- Naming a type `Continuation` now might take the "good" name away if, after we have move-only types at some point in the future, we want to introduce a continuation type that statically enforces the exactly-once property.
-->

- 虽然错误使用 `CheckedContinuation` 的后果没有 `UnsafeContinuation` 那么严重，但它仍然只是尽最大努力检查一些常见的错误模式，并不能掩盖掉错误使用 continuation 的后果：释放一个 continuation 而不恢复它仍然会泄露未恢复的任务，而试图多次恢复一个 continuation 仍然会导致通过 continuation 传递的信息丢失。如果 `with*Continuation` 的操作错误使用 continuation，仍然是一个严重的程序逻辑错误； `CheckedContinuation` 只能使错误变得更加明显。
- 如果我们在未来的某个时候有了 move-only 类型，我们想引入一个静态的只恢复 exactly-once 的 continuation 类型，那么现在命名一个类型 `Continuation` 会占用掉这个"好"名字。

### 不暴露 `UnsafeContinuation` 接口

<!--
One could similarly make an argument that `UnsafeContinuation` shouldn't be exposed at all, since the `Checked` form can always be used instead. We think that being able to avoid the cost of checking when interacting with performance-sensitive APIs is valuable, once users have validated that their interfaces to those APIs are correct.
-->

同样的，我们也可以得出另一个论点，即 `UnsafeContinuation` 根本不应该暴露给上层用户，因为用户总是会使用 `Checked` 的版本。但我们认为，在与性能敏感的 API 交互时，一旦用户验证了他们调用这些 API 的方式是正确的，避免掉 checked 的成本就是有价值的。

### 让 `CheckedContinuation` 在错误使用时全部触发 trap, 或者全部 log 出来

<!--
`CheckedContinuation` is proposed to trap when the program attempts to resume the same continuation twice, but only log a warning if a continuation is abandoned without getting resumed. We think this is the right tradeoff for these different situations for the following reasons:
-->

`CheckedContinuation` 的目的是当程序试图多次恢复同一个 continuation 时捕获这种情况，但如果一个 continuation 被释放而没有得到恢复，则只 log 一个警告。我们认为这是对不同情况的正确权衡，原因如下：

<!--
- With `UnsafeContinuation`, resuming multiple times corrupts the process and leaves it in an undefined state. By trapping when the task is resumed multiple times, `CheckedContinuation` turns undefined behavior into a well- defined trap situation.  This is analogous to other checked/unchecked pairings in the standard library, such as `!` vs. `unsafelyUnwrapped` for `Optional`.
- By contrast, failing to resume a continuation with `UnsafeContinuation` does not corrupt the task, beyond leaking the suspended task's resources; the rest of the program can continue executing normally. Furthermore, the only way we can currently detect and report such a leak is by using a class `deinit` in its implementation. The precise moment at which such a deinit would execute is not entirely predictable because of refcounting variability from ARC optimization. If `deinit` were made to trap, whether that trap is executed and when could vary with optimization level, which we don't think would lead to a good experience.
-->

- 对于 `UnsafeContinuation`，多次恢复会破坏进程，使其处于**未定义状态**。通过在任务多次恢复时触发 trap，`CheckedContinuation` 就会将未定义的行为变成了定义良好的 trap。这类似于标准库中其它的 checked/unchecked 配对，例如 `!` 与 `Optional` 的 `unsafelyUnwrapped`。
- 相比之下，如果没有用 `UnsafeContinuation` 来恢复 continuation，除了泄露暂停的任务的资源之外，并不会破坏任务，程序的其它部分也可以继续正常执行。此外，目前我们能检测和报告这种泄漏的唯一方法是在其实现中使用类的 `deinit`。由于 ARC 优化的 refcounting 变化，这样 deinit 执行的准确时机是完全无法预测的。如果把 `deinit` 做成 trap，那么这个 trap 是否被执行以及何时执行可能会随着优化等级的变化而变化，我们认为这不会带来很好的体验。

### 让 `*Continuation` 提供更多 `Task` API，或者允许 continuation 恢复 `Handle` 

<!--
The full `Task` and `Handle` API provides additional control over the task state to holders of the handle, particularly the ability to query and set cancellation state, as well as await the final result of the task, and one might wonder why the `*Continuation` types do not also expose this functionality.  The role of a `Continuation` is very different from a `Handle`, in that a handle represents and controls the entire lifetime of the task, whereas a continuation only represents a *single suspension point* in the lifetime of the task.  Furthermore, the `*Continuation` API is primarily designed to allow for interfacing with code outside of Swift's structured concurrency model, and we believe that interactions between tasks are best handled inside that model as much as possible.
-->

完整的 `Task` 和 `Handle` API 为 Handle 的持有者提供了对任务状态的额外控制，特别是查询和设置取消状态，以及等待任务最终结果的能力，大家可能会问，为什么 `*Continuation` 类型没有这些功能？`Continuation` 与 `Handle` 的作用有很大的不同，Handle 代表和控制任务的整个生命周期，而 continuation 只代表任务生命期中的**一个暂停点**。此外，`*Continuation` API 的设计主要是为了与 Swift 结构化并发模型之外的代码进行对接，我们认为任务之间的交互最好还是尽量在这个模型里处理。

<!--
Note that `*Continuation` also does not strictly need direct support for any task API on itself. If, for instance, someone wants a task to cancel itself in response to a callback, they can achieve that by funneling a sentinel through the continuation's resume type, such as an Optional's `nil`:
-->

注意，`*Continuation` 其实也不需要自己去直接提供任何任务的 API。例如，如果有人想让一个任务在响应回调时自行取消，他们可以通过控制 continuation 的 resume 类型来实现（例如一个 Optional 的 `nil`）：

```swift
let callbackResult: Result? = await withUnsafeContinuation { c in
  someCallbackBasedAPI(
    completion: { c.resume($0) },
    cancellation: { c.resume(nil) })
}

if let result = callbackResult {
  process(result)
} else {
  cancel()
}
```

### 提供 API 立刻恢复任务，避免队列跳转

<!--
Some APIs, in addition to taking a completion handler or delegate, also allow the client to control *where* that completion handler or delegate's methods are invoked; for instance, some APIs on Apple platforms take an argument for the dispatch queue the completion handler should be invoked by. In these cases, it would be optimal if the original API could resume the task directly on the dispatch queue (or whatever other scheduling mechanism, such as a thread or run loop) that the task would normally be resumed on by its executor. To enable this, we could provide a variant of `with*Continuation` that, in addition to providing a continuation, also provides the dispatch queue that the task expects to be resumed on. The `*Continuation` type in turn could provide an `unsafeResumeImmediately` set of APIs, which would immediately resume execution of the task on the current thread. This would enable something like this:
-->

有些 API 除了接收一个 completionHandler 或 delegate，还允许客户端控制 completionHandler 或 delegate 的方法调用的位置；例如，Apple 平台上的一些 API 接收一个参数，作为 completionHandler 应该被使用的 DispatchQueue。在这些情况下，如果原始 API 能够直接在 DispatchQueue（或其它任何调度机制，如 Thread 或 Runloop）上恢复任务，那将是最理想的。为了实现这一点，我们可以提供一个 `with*Continuation` 的变体，除了提供一个 continuation 之外，还提供任务期望被恢复的 DispatchQueue。`*Continuation` 类型也可以提供一套 `unsafeResumeImmediately` 的 API，它将立即在当前线程上恢复任务的执行。这样就可以实现下面的功能：

```swift
// Given an API that takes a queue and completion handler:
func doThingAsynchronously(queue: DispatchQueue, completion: (ResultType) -> Void)

// We could wrap it in a Swift async function like:
func doThing() async -> ResultType {
  await withUnsafeContinuationAndCurrentDispatchQueue { c, queue in
    // Schedule to resume on the right queue, if we know it
    doThingAsynchronously(queue: queue) {
      c.unsafeResumeImmediately(returning: $0)
    }
  }
}
```

<!--
However, such an API would have to be used very carefully; the programmer would have to be careful that `unsafeResumeImmediately` is in fact invoked in the correct context, and that it is safe to take over control of the current thread from the caller for a potentially unbounded amount of time.  If the task is resumed in the wrong context, it will break assumptions in the written code as well as those made by the compiler and runtime, which will lead to subtle bugs that would be difficult to diagnose. We can investigate this as an addition to the core proposal, if "queue hopping" in continuation- based adapters turns out to be a performance problem in practice.
-->

然而，这样的 API 必须非常谨慎地使用；程序员必须保证 `unsafeResumeImmediately` 在正确的上下文中被调用，并且在任意时间内，从调用者手中接管当前线程的行为必须保证是安全的。如果在错误的上下文中恢复任务，就会破坏代码逻辑以及编译器和运行时的假设，这将导致难以诊断的奇妙 bug。如果 continuation-based adapter 的"队列跳转"在实践中被证实是一个性能问题，我们可以将其作为核心提案的补充进行研究。

## 修订历史

Third revision:

- Replaced separate `*Continuation<T>` and `*ThrowingContinuation<T>` types with a single `Continuation<T, E: Error>` type parameterized on the error type.
- Added a convenience `resume()` equivalent to `resume(returning: ())` for continuations with a `Void` return type.
- Changed `with*ThrowingContinuation` to take an `operation` block that may throw, and to immediately resume the task throwing the error if an uncaught error propagates from the operation.

Second revision:

- Clarified the execution behavior of `with*Continuation` and `*Continuation.resume`, namely that `with*Continuation` immediately executes its operation argument in the current context before suspending the task, and that `resume` immediately returns to its caller after un-suspending the task, leaving the task to be scheduled by its executor.
- Removed an unnecessary invariant on when `resume` must be invoked; it is valid to invoke it exactly once at any point after the `with*Continuation` operation has started executing; it does not need to run exactly when the operation returns.
- Added "future direction" discussion of a potential more advanced API that could allow continuations to directly resume their task when the correct dispatch queue to do so is known.
- Added `resume()` on `Void`-returning `Continuation` types.
