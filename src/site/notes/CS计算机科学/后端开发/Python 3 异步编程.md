---
{"dg-publish":true,"permalink":"/CS计算机科学/后端开发/Python 3 异步编程/","tags":["Python"],"noteIcon":"","created":"2024-06-22T22:30:28.665+08:00","updated":"2024-04-02T10:01:19.000+08:00"}
---

# Python 3 异步编程

基础概念：

https://zhuanlan.zhihu.com/p/337978321

协程和多线程都是实现并发执行的有效方法，但它们各有优势和适用场景。是否需要在有了协程的情况下使用多线程，取决于具体的应用需求和环境。

### 协程的优势：
1. **轻量级**：协程比线程更轻量，创建和切换的开销更小。
2. **非阻塞I/O**：协程非常适合于处理 I/O 密集型任务，如网络请求、文件读写等。
3. **简化异步编程**：协程通过 `async` 和 `await` 语法，简化了异步编程的复杂性。
4. **更好的性能**：在 I/O 密集型任务中，协程通常可以提供更好的性能和资源利用率。

### 多线程的优势：
1. **CPU 密集型任务**：多线程可以在多核 CPU 上并行执行，适合 CPU 密集型任务。
2. **阻塞操作**：对于阻塞型操作（如等待外部资源），多线程可以使得一个线程的阻塞不会影响其他线程的执行。
3. **现有库的支持**：许多现有的库和框架已经是线程安全的，直接支持多线程。

### 协程与多线程结合使用：

在某些情况下，结合使用协程和多线程可以提供最佳的性能和资源利用。例如：

- 使用协程处理 I/O 密集型任务，而将 CPU 密集型任务放到线程中执行。
- 在一个协程中启动多个线程，每个线程执行不同的任务，同时利用协程来管理这些线程和处理 I/O 任务。

### 总结：
- 如果您的应用主要是 I/O 密集型的，那么使用协程可能就足够了。
- 如果您需要执行 CPU 密集型任务，或需要与现有的多线程库或框架集成，那么使用多线程是合适的。
- 在某些复杂的应用场景中，结合使用协程和多线程可以充分利用 CPU 和 I/O 资源，提高应用性能。

协程（Coroutine）和事件循环（Event Loop）是异步编程中的两个核心概念，尤其是在 Python 的 `asyncio` 库中。

### 协程（Coroutine）
1. **定义**：协程是一种程序组件，它可以在执行过程中挂起（暂停执行）并在稍后某个时刻从挂起的地方继续执行。这与传统的函数不同，后者一旦开始执行，就会运行到结束，中途无法停止。
2. **特点**：
   - **异步操作**：协程通常用于执行异步操作，比如 I/O 操作（文件读写、网络请求等），这些操作可能需要等待外部事件的完成，而不是占用 CPU 时间进行计算。
   - **非阻塞**：在协程等待一个操作完成时，程序可以执行其他任务，而不是停滞在那里。这是通过协程的挂起和恢复实现的。

3. **在 Python 中的使用**：
   - 使用 `async def` 定义协程函数。
   - 使用 `await` 关键字挂起协程的执行，等待异步操作完成。

### 事件循环（Event Loop）
1. **定义**：事件循环是一个编程构造，用于调度和管理程序中的各种事件和操作。在异步编程中，它是运行异步程序和处理异步事件的核心。
2. **功能**：
   - **管理操作**：事件循环负责调度协程和其他异步操作，例如 I/O 事件、定时事件等。
   - **处理回调**：当异步操作完成时，事件循环负责触发相应的回调函数。
   - **循环机制**：事件循环以循环的方式运行，等待和分发事件和操作。

3. **在 Python 中的使用**：
   - `asyncio` 模块提供了事件循环的实现。
   - 使用 `asyncio.run()` 启动事件循环，并执行最高级别的入口协程。
   - 事件循环会运行直到所有的任务都完成，或者被显式停止。

