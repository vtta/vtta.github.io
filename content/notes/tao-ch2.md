+++
title = "《Rust编程之道》第二章笔记"
weight = 1
order = 1
date = 2020-01-13
#insert_anchor_links = "right"

[taxonomies]
tags = ["rust", "notes"]
+++

<!-- more -->


- 位置表达式和值表达式
  - 也就是C++的左值和右值
  - 位置表达式 / Place Expression / lvalue
    - 包括
      - 本地变量
      - 静态变量
      - 解引用`*expr`
      - 数组索引`expr[expr]`
      - 字段引用`expr.field`
      - 位置表达式组合
    - 代表了持久性数据
    - 位置表达式上下文
      - 赋值或者复合赋值语句左侧的操作数
      - 一元引用表达式的独立操作数
      - 包含隐式借用（引用）的操作数
      - `match`判别式或`let`绑定右侧在使用`ref`模式匹配的时候也是位置上下文
  - 值表达式 / Value Expression / rvalue
    - 值表达式一般只引用了某个存储单元地址中的数据
    - 相当于只读数据值
    - 代表了临时数据
- 所有权
  - 所有权的转移
    - 当位置表达式出现在值上下文中时，该位置表达式将会把内存地址转移给另外一个位置表达式
- 类型
  - `as`转换
    - 高位会被截断
      - e.g. `＇ಠ＇`转换为`i8`最终得到`-96`
  - `Range`
    - `std::ops::Range` => [begin, end)
      - `(begin..end)`
    - `std::ops::RangeInclusive` => [begin, end]
      - `(begin..=end)`
- `trait`
  - `trait`和`trait object`
  - ```rust
    struct Duck;
    struct Pig;
    trait Fly {
        fn fly(&self) -> bool;
    }
    impl Fly for Duck {
        fn fly(&self) -> bool {
            true
        }
    }
    impl Fly for Pig {
        fn fly(&self) -> bool {
            false
        }
    }
    // compile time
    fn fly_static<T: Fly>(x: &T) -> bool {
        x.fly()
    }
    // run time
    fn fly_dyn(x: &dyn Fly) -> bool {
        x.fly()
    }
    ```
- 输出
  - `{}` Display
  - `{:?}` Debug
  - `{:p}` 指针
  - `{:b}` 二进制
  - `{:o}` 八进制
  - `{:x}` `{:X}` 十六进制
  - `{:e}` `{:E}` 指数
