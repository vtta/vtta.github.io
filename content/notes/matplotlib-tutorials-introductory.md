+++
title = "matplotlib 介绍"
weight = 1
order = 1
date = 2020-02-09
#insert_anchor_links = "right"

[taxonomies]
tags = ["notes"]

+++

## `matplotlib`基本概念

### 0x00 - 顶层

- 最顶层抽象：`state-machine environment`
- 所在的模块：[`matplotlib.pyplot`](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.html#module-matplotlib.pyplot)
- 在这层对当前图像/轴增加元素：
  - 线条
  - 图片
  - 文本

### 0x01 - 面向对象接口

- 基本函数

  - 图像创建

  - 添加轴

  - ```python
    import matplotlib.pyplot as plt
    import numpy as np
    ```

- 图像的组成部分

  - <img src="https://matplotlib.org/_images/anatomy.png" />

  - ### [`Figure`](https://matplotlib.org/api/_as_gen/matplotlib.figure.Figure.html#matplotlib.figure.Figure)

    - The figure keeps track of all the child [`Axes`](https://matplotlib.org/api/axes_api.html#matplotlib.axes.Axes), a smattering of 'special' artists (titles, figure legends, etc), and the **canvas**.

    - A figure can have any number of [`Axes`](https://matplotlib.org/api/axes_api.html#matplotlib.axes.Axes), but to be useful should have at least one.

    - ```python
      # an empty figure with no axes
      fig = plt.figure()
      # Add a title so we know which it is
      fig.suptitle('No axes on this figure')
      
      # a figure with a 2x2 grid of Axes
      fig, ax_lst = plt.subplots(2, 2) 
      ```

  - ### [`Axes`](https://matplotlib.org/api/axes_api.html#matplotlib.axes.Axes)

    - 是看上去的图像
    - 是带有数据的图像区域
    - 一个图像`Figure`可以有多个`Axes`
    - 一个`Axes`有维度个数个`Axis`
      - `Axis`控制数据的范围
      - 可以在`Axes`中设置
      - [`set_xlim()`](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.set_xlim.html#matplotlib.axes.Axes.set_xlim)
      - [`set_ylim()`](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.set_ylim.html#matplotlib.axes.Axes.set_ylim)
    - 有标题[`set_title()`](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.set_title.html#matplotlib.axes.Axes.set_title)
    - 有标签[`set_xlabel()`](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.set_xlabel.html#matplotlib.axes.Axes.set_xlabel) [`set_ylabel()`](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.set_ylabel.html#matplotlib.axes.Axes.set_ylabel)
  - `Axes`是和面向对象接口打交道的入口
  
- ### [`Axis`](https://matplotlib.org/api/axis_api.html#matplotlib.axis.Axis)

  - 数据线对象？
    - 设置数据范围和`ticks`
    - `ticks`：轴上标记
      - 位置由[`Locator`](https://matplotlib.org/api/ticker_api.html#matplotlib.ticker.Locator)确定
      - 格式由[`Formatter`](https://matplotlib.org/api/ticker_api.html#matplotlib.ticker.Formatter)确定
    - `ticklabels`：标记的标签

- ### [`Artist`](https://matplotlib.org/api/artist_api.html#matplotlib.artist.Artist)

  - 图上可见的所有东西都是`artist`
    - `Figure`, `Axes`, `Axis`都是
  - 包括 `Text`, `Line2D`, `collection`, `Patch`...
  - 图像渲染时所有的`artist`被画到**canvas**上
  
- 绘制函数的输入类型

  - 绘制前最好转换成`np.array`

  - ```python
    a = pandas.DataFrame(np.random.rand(4,5), columns = list('abcde'))
    a_asarray = a.values
    
    b = np.matrix([[1,2],[3,4]])
    b_asarray = np.asarray(b)
    
    ```

- `Matplotlib` vs `pyplot` vs `pylab`

  - `Matplotlib`是整个库
  - `pyplot`是里面一个模块
  - `pyplot`里面的函数都作用于当前图像 (状态机)
  - `pylab`是为了方便将`pyplot`和`numpy`一起`import`
  - `pylab`已过气

- Coding Styles

  - 用`pyplot`接口创建图像

  - 其他全部用对象的方法完成

  - 对不同数据集重复创建某个图的推荐方法:

  - ```python
    def my_plotter(ax, data1, data2, param_dict):
        """
        A helper function to make a graph
    
        Parameters
        ----------
        ax : Axes
            The axes to draw to
    
        data1 : array
           The x data
    
        data2 : array
           The y data
    
        param_dict : dict
           Dictionary of kwargs to pass to ax.plot
    
        Returns
        -------
        out : list
            list of artists added
        """
        out = ax.plot(data1, data2, **param_dict)
        return out
    
    # which you would then use as:
    
    data1, data2, data3, data4 = np.random.randn(4, 100)
    fig, (ax1, ax2) = plt.subplots(1, 2)
    my_plotter(ax1, data1, data2, {'marker': 'x'})
    my_plotter(ax2, data3, data4, {'marker': 'o'})
    
    ```

- 后端

  - `Cocoa`后端`%matplotlib osx​`
  - 与人交互
    - [`matplotlib.pyplot.ion()`](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.ion.html#matplotlib.pyplot.ion)
    - [`matplotlib.pyplot.ioff()`](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.ioff.html#matplotlib.pyplot.ioff)
    - 交互模式每个调用都会立刻显示出结果
    - 可用[`draw()`](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.draw.html#matplotlib.pyplot.draw)刷新
    - 非交互模式用[`show()`](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.show.html#matplotlib.pyplot.show)输出
  - 输出文件

- 性能

  - 画线

    - `path.simplify`是否简化线段
    - `path.simplify_threshold`简化阀值

  - ```python
    import numpy as np
    import matplotlib.pyplot as plt
    import matplotlib as mpl
    
    # Setup, and create the data to plot
    y = np.random.rand(100000)
    y[50000:] *= 2
    y[np.logspace(1, np.log10(50000), 400).astype(int)] = -1
    mpl.rcParams['path.simplify'] = True
    
    mpl.rcParams['path.simplify_threshold'] = 0.0
    plt.plot(y)
    plt.show()
    
    mpl.rcParams['path.simplify_threshold'] = 1.0
    plt.plot(y)
    plt.show()
    ```

  - 其他的性能优化暂时用不到,就先放着吧