### 协程与事件循环的关系
- 协程依赖于事件循环来管理它们的执行。
- 当协程通过 `await` 挂起时，事件循环可以执行其他任务，直到挂起的操作完成并通过事件循环再次唤醒协程。
- 事件循环负责调度协程的执行，处理它们的挂起与恢复，使得异步编程成为可能。

在 Python 的 `asyncio` 库中，协程和事件循环共同工作，使得编写高效的异步代码成为可能，特别是在涉及大量 I/O 操作的场景中。

`await`, `run_async()`, 和 `asyncio.run()` 是 Python 异步编程中的重要概念和函数，它们在处理异步操作时有不同的作用和用法。以下是它们之间的主要区别：

1. **`await`**:
   - `await` 用于等待一个异步函数（协程）的结果。
   - 它只能在定义为 `async` 的函数内部使用。
   - 当你在一个协程中使用 `await` 时，程序执行将暂停，直到 `await` 后的异步操作完成，然后继续执行后续代码。
   - 示例：
	 ```python
	 async def async_function():
		 result = await some_async_operation()
	 ```

2. **`run_async()`**:
   - `run_async()` 不是 Python 标准库的一部分。Tortoise ORM 提供了一个 `run_async` 函数，用于在非异步环境中运行异步代码。
    - 这个函数接受一个协程作为参数，并运行这个协程。
   
   - 示例（使用 Tortoise ORM）：
	 ```python
	 from tortoise import run_async

	 async def async_function():
		 # 异步操作
		 pass

	 run_async(async_function())
	 ```

3. **`asyncio.run()`**:
   - `asyncio.run()` 是 Python 3.7+ 中引入的一个函数，用于运行最高级别的入口点 "main" 函数，并负责管理异步事件循环。
   - 这个函数将创建一个新的事件循环，并在这个循环里运行传入的协程。
   - 它适用于只运行一次的异步代码。在协程完成后，事件循环将关闭。
   - 在 协程内部(async await ) 不应该使用 `asyncio.run()`。
   

`asyncio.run()`函数的参数应该是一个协程对象，通常是通过调用一个异步函数（使用`async def`定义的函数）来创建的。

   

例如：

```python
async def main():
   # 这里是你的异步代码
   pass

# 运行main协程
asyncio.run(main())
```

在这个例子中，`main`是一个异步函数，调用`main()`会创建一个协程对象，然后`asyncio.run(main())`会运行这个协程。

请注意，`asyncio.run()`会创建一个新的事件循环，运行协程，然后关闭事件循环。如果你的代码已经在一个事件循环中运行，你应该使用`await`来运行协程，而不是`asyncio.run()`。

通常情况下，`await` 用于协程内部等待其他异步操作，`asyncio.run()` 用于程序的入口点，而 `run_async()`（如果是第三方库提供的）用于特定的场景，比如在同步环境中运行异步代码。

## 异步编程

Python 的 `async` 和 `await` 关键字是用于异步编程的一部分，主要出现在 Python 3.5 及以后的版本。这一机制允许 Python 程序以非阻塞的方式执行 I/O 密集型任务，如网络请求、数据库操作等。这对于提高程序性能和响应性特别有用。

### Async

`async` 关键字用于定义一个“异步函数”。这种函数执行时，不会阻塞调用它的线程。相反，它允许程序在等待操作完成时继续执行其他任务。

```python
async def my_async_function():
	# 异步操作
```

### Await

`await` 关键字则用于异步函数内部，用来挂起异步操作的执行。`await` 后面跟的是一个表达式，通常是一个异步函数调用。当程序执行到 `await` 时，它会将控制权交还给事件循环，并暂停该异步函数的进一步执行，直到 `await` 的表达式完成。  
async修饰的函数（也称为异步函数或协程函数）可以不包含await表达式。然而，如果一个async函数没有await表达式，那么它就是一个同步函数，因为它不会挂起执行。  

```python
async def another_async_function():
	await my_async_function()  # 等待 my_async_function 完成
	# my_async_function 完成后的操作
```

### 事件循环

