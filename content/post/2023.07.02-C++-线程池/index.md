+++
author = "baoguli"
title = "线程池"
date = "2023-7-2"
description = "简单聊聊基于 c++ 实现的线程池"
tags = [
    "c++",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  


线程池是一种高级的并发编程模式，它可以在一个固定数量的线程中执行多个任务，而不需要每次创建和销毁线程。

线程池的基本思想是：

- 创建一个任务队列，用来存放需要执行的任务。
- 创建一个线程集合，用来从任务队列中取出任务并执行。
- 创建一个管理器，用来向任务队列中添加任务，以及控制线程的启动和停止。

为了实现这个思想，我们需要使用一些 C++11 提供的多线程编程工具，比如：

- std::thread：表示一个可执行的线程对象。
- std::mutex：表示一个互斥锁，用来保护共享数据的访问。
- std::condition_variable：表示一个条件变量，用来让线程在某个条件满足之前等待，或者通知其他线程某个条件已经满足。
- std::packaged_task：表示一个封装了函数对象的任务，可以将执行结果和返回值分离。
- std::future：表示一个异步操作的结果，可以用来获取任务的返回值或者等待任务完成。

下面是一个简单的线程池的示例代码：

```cpp
// 线程池类
class ThreadPool {
public:
    // 构造函数，创建指定数量的线程
    ThreadPool(size_t num_threads) {
        start(num_threads);
    }
    // 析构函数，停止所有线程
    ~ThreadPool() {
        stop();
    }

    // 向任务队列中添加任务，并返回一个future对象
    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args) -> std::future<std::invoke_result_t<F, Args...>> {
        // 获取任务的返回类型
        using return_type = std::invoke_result_t<F, Args...>;

        // 创建一个packaged_task对象，用来封装函数对象和返回值
        auto task = std::make_shared<std::packaged_task<return_type()>>(std::bind(std::forward<F>(f), std::forward<Args>(args)...));

        // 获取future对象，用来获取返回值或者等待任务完成
        std::future<return_type> result = task->get_future();

        std::unique_lock<std::mutex> lock(queue_mutex);

        // 将任务添加到队列中
        tasks.emplace([task](){ (*task)(); });

        // 通知一个等待的线程
        condition.notify_one();

        return result;
    }

private:
    // 任务队列
    std::queue<std::function<void()>> tasks;

    // 互斥锁
    std::mutex queue_mutex;

    // 条件变量
    std::condition_variable condition;

    // 线程集合
    std::vector<std::thread> workers;

    // 停止标志
    std::atomic<bool> stop_flag;

    // 启动指定数量的线程，并让每个线程执行thread_loop函数
    void start(size_t num_threads) {
        for (size_t i = 0; i < num_threads; ++i) {
            workers.emplace_back([this] {
                this->thread_loop();
            });
        }
    }

    // 停止所有线程
    void stop() {
        // 设置停止标志为true
        stop_flag.store(true);

        // 唤醒所有等待的线程
        condition.notify_all();

        // 等待所有线程结束
        for (std::thread& worker : workers) {
            if (worker.joinable()) {
                worker.join();
            }
        }
    }

    // 每个线程执行的循环函数
    void thread_loop() {
        while (true) {
            // 定义一个空的任务
            std::function<void()> task;

            std::unique_lock<std::mutex> lock(queue_mutex);

            // 等待条件变量，直到有任务或者停止标志为true
            condition.wait(lock, [this] {
                return !this->tasks.empty() || this->stop_flag.load();
            });

            // 如果停止标志为true，退出循环
            if (this->stop_flag.load()) {
                break;
            }

            // 取出一个任务
            task = std::move(this->tasks.front());
            this->tasks.pop();

            lock.unlock();

            // 执行任务
            task();
        }
    }
};
```

每个线程都会不断地循环接收任务执行任务，当然，在没有任务的时候，线程会被阻塞，在条件满足时，线程才会执行：

```cpp
condition.wait(lock, [this] {
    return !this->tasks.empty() || this->stop_flag.load();
});
```

条件变量是一种多线程编程中常用的同步机制，它可以让一个线程在某个条件满足之前**阻塞**等待，或者通知其他线程某个条件已经满足。 

条件变量通常和互斥锁配合使用，因为条件的判断和修改往往涉及到共享数据的访问，需要保证**原子性**和**可见性**。 

在这段代码中，条件变量是 condition，互斥锁是 lock，条件是 !this->tasks.empty() || this->stop_flag.load()，也就是任务队列不为空或者停止标志为true。

具体的流程如下：

- 首先，线程上锁，保护任务队列。
- 然后，线程调用 condition.wait(lock, [this] {return !this->tasks.empty() || this->stop_flag.load();}); 方法，表示等待条件变量。
- 这个方法会做以下几件事：
  - 它会检查条件是否满足，如果满足，就直接返回，继续执行后面的代码。
  - 如果不满足，它会将当前线程放入一个等待队列中，并解锁互斥锁，让其他线程有机会获取互斥锁和修改条件。
  - 然后，它会让当前线程进入阻塞状态，等待被唤醒。
  - 当其他线程调用 condition.notify_one() 或者 condition.notify_all() 方法时，它会从等待队列中选择一个或者多个线程，并将它们唤醒。
  - 被唤醒的线程会重新获取互斥锁，并再次检查条件是否满足。
  - 如果满足，就返回，并继续执行后面的代码。
  - 如果不满足，就重复上述过程，直到条件满足或者发生异常。

所以，这一段代码的作用就是让线程在任务队列为空或者停止标志为false时等待，而在任务队列不为空或者停止标志为true时继续。



用户向线程池添加任务的 enqueue 函数，它是一个公开接口，返回一个future对象，用来获取任务的返回值或者等待任务完成。

这个函数的模板参数有两个：

- F：表示可调用对象的类型，比如函数，函数指针，仿函数，lambda 表达式等。
- Args：表示可变参数包的类型，也就是可调用对象的参数类型。

这个方法的返回类型是 std::future<std::invoke_result_t<F, Args...>>，它表示一个异步操作的结果，其中 std::invoke_result_t<F, Args...>是可调用对象在给定参数时的返回类型。

这个方法的具体流程如下：

- 创建一个 std::packaged_task<return_type()> 对象，它是一个封装了可调用对象和返回值的任务。它的构造函数接受一个 std::bind 表达式，将可调用对象和参数绑定在一起。这里使用了 std::forward 来保持参数的完美转发。
- 接着，调用 packaged_task 对象的 get_future 方法，获取一个 std::future<return_type> 对象，它可以用来获取任务的返回值或者等待任务完成。
- 然后，上锁，保护任务队列。任务队列是一个 std::queue<std::function<void()>> 对象，它存放了等待执行的任务。
- 接着，将 packaged_task 对象添加到任务队列中。由于任务队列只接受 std::function<void()> 类型的对象，所以这里使用了一个 lambda 表达式来适配。lambda 表达式捕获了 packaged_task 对象的智能指针，并调用它。
- 然后，调用条件变量的 notify_one 方法，通知一个等待的线程有新的任务可以执行。
- 最后，返回 future 对象给用户。

所以，这段代码的作用就是让用户可以向线程池中添加任意类型和参数的任务，并得到一个future对象来控制异步操作。



这个线程池的使用方法很简单，只需要创建一个 ThreadPool 对象，然后调用 enqueue 方法，传入想要执行的函数对象和参数，就可以得到一个 future 对象，用来获取返回值或者等待任务完成。例如：

```cpp
// 创建一个线程池，包含4个线程
ThreadPool pool(4);

// 向线程池中添加一个任务，计算1+2，并返回一个future对象
auto result = pool.enqueue([](int a, int b) {
    return a + b;
}, 1, 2);

// 获取任务的返回值，或者等待任务完成
std::cout << "1 + 2 = " << result.get() << std::endl;
```


在上面的代码中，创建了一个线程池，包含四个线程。用户向线程池中添加任务，这些任务会被放入一个任务队列中，等待被执行。每个线程都会在一个循环中，从任务队列中取出一个任务，并执行它。如果任务队列为空，线程就会等待条件变量的通知，直到有新的任务或者停止标志为 true。

所有的任务由这四个线程轮流执行。当然，具体哪个线程执行哪个任务，是由操作系统的调度算法决定的，我们无法预测或者控制。我们只能保证每个任务都会被某个线程执行，并且不会有两个线程同时执行同一个任务。


开多少线程是由**多个因素**决定的，比如CPU的核数，任务的数量，任务的类型，任务的执行时间，内存的限制等等。

一般来说，开多少线程需要**平衡**两方面的考虑：

- 一方面，如果开的线程太少，那么可能会造成 CPU 的**利用率低**，有些核心会空闲，而有些任务会等待很久才能被执行。
- 另一方面，如果开的线程太多，那么可能会造成系统的**开销大**，因为每个线程都需要占用一定的内存和时间来创建和销毁，而且过多的线程会导致频繁的上下文切换和竞争，降低性能。

所以，开多少线程需要根据具体的情况来**动态调整**，以达到最优的效果。

有一些经验性的公式可以帮助估算一个合理的线程数，比如：

- 如果任务都是 CPU 密集型的，那么线程数可以设置为 CPU 核数+1。
- 如果任务都是 IO 密集型的，那么线程数可以设置为 CPU 核数*2。
- 如果任务既有 CPU 密集型又有 IO 密集型的，那么线程数可以设置为 CPU 核数/(1-阻塞系数)，其中阻塞系数是一个0到1之间的小数，表示任务在等待 IO 操作完成的时间占总时间的比例。

当然，这些公式只是一个参考，并不一定适用于所有的场景。我们还需要根据实际的测试和反馈来调整线程数，以达到最佳的性能和效率。
