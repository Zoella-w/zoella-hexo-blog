---
title: IB-Rust-并发1
date: 2025-06-16 16:28:03
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson25](https://github.com/Zoella-w/IB-Rust/tree/main/25_concurrent1)

## 并发与并行的区别

- 并发（Concurrency）：多个任务在时间上交替执行
- 并行（Parallelism）：多个任务同时执行

## 进程与线程

- 在大部分 OS 里，代码运行在进程（process）中，OS同时管理多个进程
- 在程序里，各独立部分可以同时运行，运行这些独立部分的就是线程（thread）
  - 多线程运行 可以提升性能表现 和 增加复杂性，但无法保障各线程的执行顺序

### 多线程导致的问题

- 竞争状态，线程以不一致的顺序访问数据或资源
- 死锁，两个线程彼此等待对方使用完所持有的资源，线程无法继续
- 只在某些情况下发生的 Bug，很难可靠地复制现象和修复

## 实现线程的方式

- 通过调用 OS 的 API 来创建线程：`1:1模型`
  - 需要较小的运行时
- 语言自己实现的线程（绿色线程）：`M:N模型`（Go语言）
  - 需要更大的运行时
- Rust：需要权衡运行时的支持
- Rust 标准库仅提供 `1:1模型` 的线程

### Rust 如何创建线程

通过 `thread::spawn` 函数可以创建新线程，其参数是一个闭包（在新线程里运行的代码）

```rust
use std::thread;
use std::time::Duration;
fn thread_study() {
    thread::spawm(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });
    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

### 通过 `join Handle` 来等待所有线程完成

`thread::spawn` 函数的返回值类型是 `JoinHandle`

`JoinHandle` 持有值的所有权，调用其 `join` 方法，会阻止当前运行线程的执行，直到 handle 所表示的线程终结

```rust
let handle = thread::spawn(|| {
    for i in 1..10 {
        println!("hi number {} from the spawned thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
});
for i in 1..5 {
    println!("hi number {} from the main thread!", i);
    thread::sleep(Duration::from_millis(1));
}
handle.join().unwrap();
```

### 使用 `move` 闭包

- move 闭包通常和 `thread::spawn` 函数一起使用，它允许使用用其它线程的数据
- 创建线程时，把值的所有权从一个线程转移到另一个线程

```rust
let v = vec![1, 2, 3];
// error: closure may outlive the current function, but it borrows `v`, which is owned by the current function
let handle = thread::spawn(move || {
    println!("Here is a vector: {:?}", v);
});
// drop(v); // error: use of moved value: `v`
handle.join().unwrap();
```

## 多线程通信
### 消息传递

一种流行且能保证安全并发的技术是：消息传递，线程（或 `Actor`）通过彼此发送消息（数据）来进行通信

- Go 语言：不要用共享内存来通信，要用通信来共与享内存（与 Rust 相似）
- Rust：Channel（标准库提供）

### Channel

- `Channel` 包含：发送端、接收端
- 调用发送端的方法，发送数据
- 接收端会检查和接收到达的数据
- 如果发送端、接收端中任意一端被丢弃了，那么 `Channel` 就又“关闭”了

#### 创建 channel

- 使用 `mpsc::channel` 函数来创建 Channel
  - `mpsc` 表示`multiple producer,single consumer`（多个生产者、一个消者）
  - 返回一个 `tuple`（元组）：里面元素分别是发送端、接收端
- 使用 `mpsc::sync_channel` 来创建带缓冲区的 channel
  - 入参为缓冲区大小，当缓冲区塞满时进行阻塞

```rust
use std::sync::mpsc;
fn thread_study() {
    let (tx, rx) = mpsc::channel();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_millis(200));
        }
    });
    for received in rx {
        println!("Got: {received}");
    }
}
```

## 总结

- 发送端的 `send` 方法
  - 参数：想要发送的数据
  - 返回：`Result<T,E>`
    - 如果有问题(例如接收端已经被丢弃)，就返回一个错误语
- 接收端的方法
  - `recv` 方法：阻止当前线程执行，直到 Channel 中有值被送来
    - 一旦有值收到，就返回 `Result<T,E>`
    - 当发送端关闭，就会收到一个错误
  - `try_recv` 方法：不会阻塞,
    - 立即返回 `Result<T,E>`:
      - 有数据达到：返回 Ok，里面包含着数据
      - 否则，返回错误
    - 通常会使用循环调用来检查 `try_recv` 的结果

## 课后习题1：实现多线程文件处理器

你需要编写一个多线程文件处理器，它从一个通道（channel）中接收文件路径，并在线程池中处理这些文件。文件处理的具体任务可以是读取文件内容并打印到控制台。你需要使用 Rust 的带缓冲区的 channel 来控制并发线程的数量，从而限制同时处理的文件数量。

- 具体要求
  - 文件处理任务：
  - 定义一个函数 process_file，该函数接受一个文件路径，读取文件内容，并将内容打印到控制台。
- 多线程控制：
  - 创建一个带缓冲区的 channel，用于在主线程和工作线程之间传递文件路径。
  - 使用多线程来实现文件处理的并发性，限制线程的并发数量（例如，最多同时处理 4 个文件）。
- 主线程作为生产者：
  - 主线程负责向通道发送文件路径。假设我们有 10 个文件路径要处理。
- 工作线程作为消费者：
  - 创建多个工作线程，每个线程从通道中接收文件路径，并调用 process_file 函数来处理文件。

```rust
use std::path::PathBuf;
use std::sync::mpsc;
use std::{fs, thread};