异步编程依赖于事件循环（event loop），它是整个机制的核心。事件循环负责调度异步操作，并在操作完成时恢复相应的异步函数。在 Python 中，`asyncio` 模块提供了管理事件循环的工具。

## 协程

协程（Coroutine）是一种程序组件，它在高级语言中用于并发编程。与传统的子程序（如函数或方法）不同，协程的特点在于它可以在特定位置暂停执行，并在之后从同一点恢复执行。这使得协程非常适合处理异步操作和构建并发应用程序。

协程的关键特性包括：

非阻塞性：协程可以在等待一个操作完成时让出控制权，允许其他协程运行，从而实现非阻塞执行。

协作式多任务：与抢占式多任务（如线程）不同，协程需要明确地让出控制权（通常通过暂停点，如 yield 或 await）。这意味着协程控制其自身的执行，从而简化了同步和状态管理。

高效的异步操作：在处理 I/O 密集型任务（如网络通信或文件操作）时，协程允许程序在等待操作完成时执行其他任务，提高了程序的效率和响应性。

简化的错误处理和资源管理：协程能够使用类似于同步代码的结构来处理错误和管理资源，这简化了异步代码的编写和理解。

### 使用示例
```python
import asyncio

async def hello():
	print('Hello')
	await asyncio.sleep(1)
	print('World')

async def main():
	await hello()

asyncio.run(main())
```

在这个示例中，`hello` 是一个异步函数。它首先打印 "Hello"，然后 `await asyncio.sleep(1)` 挂起函数执行，等待 1 秒。在这段时间内，事件循环可以处理其他任务。1 秒后，事件循环再次唤醒 `hello` 函数，接着打印 "World"。

### 注意事项
- `async` 和 `await` 用于异步编程，主要用于 I/O 密集型任务。对于计算密集型任务，通常需要使用多线程或多进程。  
- `async` 函数必须在异步环境中执行，不能直接调用。它们通常被 `await`、被其他 `async` 函数调用，或通过像 `asyncio.run()` 这样的函数启动。  
- 异步编程在某些情况下可以显著提高性能，但它也引入了额外的复杂性。在设计异步应用时，要特别注意错误处理和数据共享的问题。  
- 当你在 await 表达式之后定义了一些变量，这些变量只有在 await 之后的异步操作完成后才会被赋值。如果在 await 之后、异步操作完成之前的代码尝试访问这些变量，将会发生错误，因为这些变量尚未被定义或赋值。  
- 不能在普通函数中直接调用异步函数。你需要在协程或者其他异步函数中使用await关键字来调用异步函数。如果你需要在普通函数中运行异步函数，你可以使用asyncio.run函数。  

### 异步机制结合多线程

在Python中，如果一个线程被阻塞（例如，它正在等待I/O操作完成或者正在执行一个长时间运行的计算），这不会影响其他线程的执行。这是因为Python的线程是操作系统级别的线程，每个线程都有自己的执行上下文，并且由操作系统调度。

当一个线程被阻塞时，操作系统会将CPU的控制权转移到其他未被阻塞的线程，这样其他线程可以继续执行。这就是所谓的上下文切换。

然而，需要注意的是，由于Python的全局解释器锁（GIL）的存在，Python在任何时候只允许一个线程执行Python字节码。这意味着，尽管Python的线程是操作系统级别的线程，但是它们在执行Python代码时并不是真正的并行执行。这可能会限制Python多线程的性能，特别是在CPU密集型任务中。

如果你的代码中有一个线程被阻塞，并且你希望其他线程能够并行执行，你可能需要考虑使用进程或者异步编程。进程是真正的并行执行，每个进程有自己的Python解释器和内存空间。异步编程使用单线程和事件循环来实现并发，它适合I/O密集型任务。

可以在线程中使用async和await，但是需要注意的是，asyncio库是基于单线程的，它使用协程（coroutine）来实现并发。这意味着，尽管你可以在多线程环境中使用async和await，但是你不能在一个线程中等待另一个线程的协程。  
如果你想在一个线程中运行asyncio事件循环，你需要使用asyncio.run或者loop.run_until_complete来运行你的协程。然后，在另一个线程中，你可以使用asyncio.run_coroutine_threadsafe来安全地运行协程。  

