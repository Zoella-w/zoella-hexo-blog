---
title: IB-Rust-智能指针2
date: 2025-06-09 13:08:47
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson19](https://github.com/Zoella-w/IB-Rust/tree/main/19_smart_pointer2)


## 课程概述

- `Rc<T>`
- `RefCell<T>`
- `Weak<T>`

## `Rc<T>` 引用计数指针

### 什么是 `Rc<T>`？

- Rc 是 Reference Counted（引用计数） 的缩写
- 允许多所有者的共享所有权模型

### 举例说明

使用 `Box<T>` 定义 cons list 的例子。这一次，我们希望创建两个共享第三个列表所有权的列表

尝试使用 `Box<T>` 定义的 List：

```rust
use crate::List::{Cons, Nil};
fn main() {
    let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a)); // use of moved value: `a`
}
enum List {
    Cons(i32, Box<List>),
    Nil,
}
```

### 使用 `Rc<T>` 实现

```rust
use crate::List::{Cons, Nil};
use std::rc::Rc;
fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("a: {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("a: {}", Rc::strong_count(&a));
    let c = Cons(4, Rc::clone(&a));
    println!("a: {}", Rc::strong_count(&a));
}
enum List {
    Cons(i32, Rc<List>),
    Nil,
}
```

## `RefCell<T>`

### 什么是`RefCell<T>`？

`RefCell<T>` 是一个智能指针类型，允许在编译时无法确定的情况下，在运行时进行借用检查

### 主要特征

- 内部可变性:
  - `RefCell<T>` 允许你在其拥有的 `T` 内部进行修改，即使 `RefCell` 本身是不可变的。这是通过在运行时进行借用检查实现的
- 运行时借用检查:
  - `RefCell` 使用动态借用检查，确保在运行时遵循 Rust 的借用规则，即在任何时刻，`RefCell` 只能有一个可变借用或多个不可变借用，但不能同时存在
- `borrow` 和 `borrow_mut` 方法:
  - `RefCell` 提供了两个方法来获取对内部数据的借用：
    - `borrow()`：获取不可变借用（`Ref<T>`），可以同时有多个
    - `borrow_mut()`：获取可变借用（`RefMut<T>`），在同一时间只能有一个

### 关键点

- 借用规则:
  - `RefCell` 在运行时检查借用规则，以防止数据竞争和未定义行为。编译器不进行这些检查，而是依赖 `RefCell` 在运行时进行
- 运行时开销:
  - 因为 `RefCell` 需要在运行时检查借用规则，所以它会引入一定的性能开销。这在需要在编译时确定所有借用规则的场景中不可替代
- 错误处理:
  - 如果违反了借用规则（例如，尝试同时获取多个可变借用），`RefCell` 会在运行时引发 panic

### 使用场景

- 数据结构:
  - 在需要可变性但又受限于 Rust 的所有权系统时，`RefCell` 允许在数据结构中使用内部可变性。例如，实现需要共享但修改的数据结构（如图、树）
- 单线程环境:
  - `RefCell` 主要用于单线程环境。如果你需要在多线程环境中处理内部可变性，应该使用 Mutex 或 RwLock 这类类型。

## 引用循环与内存泄漏

引用计数（`Rc<T>`）和原子引用计数（`Arc<T>`）可以让多个所有者共享同一个数据。然而，这种共享机制如果不当使用，可能会导致引用循环（reference cycle），从而造成内存泄漏

### 例子

```rust
use std::rc::Rc;
use std::cell::RefCell;
#[derive(Debug)]
struct Node {
    value: i32,
    next: Option<Rc<RefCell<Node>>>,
}
fn main() {
    let first = Rc::new(RefCell::new(Node { value: 1, next: None }));
    let second = Rc::new(RefCell::new(Node { value: 2, next: None }));
    // 创建引用循环
    first.borrow_mut().next = Some(Rc::clone(&second));
    second.borrow_mut().next = Some(Rc::clone(&first));
    // 如果尝试打印引用计数，将看到引用循环已经发生
    println!("first strong = {}, weak = {}", Rc::strong_count(&first), Rc::weak_count(&first));
    println!("second strong = {}, weak = {}", Rc::strong_count(&second), Rc::weak_count(&second));
    // println!("{:?}", &first); // stack overflow
}
```

## `Weak<T>` 弱引用

### `Weak<T>` 的特点

- 非所有权引用: `Weak<T>` 并不拥有数据的所有权，因此它不会影响 `Rc<T>` 的引用计数
- 不会引发内存泄漏: 由于 `Weak<T>` 不增加引用计数，可以避免引用循环问题，从而避免内存泄漏
- 必须升级: `Weak<T>` 是一个非所有权引用，因此在使用数据之前，需要通过 `upgrade()` 方法将其升级为 `Rc<T>`。如果数据已经被释放，`upgrade()` 会返回 `None`

### 解决循环引用

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};
#[derive(Debug)]
struct Node {
    value: i32,
    next: Option<Rc<RefCell<Node>>>,
    prev: Option<Weak<RefCell<Node>>>, // 添加一个弱引用来指向前一个节点
}
fn main() {
    let first = Rc::new(RefCell::new(Node {
        value: 1,
        next: None,
        prev: None,
    }));
    let second = Rc::new(RefCell::new(Node {
        value: 2,
        next: None,
        prev: None,
    }));
    // 创建非循环引用
    first.borrow_mut().next = Some(Rc::clone(&second));
    second.borrow_mut().prev = Some(Rc::downgrade(&first));
    // 如果尝试打印引用计数
    println!(
        "first strong = {}, weak = {}",
        Rc::strong_count(&first),
        Rc::weak_count(&first)
    ); // first strong = 1, weak = 1
    println!(
        "second strong = {}, weak = {}",
        Rc::strong_count(&second),
        Rc::weak_count(&second)
    ); // second strong = 2, weak = 0
    // 因为 second 还拥有 first.borrow_mut().next 的所有权
    println!("{:?}", &first);
}
```

### 强引用与弱引用的主要区别

1. **所有权**:
   - 强引用 (`Rc<T>`): 持有数据的所有权，保证数据在作用域内不会被释放。
   - 弱引用 (`Weak<T>`): 不持有数据的所有权，不影响数据的生命周期。

2. **引用计数**:
   - 强引用: 增加引用计数，数据被多个所有者共享。
   - 弱引用: 不增加引用计数，不干扰 `Rc<T>` 的生命周期管理。

3. **内存管理**:
   - 强引用: 只有当所有强引用都被丢弃时，数据才会被释放。
   - 弱引用: 只能通过升级 (`upgrade()`) 来访问数据，如果数据已经被释放，则升级会失败。

4. **适用场景**:
   - 强引用: 当你希望共享数据并确保数据在至少一个强引用存在时不会被释放。
   - 弱引用: 当你需要避免引用循环或只需要偶尔访问数据，不想持有其所有权时。

## 课后作业

任务: 实现一个简单的社交网络系统，包含用户和朋友关系。使用 `Rc<T>`, `RefCell<T>`, `Weak<T>` 来处理用户和朋友之间的关系，并避免循环引用导致的内存泄漏

要求:
- 用户结构: 每个用户拥有一个名字和一个朋友列表。
- 添加朋友: 支持在两个用户之间建立朋友关系。
- 展示朋友关系: 能够展示每个用户的朋友列表。
- 循环引用: 处理用户之间的双向引用，确保不产生循环引用。

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

struct User {
    name: String,
    // 一个用户拥有一个朋友列表（Vec），这个列表存储的是对其他用户的弱引用（Weak）
    // 每个朋友用户（即其他用户）需要具有内部可变性，所以每个朋友用户被包裹在 RefCell 中
    // 用户自己需要能够修改朋友列表（添加朋友），所以整个列表被包裹在 RefCell 中以提供内部可变性
    friends: RefCell<Vec<Weak<RefCell<User>>>>,
}

impl User {
    // 用户对象需要：共享所有权（Rc）、提供内部可变性（RefCell）
    fn new(name: &str) -> Rc<RefCell<Self>> {
        Rc::new(RefCell::new(User {
            name: name.to_string(),
            friends: RefCell::new(Vec::new()),
        }))
    }

    // 添加好友：在 user1 和 user2 之间建立好友关系
    fn add_friend(user1: &Rc<RefCell<Self>>, user2: &Rc<RefCell<Self>>) {
        // 在 user1 的朋友列表中添加 user2 的弱引用
        user1
            .borrow_mut() // 获取 user1 的可变借用
            .friends
            .borrow_mut() // 获取 user1 好友列表的可变借用
            .push(Rc::downgrade(user2)); // 添加 user2 的弱引用
        // 在 user2 的朋友列表中添加 user1 的弱引用
        user2
            .borrow_mut()
            .friends
            .borrow_mut()
            .push(Rc::downgrade((user1)));
    }

    // 显示用户的好友列表
    fn show_friends(&self) {
        println!("{} 的朋友:", self.name);
        // 遍历好友列表中的每个弱引用
        for (i, friend) in self.friends.borrow().iter().enumerate() {
            // 将弱引用升级为强引用
            if let Some(friend_rc) = friend.upgrade() {
                let friend_ref = friend_rc.borrow();
                // 打印朋友名称
                println!("  {}. {}", i + 1, friend_ref.name);
            }
            // 朋友已释放（异常情况）
            else {
                println!("  {}. [已移除的朋友]", i + 1);
            }
        }
        println!(); // 添加空行分隔
    }
}

fn main() {
    // 创建用户 Alice
    let alice = User::new("Alice");
    println!(
        "[创建] Alice, 初始强引用计数 = {}",
        Rc::strong_count(&alice)
    );
    // 创建用户 Bob
    let bob = User::new("Bob");
    println!("[创建] Bob, 初始强引用计数 = {}", Rc::strong_count(&bob));
    // 创建用户 Charlie
    let charlie = User::new("Charlie");
    println!(
        "[创建] Charlie, 初始强引用计数 = {}",
        Rc::strong_count(&charlie)
    );

    // 建立朋友关系
    // 添加 Alice 和 Bob 为朋友
    User::add_friend(&alice, &bob);
    println!("[添加好友] Alice 和 Bob");
    println!(
        "  Alice 强引用计数 = {}, 弱引用计数 = {}",
        Rc::strong_count(&alice),
        Rc::weak_count(&alice)
    );
    println!(
        "  Bob 强引用计数 = {}, 弱引用计数 = {}",
        Rc::strong_count(&bob),
        Rc::weak_count(&bob)
    );
    // 添加 Alice 和 Charlie 为朋友
    User::add_friend(&alice, &charlie);
    println!("[添加好友] Alice 和 Charlie");
    println!(
        "  Alice 强引用计数 = {}, 弱引用计数 = {}",
        Rc::strong_count(&alice),
        Rc::weak_count(&alice)
    );
    println!(
        "  Charlie 强引用计数 = {}, 弱引用计数 = {}",
        Rc::strong_count(&charlie),
        Rc::weak_count(&charlie)
    );
    // 添加 Bob 和 Charlie 为朋友
    User::add_friend(&bob, &charlie);
    println!("[添加好友] Bob 和 Charlie");
    println!(
        "  Bob 强引用计数 = {}, 弱引用计数 = {}",
        Rc::strong_count(&bob),
        Rc::weak_count(&bob)
    );
    println!(
        "  Charlie 强引用计数 = {}, 弱引用计数 = {}",
        Rc::strong_count(&charlie),
        Rc::weak_count(&charlie)
    );

    // 创建临时用户 Dave 来演示弱引用
    {
        let dave = User::new("Dave");
        println!(
            "[创建临时用户] Dave, 初始强引用计数 = {}",
            Rc::strong_count(&dave)
        );
        // 添加 Alice 和 Dave 为朋友
        // 注意：这里只单向添加，避免循环引用
        alice
            .borrow_mut()
            .friends
            .borrow_mut()
            .push(Rc::downgrade(&dave));
        println!("[添加好友] Alice 和 Dave（弱引用）");
        println!(
            "  Alice 强引用计数 = {}, 弱引用计数 = {}",
            Rc::strong_count(&alice),
            Rc::weak_count(&alice)
        );
        println!(
            "  Dave 强引用计数 = {}, 弱引用计数 = {}",
            Rc::strong_count(&dave),
            Rc::weak_count(&dave)
        );
        // 展示 Alice 的朋友（包括 Dave）
        println!("[展示 Alice 的完整朋友列表]");
        alice.borrow().show_friends();
        // 作用域结束，Dave 将被销毁
        println!("[临时用户 Dave 离开作用域]");
    }

    // 展示最终朋友关系
    // 展示 Alice 的朋友列表（Dave 已消失）
    println!("[Alice 的最终朋友列表]");
    alice.borrow().show_friends();
    // 展示 Bob 的朋友列表
    println!("[Bob 的朋友列表]");
    bob.borrow().show_friends();
    // 展示 Charlie 的朋友列表
    println!("[Charlie 的朋友列表]");
    charlie.borrow().show_friends();

    // 最终引用计数状态
    println!("程序结束前引用计数状态:");
    println!(
        "Alice: 强引用计数 = {}, 弱引用计数 = {}",
        Rc::strong_count(&alice),
        Rc::weak_count(&alice)
    );
    println!(
        "Bob: 强引用计数 = {}, 弱引用计数 = {}",
        Rc::strong_count(&bob),
        Rc::weak_count(&bob)
    );
    println!(
        "Charlie: 强引用计数 = {}, 弱引用计数 = {}",
        Rc::strong_count(&charlie),
        Rc::weak_count(&charlie)
    );
}
```
