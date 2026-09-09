# 一组 Rust 谜题

在水群中无意之中见到这组聊天记录，甚感新奇，是以记之。

这组谜题的特点是代码能编译，但解释全瞎编，因此 Rust 初学者慎入。

聊天记录的作者叫 ElenMiku，本文仅是对其聊天记录的摘抄。

本文的所有代码均在 `rustc 1.98.1 (48a229cea 2026-09-01) (Arch Linux rust 1:1.98.1-1)` 环境下通过编译，不保证在以后的版本中也能编译。

## 谜题 1

你知道吗？Rust 也有空值合并运算符

```rust
fn foo(xs: &mut Vec<Option<i32>>) -> Option<i32> {
    Some(xs.pop() ?? -1)
}
```

[解答](/website/2026/9/puzzle-1-answer.md)

## 谜题 2

你知道吗？Rust 支持命名参数，声明参数名后即可乱序传参

```rust
let connect = |timeout, retries| {};

let (timeout, retries);
connect(retries = 3, timeout = 30);

assert_eq!(timeout, 30);
assert_eq!(retries, 3);
```

## 谜题 3

你知道吗？Rust 可以直接匹配闭包的实现，形式化验证这一块

```rust
let identity = |x: i32| x;
assert!(matches!(identity, |x| x));
```

## 谜题 4

你知道吗？Rust 的空块参与算术时会按语境转换成 0 或 1

```rust
fn sub(ref x: i32) -> i32 { {} - x }
fn mul(ref x: i32) -> i32 { {} * x }

assert_eq!(sub(42), -42);
assert_eq!(mul(42), 42);
```

## 谜题 5

你知道吗？Rust 支持局部函数特化，离开作用域后自动恢复泛型版本

```rust
fn foo<T>(_: T) -> i32 { 0 }

assert_eq!(foo(42u8), 0);
{
    fn foo<u8>(_: u8) -> i32 { 1 }

    assert_eq!(foo(42u8), 1);
}
assert_eq!(foo(42u8), 0);
```

## 谜题 6

你知道吗？Rust 会为 ZST 自动实现 Copy

```rust
#[derive(Default)]
struct token;

let token = token::default();
drop(token);
drop(token);
```

## 谜题 7

你知道吗？Rust 也支持整数提升，u8 取负前会先提升成 i32

```rust
assert_eq!(concat!(-1u8), "-1");
assert_eq!(concat!(-255u8), "-255");
```

## 谜题 8

你知道吗？为了防止返回值不符合预期，可以对返回值进行匹配

```rust
fn foo() -> i32 {
    match return 1 {
        1 => todo!(),
        _ => panic!(),
    }
}
```

## 谜题 9

你知道吗？只要方法签名一致，就不必显式 impl trait

```rust
trait HasLen { fn len(&self) -> usize; }
type Getter<T: HasLen> = fn(&T) -> usize;

let len: Getter<Vec<i32>> = Vec::len;
assert_eq!(len(&vec![1, 2, 3]), 3);
```

## 谜题 10

你知道吗？Rust 支持运行时添加方法

```rust
struct X;
let enabled = true;

if enabled {
    impl X {
        fn foo(&self) -> i32 { 42 }
    }
}

assert_eq!(X.foo(), 42);
```

## 谜题 11

你知道吗？Rust 支持参数展开，声明时用括号标记参数组

```rust
let add = |(a, b)| a + b;
let cases = [(20, 22), (19, 23)];

for args in cases.iter() {
    assert_eq!(add(*args), 42);
}
```