```python
import asyncio
import threading
async def my_coroutine():
	await asyncio.sleep(1)
	print("Coroutine finished")

def run_event_loop():
	asyncio.run(my_coroutine())

### 在一个新的线程中运行事件循环
thread = threading.Thread(target=run_event_loop)
thread.start()
```

在这个例子中，我们在一个新的线程中运行了asyncio事件循环，并在这个事件循环中运行了一个协程。  

在Python中，使用`asyncio`库进行异步编程时，事件循环（Event Loop）是核心概念之一。事件循环负责管理和分发程序中的各种事件，以及在异步程序中调度协程的执行。当你使用`asyncio.get_event_loop()`和`loop.run_until_complete(main())`时，你实际上是在设置和启动一个事件循环来运行你的异步代码。让我们一步步详细解释这个过程。

### `asyncio.get_event_loop()`

- `asyncio.get_event_loop()`函数用于获取当前上下文中的事件循环。
- 如果当前上下文没有事件循环，且在一个主线程中调用，它将创建一个新的事件循环并返回。
- 在子线程中调用时，如果没有已经运行的事件循环，它会抛出一个异常，因为默认情况下子线程不会自动创建事件循环。

### `loop.run_until_complete(main())`

- `loop.run_until_complete(future)`是一个阻塞（blocking）调用，用于运行传入的`future`对象，直到它完成。
- `future`对象可以是一个`asyncio.Future`对象或者是一个通过`async def`定义的协程对象。
- 在这个调用中，`main()`是一个协程。`run_until_complete`方法将会运行事件循环，直到`main()`协程执行完成。在此期间，事件循环会自动处理所有排队的异步任务和回调。
- 一旦`main()`协程完成，`run_until_complete`方法将返回协程的结果，并停止事件循环。
- 如果在`main()`协程执行期间发生了未被处理的异常，它将被抛出到`run_until_complete`方法外部。

### 示例解释

考虑以下代码：

```python
import asyncio

async def main():
    # 异步代码或协程调用
    pass

loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```

在这段代码中：

- 首先，通过`asyncio.get_event_loop()`获取当前线程的事件循环。
- 然后，使用该事件循环的`run_until_complete`方法来运行`main`协程。
- 这个调用会一直阻塞，直到`main()`协程执行完毕。在此期间，事件循环会处理其他异步操作，比如I/O事件、其他协程的调度等。

### 注意

从Python 3.7开始，可以直接使用`asyncio.run(coroutine())`作为启动新异步程序的首选方法，因为它更简洁，自动创建和关闭事件循环，减少了样板代码。例如：

```python
import asyncio

async def main():
    # 异步代码或协程调用
    pass

asyncio.run(main())
```

`asyncio.run(main())`是一个高级API，用于运行最高级别的入口点“main”协程，并自动管理事件循环的生命周期。在大多数情况下，推荐使用`asyncio.run()`，除非你需要更细粒度的控制事件循环，或者在老版本的Python中（3.7以下），你必须手动管理事件循环。

## 新线程中运行 asyncio.run() 的坑

asyncio.run() 是一个阻塞操作，它会运行传入的协程，直到该协程完成。如果你的 start_ws_own_listener(ws_own_url) 协程永远不会结束（例如，它包含一个无限循环），那么 asyncio.run() 将永远不会返回，后面的代码将不会执行。

如果你想要在 start_ws_own_listener(ws_own_url) 运行的同时执行其他代码，你可以使用 asyncio.create_task() 来创建一个新的任务，这个任务会在当前的事件循环中运行。

在新线程中使用 asyncio.run()  ，会报错：There is no current event loop in thread 'Thread-1 (start_ws_own_listener)'.

asyncio.run()函数是一个高级函数，用于在主线程中执行协程。它会自动创建一个新的事件循环，然后运行你提供的协程，最后关闭事件循环。但是，asyncio.run()假设它是在主线程中被调用的，如果在其他线程中调用asyncio.run()，它会尝试获取当前线程的事件循环，如果当前线程没有事件循环，就会抛出错误。

