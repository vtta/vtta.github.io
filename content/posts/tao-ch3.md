+++
title = "《Rust编程之道》第三章笔记"
weight = 1
order = 1
date = 2020-01-14
#insert_anchor_links = "right"

[taxonomies]
tags = ["notes", "rust"]
+++



- 类型
  - 胖指针
    ```rust
    use std::mem::size_of;
    assert_eq!(size_of::<&[u32; 5]>(), 8);
    assert_eq!(size_of::<&[u32]>(), 16);
    ```
  - 