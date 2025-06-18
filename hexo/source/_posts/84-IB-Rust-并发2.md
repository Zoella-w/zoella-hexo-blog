---
title: IB-Rust-并发2
date: 2025-06-16 21:14:26
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson26](https://github.com/Zoella-w/IB-Rust/tree/main/26_concurrent2)

## 使用共享来实现并发

- Go 语言：**不要用共享内存来通信，要用通信来共享内存**
- Rust 支持通过共享状态来实现并发
- Channel 类似单所有权：一旦将值的所有权转移至 Channeel，就无法使用它了
- 共享内存并发类似多所有权：多个线程可以同时访问同一块内存

### 使用 Mutex 来每次只允许一个线程来访问数据

- Mutex 是 mutualexclusion（互斥锁）的简写
- 在同一时刻，Mutex 只允许一个线程来访问某些数据
- 想要访问数据：
    - 线程必须首先获取互斥锁（lock）
    - lock 数据结构是 mutex 的一部分，它能跟踪谁对数据拥有独占访问权

### Mutex 的两条规则

- 在使用数据之前，必须尝试获取锁（lock）
- 使用完 mutex 所保护的数据，必须对数据进行解锁，以便其它线程可以获取锁

### `Mutex<T>` 的 API

通过 `Mutex::new(数据)` 来创建 `Mutex<T>`（智能指针）

- 访问数据前，通过 lock 方法来获取锁
  - 会阻塞当前线程
  - lock 有可能会失败
  - 返回的是 MutexGuard（智能指针，实现了 Deref 和 Drop 两个 trait）

```rust
let m = Mutex::new(5); // 5 就是要保护的数据
{
    let mut num = m.lock().unwrap();
    *num = 6;
    // 因为 MutexGuard 实现了 Deref 和 Drop Trait
    // 在离开这个作用域的时候会自动释放掉，就会自动解锁，所以不需要我们解锁
}
println!("m={:?}", m); // m=Mutex { data: 6, poisoned: false, .. }
```

## 使用 `Arc<T>` 来进行原子引用记数

- `Arc<T>` 和 `Rc<T>`类似，它可以用于并发情景
  - `A:atomic` 原子的
- 为什么所有的基础类型都不是原子的？为什么标准库类型不默认使用 `Arc<T>`？
  - 这是因为需要性能作为代价
- `Arc<T>` 和 `RC<T>` 的 API 是相同的

```rust
// let counter = Rc::new(Mutex::new(0));
let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];
for _ in 0..10 {
    // let counter = Rc::clone(&counter);
    // error: `Rc<Mutex<i32>>` cannot be sent between threads safely
    // Rc 没有实现 Send Trait
    // 所以在智能指针中，说 Rc 只能在单线程中使用
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    });
    handles.push(handle);
}
for handle in handles {
    handle.join().unwrap();
}
println!("Result: {}", *counter.lock().unwrap()); // error: borrow of moved value: `counter`
```

## `RefCell<T>`/`Rc<T>` vs `Mutex<T>`/`Arc<T>`

- `Mutex<T>` 提供了内部可变性，和 `Cell` 家族一样
- 我们使用 `RefCell<T>` 来改变 `Rc<T>` 里面的内容
- 我们使用 `Mutex<T>` 来改变 `Arc<T>` 里面的内容
- 注意：`Mutex<T>` 有死锁风险

## Send 和 Sync trait

- Rust 语言的并发特性较少，目前讲的并发特性都来自标准库（而不是语言本身）
- 无需局限于标准库的并发，可以自己实现并发
- 在 Rust 语言中有两个并发概念：
  - `std::marker::Sync` 和 `std::marker::Send` 这两个 trait

### Send：允许线程间转移所有权

- 实现 Send trait 的类型可在线程间转移所有权
- Rust 中几乎所有的类型都实现了 Send
  - 但 `Rc<T>` 没有实现 Send，它只用于单线程情景
- 任何完全由 Send 类型组成的类型也被标记为 Send
- 除了原始指针之外，几乎所有的基础类型都是 Send

### Sync：允许从多线程访问

- 实现 Sync 的类型可以安全的被多个线程引用
- 也就是说：**如果 T 是 Sync，那么 &T 就是 Send**
  - 引用可以被安全的送往另一个线程
- 基础类型都是 Sync
- 完全由 Sync 类型组成的类型也是 Sync
  - 但 `Rc<T>` 不是 Sync 的
  - `RefCell<T>` 和 `Cell<T>` 家族也不是 Sync 的
  - 而 `Mutex<T>` 是 Sync 的

## 课后习题：实现一个多线程任务调度器

- 任务描述
  - 你需要编写一个简单的多线程任务调度器，它能够接收多个任务，并将这些任务分发到多个工作线程中执行。调度器使用 Channel 进行任务的分发和结果的收集。你需要使用 Rust 的 Send 和 Sync 特性来确保任务调度器在多线程环境中的安全性。
- 具体要求
- 任务结构：
  - 定义一个 Task 结构体，表示需要执行的任务。任务包含一个唯一的 id 和一个用于执行的闭包。
- 调度器结构：
  - 创建一个 Scheduler 结构体，包含一个任务队列和一个线程池。调度器应当使用 channel 来分发任务到不同的工作线程。
- 功能实现：
  - 调度器应当具有以下功能：
  - 添加任务：向调度器添加一个任务。
  - 启动调度器：启动多个线程，开始从任务队列中获取任务并执行。
  - 获取结果：在所有任务完成后，收集并打印每个任务的执行结果。
- 多线程安全：
  - 通过使用 Arc 和 Mutex 确保任务队列在多个线程之间的安全访问。
  - 确保任务的结果能够正确地在线程之间传递和收集。

### 问题提示

- 任务队列：
  - 使用 Mutex 来保护任务队列，确保多个线程不会同时修改队列中的数据。
  - 使用 Arc 来共享任务队列的所有权，使得多个线程能够访问同一个任务队列。
- 任务分发：
  - 使用 channel 来将任务的完成状态发送回主线程，从而可以在主线程中收集和打印任务完成的结果。
- 线程池：
  - 通过循环创建多个工作线程，每个线程从任务队列中取出任务并执行。线程池的大小可以通过 Scheduler 的构造函数来指定。
- 任务执行：
  - 每个任务都应该是一个闭包，使用 `Box<dyn FnOnce()>` 将其存储在 Task 结构体中。


```rust
use std::collections::VecDeque;
use std::sync::{
    Arc, Mutex,
    mpsc::{self, Receiver, Sender},
};
use std::thread;
use std::time::Duration;

/// 任务结构体
struct Task {
    id: usize,
    // 任务执行逻辑（装箱闭包）
    // 1、基础闭包类型: FnOnce() -> String
    //   - FnOnce(): 可被调用一次的闭包
    //   - -> String: 返回字符串结果
    // 2、+ Send: 添加线程安全约束，可在线程间安全传递
    // 3、dyn: 允许包装多种不同类型的闭包实现
    // 4、Box<...>: 堆内存分配，将动态大小的特征对象固定在堆内存
    f: Box<dyn FnOnce() -> String + Send>,
}

impl Task {
    /// 创建新任务
    // 'static: 无外部引用依赖
    fn new(id: usize, f: impl FnOnce() -> String + Send + 'static) -> Self {
        Task { id, f: Box::new(f) }
    }
    /// 执行任务
    fn execute(self) -> String {
        (self.f)()
    }
}

/// 调度器结构体
struct Scheduler {
    // 1、VecDeque：双端队列数据结构，方便先进先出、头部移除
    // 2、​​Mutex（互斥锁）​：确保同时只有一个线程访问队列
    // 3、​Arc(原子引用计数)​​：实现跨线程共享所有权，Arc::clone() 会增加引用计数
    task_queue: Arc<Mutex<VecDeque<Task>>>, // 线程安全任务队列
    result_sender: Sender<(usize, String)>, // 结果发送端
    result_receiver: Option<Receiver<(usize, String)>>, // 结果接收端
    threads: Vec<thread::JoinHandle<()>>,   // 工作线程句柄
}

impl Scheduler {
    /// 创建新的调度器
    fn new(thread_count: usize) -> Self {
        // 创建用于传递结果的通道
        let (tx, rx) = mpsc::channel();
        Scheduler {
            task_queue: Arc::new(Mutex::new(VecDeque::new())),
            result_sender: tx,
            result_receiver: Some(rx),
            threads: Vec::with_capacity(thread_count), // 预分配线程存储空间
        }
    }

    /// 添加任务到调度器
    fn add_task(&mut self, id: usize, task: impl FnOnce() -> String + Send + 'static) {
        let mut queue = self.task_queue.lock().unwrap(); // 获取队列锁
        queue.push_back(Task::new(id, task)); // 添加新任务
    }

    /// 启动调度器和工作线程
    fn run(&mut self) {
        let result_sender = self.result_sender.clone();
        let task_queue = Arc::clone(&self.task_queue);
        let thread_count = self.threads.capacity();
        // 创建工作线程池
        for _ in 0..thread_count {
            let task_queue = Arc::clone(&task_queue);
            let result_sender = result_sender.clone();
            // 创建并启动工作线程
            let handle = thread::spawn(move || {
                loop {
                    // 从任务队列中获取任务
                    let task = {
                        let mut queue = task_queue.lock().unwrap();
                        queue.pop_front()
                    };
                    match task {
                        Some(task) => {
                            // 先获取任务ID再执行
                            let id = task.id;
                            let result = task.execute();

                            // 发送结果到通道
                            result_sender.send((id, result)).unwrap();
                        }
                        None => {
                            // 队列为空，检查是否需要继续运行
                            let queue = task_queue.lock().unwrap();
                            if queue.is_empty() {
                                // 短暂睡眠后重试
                                thread::sleep(Duration::from_millis(10));
                            }
                        }
                    }
                }
            });
            self.threads.push(handle);
        }
    }

    /// 等待任务完成并收集结果
    fn wait_completion(&mut self) -> Vec<(usize, String)> {
        // 首先收集所有任务ID
        let task_count = {
            let queue = self.task_queue.lock().unwrap();
            queue.len()
        };
        // 从通道接收结果
        let receiver = self.result_receiver.take().unwrap();
        let mut results = Vec::with_capacity(task_count);
        // 等待并收集所有结果
        for _ in 0..task_count {
            match receiver.recv() {
                Ok((id, result)) => {
                    // 修复处：先复制/借用再移动，避免所有权问题
                    println!("任务 {} 完成: {}", id, &result);
                    results.push((id, result));
                }
                Err(e) => {
                    println!("接收结果出错: {}", e);
                    break;
                }
            }
        }
        // 清除任务队列
        let mut queue = self.task_queue.lock().unwrap();
        queue.clear();
        results
    }
}

fn main() {
    // 1. 初始化调度器（4个工作线程）
    let mut scheduler = Scheduler::new(4);
    // 2. 添加任务（模拟10个耗时任务）
    for i in 0..10 {
        scheduler.add_task(i, move || {
            // 计算任务耗时（动态变化），并转换为u64类型
            let sleep_ms = 200 + (i * 50) % 300;
            let sleep_duration = Duration::from_millis(sleep_ms as u64);
            thread::sleep(sleep_duration);
            // 返回格式化任务结果
            format!("任务 {} 处理完成 (耗时 {}ms)", i, sleep_ms)
        });
    }
    // 3. 启动调度器（创建工作线程）
    scheduler.run();
    // 4. 等待任务完成并收集结果
    let results = scheduler.wait_completion();
    // 5. 输出最终汇总
    println!("\n所有任务完成结果:");
    for (id, result) in results {
        println!("任务 {}: {}", id, result);
    }
}
```

### 完善进阶

- 任务优先级：扩展调度器，使其能够按照任务的优先级顺序执行。
- 任务取消：实现任务的取消功能，当任务队列中存在未完成的任务时，支持中止这些任务的执行。
- 结果收集：扩展调度器，使其能够返回每个任务的执行结果，而不仅仅是打印任务完成状态。

```rust
use std::cmp;
use std::collections::BinaryHeap;
use std::sync::{
    Arc,
    Mutex,
    atomic::{AtomicBool, Ordering}, // 明确从atomic模块导入Ordering
    mpsc::{self, Receiver, Sender},
};
use std::thread;
use std::time::Duration; // 导入标准库的cmp模块

/// 带优先级的任务结构体
struct Task {
    id: usize,
    priority: u32,                         // 任务优先级，数值越高优先级越高
    cancelled: Arc<AtomicBool>,            // 任务取消标志
    f: Box<dyn FnOnce() -> String + Send>, // 移除了Debug trait要求
}

impl Task {
    /// 创建新任务
    fn new(
        id: usize,
        priority: u32,
        cancelled: Arc<AtomicBool>,
        f: impl FnOnce() -> String + Send + 'static,
    ) -> Self {
        Task {
            id,
            priority,
            cancelled,
            f: Box::new(f),
        }
    }

    /// 执行任务
    fn execute(self) -> Option<String> {
        // 检查任务是否被取消，使用正确的Ordering变体
        if self.cancelled.load(Ordering::Relaxed) {
            return None;
        }
        // 执行任务并返回结果
        Some((self.f)())
    }
}

/// 为任务实现排序特性，优先级高的任务排在前面
impl PartialEq for Task {
    fn eq(&self, other: &Self) -> bool {
        self.priority == other.priority
    }
}

impl Eq for Task {}

impl PartialOrd for Task {
    fn partial_cmp(&self, other: &Self) -> Option<cmp::Ordering> {
        // 使用正确的路径
        Some(self.cmp(other))
    }
}

impl Ord for Task {
    fn cmp(&self, other: &Self) -> cmp::Ordering {
        // 使用正确的路径
        // 注意：BinaryHeap是最大堆，所以优先级高的任务应该排在前面
        // 因此返回other和self的比较结果
        other.priority.cmp(&self.priority)
    }
}

/// 增强版调度器结构体
struct Scheduler {
    task_queue: Arc<Mutex<BinaryHeap<Task>>>, // 使用最大堆实现优先级队列
    result_sender: Sender<(usize, Option<String>)>, // 结果发送端（包含任务取消情况）
    result_receiver: Option<Receiver<(usize, Option<String>)>>, // 结果接收端
    threads: Vec<thread::JoinHandle<()>>,     // 工作线程句柄
    stop_flag: Arc<AtomicBool>,               // 停止所有线程的标志
    cancellation_flags: Arc<Mutex<Vec<Arc<AtomicBool>>>>, // 所有任务的取消标志
    thread_count: usize,                      // 存储线程数量
}

impl Scheduler {
    /// 创建新的调度器
    fn new(thread_count: usize) -> Self {
        // 创建用于传递结果的通道
        let (tx, rx) = mpsc::channel();
        Scheduler {
            task_queue: Arc::new(Mutex::new(BinaryHeap::new())),
            result_sender: tx,
            result_receiver: Some(rx),
            threads: Vec::with_capacity(thread_count),
            stop_flag: Arc::new(AtomicBool::new(false)),
            cancellation_flags: Arc::new(Mutex::new(Vec::new())),
            thread_count, // 存储线程数量
        }
    }

    /// 添加任务到调度器
    fn add_task(
        &mut self,
        id: usize,
        priority: u32,
        task: impl FnOnce() -> String + Send + 'static,
    ) -> Arc<AtomicBool> {
        // 创建任务取消标志
        let cancelled = Arc::new(AtomicBool::new(false));
        // 存储取消标志以便后续管理
        let mut flags = self.cancellation_flags.lock().unwrap();
        flags.push(Arc::clone(&cancelled));
        drop(flags);
        let mut queue = self.task_queue.lock().unwrap();
        queue.push(Task::new(id, priority, Arc::clone(&cancelled), task));
        // 返回取消标志，允许外部取消此任务
        cancelled
    }

    /// 启动调度器和工作线程
    fn run(&mut self) {
        let result_sender = self.result_sender.clone();
        let task_queue = Arc::clone(&self.task_queue);
        let stop_flag = Arc::clone(&self.stop_flag);
        // 创建工作线程池
        for _ in 0..self.thread_count {
            // 使用存储的线程数量
            let task_queue = Arc::clone(&task_queue);
            let result_sender = result_sender.clone();
            let stop_flag = Arc::clone(&stop_flag);
            // 创建并启动工作线程
            let handle = thread::spawn(move || {
                while !stop_flag.load(Ordering::Relaxed) {
                    // 从任务队列中获取任务
                    let task = {
                        let mut queue = task_queue.lock().unwrap();
                        queue.pop()
                    };
                    match task {
                        Some(task) => {
                            // 先获取任务ID再执行
                            let id = task.id;
                            let result = task.execute();
                            // 发送结果到通道，包含任务取消情况
                            result_sender.send((id, result)).unwrap();
                        }
                        None => {
                            // 短暂睡眠后重试，避免忙等待消耗CPU
                            thread::sleep(Duration::from_millis(10));
                        }
                    }
                }
            });
            self.threads.push(handle);
        }
    }

    /// 取消所有待处理任务
    fn cancel_all(&self) {
        // 设置全局取消标志
        self.stop_flag.store(true, Ordering::Relaxed);
        // 设置所有任务的取消标志
        let flags = self.cancellation_flags.lock().unwrap();
        for flag in flags.iter() {
            flag.store(true, Ordering::Relaxed);
        }
    }

    /// 等待任务完成并收集结果
    fn wait_completion(&mut self) -> Vec<(usize, Option<String>)> {
        // 获取任务总数
        let task_count = {
            let queue = self.task_queue.lock().unwrap();
            queue.len()
        };
        // 从通道接收结果
        let receiver = self.result_receiver.take().unwrap();
        let mut results = Vec::with_capacity(task_count);
        // 等待并收集所有结果
        for _ in 0..task_count {
            match receiver.recv() {
                Ok((id, result)) => {
                    if let Some(ref res) = result {
                        println!("任务 {} 完成: {}", id, res);
                    } else {
                        println!("任务 {} 已取消", id);
                    }
                    results.push((id, result));
                }
                Err(e) => {
                    println!("接收结果出错: {}", e);
                    break;
                }
            }
        }
        // 清除任务队列和取消标志
        {
            let mut queue = self.task_queue.lock().unwrap();
            queue.clear();
            let mut flags = self.cancellation_flags.lock().unwrap();
            flags.clear();
        }
        results
    }

    /// 优雅关闭调度器
    fn shutdown(&mut self) {
        // 取消所有任务
        self.cancel_all();
        // 等待所有线程结束
        while let Some(handle) = self.threads.pop() {
            handle.join().unwrap_or_else(|_| println!("线程终止出错"));
        }
    }
}

fn main() {
    // 1. 初始化调度器（4个工作线程）
    let mut scheduler = Scheduler::new(4);
    // 2. 添加不同优先级的任务
    let mut task_flags = Vec::new();
    // 添加10个不同优先级的任务
    for i in 0..10 {
        // 生成优先级，并将usize转换为u32
        let priority = ((i % 3 + 1) * 10) as u32; // 确保结果为u32类型
        // 获取取消标志并存储
        let flag = scheduler.add_task(i, priority, move || {
            // 计算任务耗时（动态变化），并转换为u64类型
            let sleep_ms = 200 + (i * 50) % 300;
            let sleep_duration = Duration::from_millis(sleep_ms as u64);
            thread::sleep(sleep_duration);
            // 返回格式化任务结果
            format!("任务 {} 处理完成 (耗时 {}ms)", i, sleep_ms)
        });
        task_flags.push((i, flag));
    }
    // 3. 启动调度器（创建工作线程）
    scheduler.run();
    // 4. 模拟中途取消部分任务
    thread::sleep(Duration::from_millis(200));
    println!("\n取消部分任务...");
    for (id, flag) in &task_flags {
        // 取消ID为奇数的任务
        if id % 2 == 1 {
            flag.store(true, Ordering::Relaxed);
            println!("已取消任务 {}", id);
        }
    }
    // 5. 等待任务完成并收集结果
    let results = scheduler.wait_completion();
    // 6. 输出任务执行汇总
    println!("\n任务执行结果汇总:");
    println!(
        "{: <6} {: <8} {: <8} {}",
        "任务ID", "优先级", "状态", "结果"
    );
    println!("{}", "-".repeat(50));
    for (id, result) in results {
        // 查找任务的优先级
        let priority = task_flags
            .iter()
            .find(|(task_id, _)| *task_id == id)
            .and_then(|(_, flag)| {
                scheduler
                    .cancellation_flags
                    .lock()
                    .unwrap()
                    .iter()
                    .find(|f| Arc::ptr_eq(f, flag))
                    .and_then(|_| {
                        scheduler
                            .task_queue
                            .lock()
                            .unwrap()
                            .iter()
                            .find(|t| t.id == id)
                            .map(|t| t.priority)
                    })
            })
            .unwrap_or(0);
        let status = match &result {
            Some(_) => "完成",
            None => "取消",
        };
        let result_str = result.as_ref().map(String::as_str).unwrap_or("已取消");
        println!("{:<8} {:<10} {:<10} {}", id, priority, status, result_str);
    }
    // 7. 优雅关闭调度器
    scheduler.shutdown();
}
```