// 读取文件内容并打印
fn process_file(path: PathBuf) {
    // PathBuf 类型，代表文件路径
    match fs::read_to_string(&path) {
        Ok(content) => println!(
            "文件：{:?}\n内容：\n{}\n{}",
            path,
            content,
            "-".repeat(20) // 分割线
        ),
        Err(e) => eprintln!("无法读取文件 {:?}：{}", path, e),
    }
}
/*
    主线程 (生产者)
        │
        ▼
    [主通道] (同步通道，缓冲区=4) → 控制最大待处理任务数
        │
        ▼
    任务分发线程 (协调者)
        │
        ▼ (轮询分发)
    [子通道1] → 工作线程1 (消费者)
    [子通道2] → 工作线程2 (消费者)
    [子通道3] → 工作线程3 (消费者)
    [子通道4] → 工作线程4 (消费者)
*/
// 关闭顺序：
// 1. 主线程完成发送 → drop(tx) → 主通道关闭
// 2. 分发线程收完所有任务 → 循环结束 → 销毁所有 child_tx
// 3. 每个工作线程的 child_rx 接收端关闭 → 退出循环
// 4. 工作线程自然结束 → 分发线程 join → 主线程结束
fn main() {
    // 模拟包含10个文件路径的向量（file1.txt 到 file10.txt）
    let files = (1..=10)
        .map(|i| PathBuf::from(format!("file{}.txt", i)))
        .collect::<Vec<PathBuf>>();
    // 创建带缓冲区的通道 - 缓冲区大小 = 最大并发数
    // 当缓冲区满（4个任务等待）时，发送操作 (tx.send()) 会阻塞
    // 只有当工作线程从接收端取走任务后，发送操作才能继续
    // 实现任务的流量控制，防止生产者过快生产任务
    let (tx, rx) = mpsc::sync_channel(4); // 缓冲区大小 = 最大并发数
    // 创建工作线程池
    let mut workers = vec![];
    // 单个工作线程负责从通道接收任务
    let worker = thread::spawn({
        // 所有权转移，确保线程安全
        move || {
            println!("任务分发线程已启动");
            // 使用向量存储工作线程
            let mut child_workers = vec![];
            // 创建4个工作线程
            for id in 0..4 {
                // mpsc::channel() 为每个工作线程创建独立的异步通道
                // child_tx：任务分发线程使用的发送端
                // child_rx：工作线程使用的接收端
                let (child_tx, child_rx) = mpsc::channel();
                let child_worker = thread::spawn(move || {
                    println!("工作线程 {} 已启动", id);
                    // 迭代接收文件路径
                    for path in child_rx.iter() {
                        println!("工作线程 {} 正在处理：{:?}", id, path);
                        process_file(path);
                    }
                    println!("工作线程 {} 已结束", id);
                });
                // 存储所有工作线程的发送端和句柄
                child_workers.push((child_tx, child_worker));
            }
            // 轮询方式分发任务
            let mut index = 0;
            for path in rx.iter() {
                // 选择下一个工作线程
                let (child_tx, _) = &child_workers[index];
                // 发送任务到工作线程
                child_tx.send(path).expect("向工作线程发送任务失败");
                // 移动到下一个工作线程
                index = (index + 1) % child_workers.len();
            }
            // 关闭所有子通道
            for (child_tx, _) in &child_workers {
                drop(child_tx.clone());
            }
            // 等待所有工作线程完成
            for (_, child_worker) in child_workers {
                child_worker.join().expect("工作线程异常退出");
            }
            println!("任务分发线程已结束");
        }
    });
    workers.push(worker);
    // 主线程作为生产者发送文件路径
    for file in files {
        println!("主线程发送: {:?}", file);
        tx.send(file).expect("发送文件路径失败");
    }
    // 关闭发送通道
    drop(tx);
    // 等待任务分发线程完成
    for worker in workers {
        worker.join().expect("任务分发线程异常退出");
    }
    println!("所有任务已完成");
}
```

## 课后习题2：使用 Channel 实现程序的优雅停止

在这道练习中，你需要编写一个多线程程序，该程序会创建多个工作线程，持续处理任务。在接收到停止信号时，所有工作线程应该优雅地停止工作，并确保所有未完成的任务都被处理完毕。

- 具体要求
  - 你将使用 Rust 的 channel 来实现任务的调度和优雅停止机制。
  - 工作线程：
    - 创建一个工作线程池，工作线程从通道接收任务并处理。
    - 工作线程应能够响应停止信号，并在完成当前任务后优雅地退出。
- 任务结构：
  - 任务可以是简单的打印操作，模拟一些耗时工作，例如打印任务 ID 并暂停一段时间。
- 优雅停止：
  - 通过发送一个特殊的停止信号，通知所有工作线程停止接收新的任务，并在完成当前任务后退出。
  - 确保所有已接收的任务都被处理完毕。
- 主线程控制：
  - 主线程应当能够发送任务，也能够在适当的时候发送停止信号，触发工作线程的优雅停止。

```rust
use std::fmt;
use std::sync::mpsc::{self, Receiver, Sender};
use std::thread;
use std::time::Duration;

