+++
title = "PostgreSQL JIT内联初探"
weight = 1
order = 1
date = 2020-02-06
#insert_anchor_links = "right"

[taxonomies]
tags = ["notes"]
+++

## 环境准备
```bash
brew install llvm
git clone --recurse-submodules https://git.postgresql.org/git/postgresql.git
cd postgresql
git reflog expire --expire=now --all
git gc --aggressive --prune=all
git checkout REL_12_STABLE
CC=/usr/local/opt/llvm/bin/clang \
    CXX=/usr/local/opt/llvm/bin/clang++ \
    CLANG=/usr/local/opt/llvm/bin/clang \
    LLVM_CONFIG=/usr/local/opt/llvm/bin/llvm-config \
    CFLAGS=-O0 \
    ./configure \
    --prefix=$HOME/local \
    --enable-cassert \
    --enable-depend \
    --enable-debug \
    --with-llvm
make
make install
mkdir -p ~/local/var/postgres
~/local/bin/initdb -D ~/local/var/postgres
```

## 用CLion调试
先写个CMakeLists.txt
```cmake
cmake_minimum_required(VERSION 3.6)
project(postgres)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_custom_target(postgres COMMAND make -C ${PROJECT_SOURCE_DIR} && make -C ${PROJECT_SOURCE_DIR} install )


file(GLOB_RECURSE sources "*.c" "*.cpp")
file(GLOB_RECURSE headers "*.h")

add_executable(dummy ${sources} ${headers})
target_include_directories(dummy PRIVATE src/include /usr/local/opt/llvm/include)
```

`dummy`完全是为了让CLion索引源码文件加进去的，没用。

然后CLion打开，先build一下。

接着Edit configurations：
- `Target`是`postgresql`
- `Executable`选`~/local/bin/postgres`
- `Program arguments`填`-D /Users/vtta/local/var/postgres`，注意这里最好是绝对路径

![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200206214758.png)

最后在`src/backend/main/main.c`里`main`函数打个断点：

![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200206214907.png)