Python的asyncio库中的事件循环方法，它们用于在事件循环中运行协程。

loop.run_until_complete(coro): 这个方法会运行传入的协程，然后等待它完成。如果传入的是一个Future对象，它会运行事件循环，直到Future对象完成。如果传入的是一个协程，它会自动将协程包装成一个Task（也就是一个Future对象），然后运行事件循环，直到Task完成。

loop.create_task(coro): 这个方法会将传入的协程包装成一个Task（也就是一个Future对象），然后将这个Task添加到事件循环中。这个Task会在事件循环运行的过程中被调度和执行。这个方法不会运行事件循环，你需要自己手动运行事件循环（例如通过调用loop.run_forever()）。

loop.run_forever(): 这个方法会启动事件循环，让它一直运行下去，直到loop.stop()被调用。在事件循环运行的过程中，它会自动调度和执行事件循环中的所有Task。- 协程任务将执行其逻辑，然后完成，不会自动重复执行。**回调函数**: 你可以通过`loop.call_soon()`、`loop.call_later()`、`loop.call_at()`等方法提交回调函数给事件循环。这些回调函数在被调用后也不会自动重复执行。

loop.stop(): 这个方法会停止事件循环。当事件循环正在运行时（例如在loop.run_forever()中），你可以调用这个方法来停止事件循环。这个方法不会立即停止事件循环，而是会在当前的迭代结束后停止事件循环。这意味着在调用loop.stop()后，事件循环可能还会运行一段时间，直到当前的迭代结束。

loop.close(): 这个方法会关闭事件循环。一旦事件循环被关闭，它就不能再被用来运行协程了。如果你尝试在一个已经被关闭的事件循环上调用loop.run_until_complete()或者loop.run_forever()，Python会抛出RuntimeError: Event loop is closed错误。在关闭事件循环之前，你应该确保所有的任务都已经完成，否则这些未完成的任务会被取消。

loop.stop()只是暂时停止事件循环，你可以在之后再次启动它；而loop.close()会永久关闭事件循环，一旦事件循环被关闭，你就不能再使用它了。

如果你需要在新的线程中运行协程，你需要手动创建一个事件循环，然后使用这个事件循环来运行你的协程。以下是一个例子：

'''Python

import asyncio

import threading

def start_ws_own_listener():

	loop = asyncio.new_event_loop()  # 创建一个新的事件循环

	asyncio.set_event_loop(loop)  # 设置这个事件循环为当前线程的事件循环

	# 定义协程
	async def coro1(): pass  # 需要并行
	async def coro2(): pass  # 需要并行

	async def coro3(): pass  # 需要串行
	async def coro4(): pass  # 需要串行

   # 并行运行 coro1() 和 coro2()

   loop.run_until_complete(asyncio.gather(coro1(), coro2()))

	# 串行运行 coro3() coro4()等待它们完成
	loop.run_until_complete(coro3())
	loop.run_until_complete(coro4())

	# 串行运行 coro3() coro4()等待它们完成
	loop.create_task(coro3()) # 创建任务，不会立即执行
	loop.create_task(coro4()) # 创建任务，不会立即执行
	loop.run_forever()  # 不接受任何参数。会阻塞当前线程，但是运行的过程中，事件循环可以并发地运行多个任务。任务中需要调用 loop.stop() 才能跳出 loop.run_forever() 
	loop.close()  # 关闭事件循环

threading.Thread(target=start_ws_own_listener).start()  # 在新的线程中运行 start_ws_own_listener 函数

'''

loop.stop()通常在你想要停止事件循环的地方调用。具体放在哪个结构中，取决于你的程序的逻辑。

例如，如果你有一个无限循环的任务，你可能会在某个条件满足时调用loop.stop()来停止事件循环：

async def infinite_task():

    while True:

        # do something

        if some_condition:

            loop.stop()

