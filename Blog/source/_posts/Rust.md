---
title: Rust
index_img: /img/rust.png
date: 2020-12-09 13:53:19
tags: rust
---

## **数据类型**

### 元组

元组用一对 **( )** 包括的一组数据，可以包含**不同种类**的数据：

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
// tup.0 等于 500
// tup.1 等于 6.4
// tup.2 等于 1
let (x, y, z) = tup;
// y 等于 6.4
```

### 数组

数组用一对 **[ ]** 包括的**同类型**数据:

```rust
let c: [i32; 5] = [1, 2, 3, 4, 5];
// c 是一个长度为 5 的 i32 数组
```

## 关键词

### let

将值绑定到变量。

```rust
let a：&str = "rt-thread"; /* 字符串 */
let b：i32 = 123; /* 整型 */
let c：f64 = 2.0; /* 浮点型 */
```

### mut

可变的变量、引用或指针。

```rust
/* 不使用 mut */
fn main() {
    let b = 123;
    b += 1;
    println!("b = {:?}", b);
}

> error[E0384]: cannot assign twice to immutable variable `b`
> --> src\main.rs:9:5
> |
> |     let b = 123;
> |         -
> |         |
> |         first assignment to `b`
> |         help: make this binding mutable: `mut b`
> |     b += 1;
> |     ^^^^^^ cannot assign twice to immutable variable

/* 使用mut */
fn main() {
    let mut b = 123;
    b += 1;
    println!("b = {:?}", b);
}

    Finished dev [unoptimized + debuginfo] target(s) in 1.58s
     Running `target\debug\greeting.exe`
124
```

### const

常量和确定性函数。

```rust
const THING: u32 = 0xABAD1DEA;
```

不可变变量与常量的区别：

- 不可变变量可已通过 let 关键词修改变量的值，类型不可修改：

  ```rust
  let a:i32 = 123;
  let a:i32 = 456;
  ```

- 常量无法修改，值无法修改，类型无法修改：

  ```rust
  const THING: u32 = 0xABAD1DEA;
  THING = 0x01; 
  
  error[E0070]: invalid left-hand side of assignment
   --> src\main.rs:4:11
    |
  4 |     THING = 0x01;
    |     ----- ^
    |     |
    |     cannot assign to this expression
  
  error: aborting due to previous error
  
  ```

## 循环语句

### if

```rust
fn main() {
    let number = 3;
    if number < 5 {
        println!("条件为 true");
    } else {
        println!("条件为 false");
    }
}
```

- 条件表达式 number < 5 不需要用小括号包括（注意，不需要不是不允许）
-  Rust 中的 if 不存在单语句不用加 {} 的规则，不允许使用一个语句代替一个块

### else-if

```rust
if <condition> { block 1 } else { block 2 } 
```

```rust
fn main() {
    let a = 12;
    let b;
    if a > 0 {
        b = 1;
    }  
    else if a < 0 {
        b = -1;
    }  
    else {
        b = 0;
    }
    println!("b is {}", b);
}
```

Rust 中的条件表达式必须是 bool 类型。

表达式：

```rust
fn main() {
    let number = if 0 > 1 {1} else {2};
    println!("number = {}", number);
}
```

### While

```rust
fn main() {
    let mut number = 2;
    while number < 4 {
        println!("number = {}", number);
        number += 1;
    }
}
```

### For 

```rust
fn main() {
    let a = [10, 20, 30, 40];
    let mut j = 0;
    for i in a.iter() {
        println!("a[{:?}] = {:?}", j, i);
        j += 1;
    }
}
```

```rust
fn main() {
let a = [10, 20, 30, 40, 50];
    for i in 0..5 {
        println!("a[{}] = {}", i, a[i]);
    }
}
```

### Loop

```rust
fn main() {
    let s = ['R', 'U', 'N', 'O', 'O', 'B'];
    let mut i = 0;
    loop {
        let ch = s[i];
        if ch == 'O' {
            break;
        }
        println!("\'{}\'", ch);
        i += 1;
    }
}
```

## 所有权

所有权有以下三条规则：

- Rust 中的每个值都有一个变量，称为其所有者。
- 一次只能有一个所有者。
- 当所有者不在程序运行范围时，该值将被删除。

### 变量范围

```rust
{
    // 在声明以前，变量 s 无效
    let s = "runoob";
    // 这里是变量 s 的可用范围
}
// 变量范围已经结束，变量 s 无效
```

### 内存和分配

### 变量与数据交互的方式

变量与数据交互方式主要有移动（Move）和克隆（Clone）两种：

#### 移动

多个变量可以在 Rust 中以不同的方式与相同的数据交互：

```rust
fn main() {
    let x = 5;
    let y = x;
    println!("x= {}, y= {}", x, y);
}
```

这个程序将值 5 绑定到变量 x，然后将 x 的值复制并赋值给变量 y。现在栈中将有两个值 5。此情况中的数据是"基本数据"类型的数据，不需要存储到堆中，仅在栈中的数据的"移动"方式是直接复制，这不会花费更长的时间或更多的存储空间。"基本数据"类型有这些：

- 所有整数类型，例如 i32 、 u32 、 i64 等。
- 布尔类型 bool，值为 true 或 false 。
- 所有浮点类型，f32 和 f64。
- 字符类型 char。
- 仅包含以上类型数据的元组（Tuples）。

但如果发生交互的数据在堆中就是另外一种情况

```rust
fn main() {
    let x = String::from("hello");
    let y = x;
    println!("x = {}, y= {}", x, y);
}

 --> src\main.rs:4:31
  |
2 |     let x = String::from("hello");
  |         - move occurs because `x` has type `String`, which does not implement the `Copy` trait
3 |     let y = x;
  |             - value moved here
4 |     println!("x = {}, y= {}", x, y);
  |                               ^ value borrowed here after move

error: aborting due to previous error

```

