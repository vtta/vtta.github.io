+++
title = "PNA - Project 1"
weight = 1
order = 1
date = 2020-01-25
#insert_anchor_links = "right"

[taxonomies]
tags = ["notes", "rust"]
+++

纠结了好久`Raft`到底是做PingCAP的还是MIT的，
于是昨天去试了一把GO，
过完官方教程的赶紧就是，这语言是一坨x么？？？
语法就不吐槽了，
编译器什么都不管，一有问题就是`runtime error`，
要是这样开干`Raft`，
那么程序出了问题不是debug到地老天荒？？？

还是我喜欢的`rust`香，
正好昨晚看到陈天大佬的文章说，
一个编程语言不能给自己带来新的范式那何必要学呢。
而且很喜欢`rust`先进的内存模型，
虽然`C++`里面新来的和`move`有关的东西实质上干的是一个事情，但是毕竟不是语言级的支持，
虽然程序里面会混入很多为了应付编译器的琐屑代码，
但是我还是喜欢，嗯，可能是因为是抖M吧。


## `StructOpt`

BB1 做完之后想起来之前似乎好像用过`StructOpt`，
正好看到Extension里提到，于是乎撸起袖子开干，
先把命令行参数解析的工作放在最开始消灭掉。

`StructOpt`和`Clap`不能同时放在`Cargo.toml`里面，
放在一起之后运行主程序啥也不输出就退出了，
猜想可能是`StructOpt`本身依赖`Clap`，
但是版本不一样，冲突了。
于是乎干掉`Clap`，只用`StructOpt`就好。

发现单独使用`StructOpt`这些`Argument Parser`的好处就是将输入的命令本身结构化了，
能在程序中直接根据命令的结构执行对应操作。
逻辑级概念变成了代码级，
比起之前`C++`里手动解析每一种可能的情况要方便直观许多。

嗯，可能我是懒，就是不想自己在`C++`里面抽象出来，
就是不想自己写Parser，
编译原理的精髓不应该是前端，
啥时候抽时间继续把`SICP`干完，flag先立在这里了，
我爱`rust` & `scheme`。

```rust
use structopt::StructOpt;

#[derive(Debug, StructOpt)]
#[structopt(name = "kvs", about = "A command-line key-value store client")]
struct Opt {
    /// Activate debug mode
    #[structopt(short, long)]
    debug: bool,
    /// Verbose mode (-v, -vv, -vvv, etc.)
    #[structopt(short, long, parse(from_occurrences))]
    verbose: u8,
    #[structopt(subcommand)]
    cmd: KvsCmd,
}

#[derive(Debug, StructOpt)]
enum KvsCmd {
    /// Set the value of a string key to a string
    Set {
        #[structopt(name = "KEY")]
        key: String,
        #[structopt(name = "VALUE")]
        value: String,
    },
    /// Get the string value of a given string key
    Get {
        #[structopt(name = "KEY")]
        key: String,
    },
    /// Remove a given key
    Rm {
        #[structopt(name = "KEY")]
        key: String,
    },
}
```

整个`struct`和嵌套的`enum`展开之后的数据结构就是所有参数整体的逻辑结构，
用宏标记了每个field的属性，
注释会被解析为对应field的帮助信息，妙啊。