// 任务类型定义
#[derive(Debug, Clone)]
enum Task {
    // 常规任务，包含任务ID
    Job(i32),
    // 停止信号，要求所有工作线程优雅退出
    Terminate,
}

// 为 Task 实现自定义显示
impl fmt::Display for Task {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            Task::Job(id) => write!(f, "任务 #{}", id),
            Task::Terminate => write!(f, "停止信号"),
        }
    }
}

fn main() {
    // 创建4个工作线程及其通道
    let mut worker_txs = Vec::new(); // 存储工作线程的发送端
    let mut workers = Vec::new(); // 存储工作线程的句柄
    // 创建停止通知通道
    let (stop_tx, stop_rx) = mpsc::channel();
    // 创建并启动4个工作线程
    for id in 0..4 {
        // 为每个工作线程创建专用通道
        let (worker_tx, worker_rx) = mpsc::channel();
        // worker_tx 存储在 worker_txs 中，供主线程发送任务
        worker_txs.push(worker_tx);
        // 为每个工作线程创建专用的停止通知发送端
        let thread_stop_tx = stop_tx.clone();
        let worker = thread::spawn(move || {
            // worker_rx 传递给工作线程，用于接收任务
            worker_thread(id, worker_rx, thread_stop_tx);
        });
        workers.push(worker);
    }
    // 创建可变的任务分发函数（闭包）
    let mut current_worker = 0;
    let mut send_task = |task: Task| {
        // 添加 mut 关键字使闭包可变
        worker_txs[current_worker].send(task).unwrap();
        current_worker = (current_worker + 1) % worker_txs.len();
    };
    // 发送10个任务
    for task_id in 1..=10 {
        let task = Task::Job(task_id);
        println!("[主线程] 发送任务: {}", task);
        send_task(task); // 调用可变闭包进行分发
    }
    // 发送4个 Task::Terminate 停止信号
    // 每个工作线程都会收到一个停止信号
    println!("[主线程] 发送停止信号，等待工作线程完成当前任务...");
    for _ in 0..4 {
        send_task(Task::Terminate); // 调用可变闭包
    }
    // 等待工作线程发送停止确认
    let mut stopped_workers = 0;
    while stopped_workers < 4 {
        // 主线程阻塞在 stop_rx.recv() 上
        match stop_rx.recv() {
            Ok(worker_id) => {
                println!("[主线程] 收到工作线程{}的停止确认", worker_id);
                stopped_workers += 1;
            }
            Err(_) => {
                // 如果通道意外关闭，跳出循环
                println!("[主线程] 停止通道已关闭");
                break;
            }
        }
    }
    // 关闭所有工作线程通道（通知工作线程退出）
    for tx in worker_txs {
        drop(tx); // 显式关闭通道
    }
    // 等待所有工作线程结束
    for worker in workers {
        worker.join().unwrap();
    }

    println!("[主线程] 所有工作线程已退出，程序结束");
}

/// 工作线程函数
fn worker_thread(
    id: u8,
    rx: Receiver<Task>,
    stop_tx: Sender<u8>, // 用于通知主线程本线程已停止
) {
    println!("[工作线程{}] 已启动", id);
    // 处理任务的循环
    // 给循环命名 task_loop
    'task_loop: for task in rx.iter() {
        match task {
            Task::Job(task_id) => {
                println!("[工作线程{}] 开始处理: {}", id, Task::Job(task_id));
                // 模拟任务处理
                let duration = Duration::from_millis(200 + (task_id as u64 % 4) * 100);
                thread::sleep(duration);
                println!("[工作线程{}] 完成处理: {}", id, Task::Job(task_id));
            }
            Task::Terminate => {
                println!("[工作线程{}] 接收到停止信号，准备退出", id);

                // 通知主线程本线程已收到停止信号
                stop_tx.send(id).unwrap();
                break 'task_loop;
            }
        }
    }
    // 处理通道中剩余的任务（在停止信号之后发送的任务）
    println!("[工作线程{}] 处理剩余任务...", id);
    for task in rx.iter() {
        match task {
            Task::Job(task_id) => {
                println!("[工作线程{}] 处理剩余任务: #{}", id, task_id);
                thread::sleep(Duration::from_millis(50)); // 快速处理, 使用较短的休眠时间 (50ms)
            }
            Task::Terminate => {
                // 忽略额外的停止信号，避免重复处理停止信号
            }
        }
    }
    println!("[工作线程{}] 已完成所有任务，退出", id);
}
```