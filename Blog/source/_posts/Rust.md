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

```
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