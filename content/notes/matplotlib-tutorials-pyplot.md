+++
title = "matplotlib pyplot"
weight = 1
order = 1
date = 2020-02-10
#insert_anchor_links = "right"

[taxonomies]
tags = ["notes"]

+++

## `pyplot`介绍

- [`matplotlib.pyplot`](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.html#module-matplotlib.pyplot)是一些命令式函数集合

- 为了让`matplotlib`像`MATLAB`一样工作

- 状态机: 当前图像、当前绘制区域、当前`axes`得到保留

- 推荐使用`matplotlib`介绍里面的面相对象`API`

- ```python
  import matplotlib.pyplot as plt
  import numpy as np
  
  # matplotlib assumes it is a sequence of y values
  # x data are [0,1,2,3]
  plt.plot([1, 2, 3, 4])
  plt.ylabel('some numbers')
  plt.show()
  
  fig, ax = plt.subplots()
  ax.plot([1, 2, 3, 4])
  ax.set_ylabel('some numbers')
  fig.show()
  ```

- [`axis()`](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.axis.html#matplotlib.pyplot.axis)

  - `axis([xmin, xmax, ymin, ymax])`

- [`plot()`](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.plot.html#matplotlib.pyplot.plot)

  - ```python
    plot([x], y, [fmt], *, data=None, **kwargs)
    plot([x], y, [fmt], [x2], y2, [fmt2], ..., **kwargs)
    ```

  - ```python
    fmt = '[marker][line][color]'
    ```
    
  - `fmt`详细格式看链接里面**Format Strings**