在这个例子中，当some_condition满足时，loop.stop()会被调用，事件循环会在当前的迭代结束后停止。

另一个常见的用法是在接收到某个信号（例如SIGINT或SIGTERM）时停止事件循环：

)

在这个例子中，当程序接收到SIGINT或SIGTERM信号时，stop_on_signal函数会被调用，这个函数会调用loop.stop()来停止事件循环。

请注意，loop.stop()只是停止事件循环，它不会关闭事件循环。在调用loop.stop()后，你还需要调用loop.close()来关闭事件循环。

不能在loop.run_forever()运行的同时再调用loop.run_until_complete()。这是因为loop.run_until_complete()和loop.run_forever()都是用来启动事件循环的，它们会阻塞当前线程，直到相应的条件满足（对于run_until_complete是传入的协程完成，对于run_forever是loop.stop()被调用）。

如果你在loop.run_forever()运行的任务中调用loop.run_until_complete()，你会得到一个错误，提示事件循环已经在运行。

如果你需要在loop.run_forever()运行的过程中执行一个协程，你可以使用asyncio.ensure_future()或loop.create_task()来创建一个新的任务，这个任务会在事件循环运行的过程中被调度和执行。

### 事件循环的嵌套执行

首先定义了两个协程coro4()和coro5()。在coro4()中，我们使用loop.create_task()来创建一个新的任务，这个任务会在事件循环运行的过程中被调度和执行。

import asyncio

async def coro5():

    await asyncio.sleep(1)

    print("coro5 finished")

async def coro4(loop):

    loop.create_task(coro5())

    print("coro4 finished")

loop = asyncio.get_event_loop()

loop.run_until_complete(coro4(loop))

在这个例子中，当我们调用loop.run_until_complete(coro4(loop))时，coro4()会被执行。在coro4()中，我们创建了一个新的任务coro5()，这个任务会在事件循环运行的过程中被调度和执行。

然而，你会注意到，即使coro5()被创建为一个新的任务，但是当coro4()完成时，事件循环也会立即停止，coro5()可能还没有机会被执行。这是因为loop.run_until_complete()会在传入的协程完成后立即停止事件循环。

如果你想要在coro4()完成后继续运行事件循环，直到所有的任务都完成，你可以使用loop.run_forever()，并在所有的任务都完成后调用loop.stop()来停止事件循环。例如：

import asyncio

async def coro5():

    await asyncio.sleep(1)

    print("coro5 finished")

async def coro4(loop):

    loop.create_task(coro5())

    print("coro4 finished")

loop = asyncio.get_event_loop()

loop.run_until_complete(coro4(loop))

loop.run_forever()

在这个例子中，即使coro4()完成，事件循环也会继续运行，直到coro5()完成。

## websocket 异步

在`websocket`库中，`on_open`，`on_message`，`on_error`和`on_close`这些回调函数通常是同步函数，而不是异步函数。这是因为`websocket`库是基于同步的Python `socket`库构建的，而不是基于异步的`asyncio`库。

这意味着，即使你在异步函数中创建和运行`WebSocketApp`，这些回调函数也应该是同步的。如果你需要在这些回调函数中执行异步操作，你需要在这些函数中创建一个新的事件循环，然后在这个事件循环中运行你的异步代码。但是请注意，这可能会导致代码变得复杂，并可能导致性能问题，因为创建和管理多个事件循环通常会有一些开销。

如果你需要在WebSocket通信中使用异步编程，你可能需要考虑使用一个支持异步的WebSocket库，如`websockets`或`aiohttp`。这些库是基于`asyncio`构建的，可以更好地支持异步操作。

### websocket有几个包：

1、功能不全的 websocket

导入的时候：

import websocket

2、websocket-client  不带服务端，不支持异步

pip3 install websocket-client

导入的时候也是：

import websocket

3、websockets 带服务端，支持异步

pip3 install websockets

导入的时候：

import websockets

## logger

logging 模块的 info、error 和 warning 方法都是同步的，不是异步的，所以你不能在它们前面使用 await 关键字。