## 索引代码
- 关闭SIP
- 用[这个神奇的东西](https://github.com/rizsotto/Bear)从`make`构建过程生成`compile_commands.json`
  ```bash
  brew install bear
  bear make
  ```
- 用`sourcetrail`索引代码
  - 添加`source group`时选择生成的`compile_commands.json`即可
  ```bash
  brew cask install sourcetrail
  ```
![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200207120732.png)


## 为什么需要内联
根据[官方文档上关于内联的介绍](https://www.postgresql.org/docs/11/jit-reason.html#JIT-INLINING)

> PostgreSQL is very extensible and allows new data types, functions, operators and other database objects to be defined; see Chapter 38. In fact the built-in objects are implemented using nearly the same mechanisms. This extensibility implies some overhead, for example due to function calls (see Section 38.3). To reduce that overhead, JIT compilation can inline the bodies of small functions into the expressions using them. That allows a significant percentage of the overhead to be optimized away.

PG是有很强可扩展性的，
但是正是由于扩展支持的函数调用等情况会增加开销。
JIT中做内联可以将这些函数体展开并集成到调用这些函数的表达式中，
消除了生成的机器码中的大量跳转指令。


## 运行实例
- 创建测试数据表
  ```sql
  show server_version;
  show port;
  select pg_backend_pid();
  
  create table t1 (id serial);
  insert INTO t1 (id) select * from generate_series(1, 10000000);
  ```

- 开启调试信息输出、开启JIT
  ```sql
  \timing
  select pg_backend_pid();
  set log_min_error_statement = DEBUG1; set log_min_messages = DEBUG1; set client_min_messages = DEBUG1;
  set jit = 'on'; set jit_above_cost = 10; set jit_inline_above_cost = 10; set jit_optimize_above_cost = 10;
  ```

- 运行查询语句
  ```sql
  explain (analyze, verbose, buffers) select count(*) from t1;
  ```
  ![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200207152320.png)
- [程序的调用栈](https://gist.github.com/vtta/430cdbf89d4cb9168f9434bf9e2e300a)
  <!--<link href="https://cdn.rawgit.com/lonekorean/gist-syntax-themes/848d6580/stylesheets/obsidian.css" rel="stylesheet" type="text css"><script src="https://gist.github.com/vtta/430cdbf89d4cb9168f9434bf9e2e300a.js" ></script>-->

- 查询计划输出
 ![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200207161151.png)

## 一个BUG
- 在JIT相关代码里面打了断点以后用`LLDB attach`，执行查询语句会触发`page fault`
- ![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200207165122.png)
- ![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200207170427.png)
- 因为postgres所有的库都是运行时动态加载，不是动态链接。所以可能要访问的内存地址处于库加载后映射到的内存区域，而库加载出现了问题。
- 验证猜想
- ![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200207165755.png)
- 可见除了系统本身的东西外并没有链接任何的库
- 将`llvmjit.so`改为启动时预先加载
- ![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200207170056.png)
- 重启LLDB尝试查询
- 果然
- ![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200207171547.png)



## 源码分析
### JIT provider的初始化
`provider_init`[加载外部函数](https://github.com/postgres/postgres/blob/2f4733993a967ce7f89d1a02c9f12988e9b7ff6f/src/backend/jit/jit.c#L115)`_PG_jit_provider_init`[初始化](https://github.com/postgres/postgres/blob/2f4733993a967ce7f89d1a02c9f12988e9b7ff6f/src/backend/jit/llvm/llvmjit.c#L125)
静态变量`provider`中的函数指针。
从这里开始每次调用`jit_compile_expr`[实际上就是使用LLVM-JIT](https://github.com/postgres/postgres/blob/2f4733993a967ce7f89d1a02c9f12988e9b7ff6f/src/backend/jit/jit.c#L180)进行编译了。

看到llvm被正确的调用了：

<img src="https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200207175752.png" style="zoom: 50%;" />

### 内联
![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGollvm_inline.png)

### 内联信息的准备
[`llvm_build_inline_plan`](https://github.com/postgres/postgres/blob/2f4733993a967ce7f89d1a02c9f12988e9b7ff6f/src/backend/jit/llvm/llvmjit_inline.cpp#L175)

![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGollvm_build_inline_plan.png)

- 构建内联所需的必要信息
- 首先检查存放字节码的目录是否存
- 将指向外部函数的引用加入工作列表
- 检查列表中每一个函数是否应该被内联
- 内联时检查函数的依赖是否过多
- 将内联过的函数做上标记
- 最后将整个模块做上标记
  
### 执行内联
[`llvm_execute_inline_plan`](https://github.com/postgres/postgres/blob/2f4733993a967ce7f89d1a02c9f12988e9b7ff6f/src/backend/jit/llvm/llvmjit_inline.cpp#L371)

![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGollvm_execute_inline_plan.png)

- 实际执行内联操作的函数
- 只内联静态全局变量
- 内联过程会删除调试信息

看到进入内联的执行
![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200207211753.png)
  
### 判断是否需要内联

- 根据当前查询计划花费[判断是否做内联](https://github.com/postgres/postgres/blob/2f4733993a967ce7f89d1a02c9f12988e9b7ff6f/src/backend/optimizer/plan/planner.c#L548)

### 判断函数是否需要做内联
[`function_inlinable`](https://github.com/postgres/postgres/blob/2f4733993a967ce7f89d1a02c9f12988e9b7ff6f/src/backend/jit/llvm/llvmjit_inline.cpp#L569)

![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGofunction_inlinable.png)

- 已存在的非静态函数不能内联
- 不能内联外界可见访问的函数
- 如果涉及变量只存在于某个文件的局部则可以内联
- 内联时要检查函数的外部引用
- 不直接内联外部引用，而是将函数加入对应的工作列表，再做决定
- 花费太小则不必做内联

