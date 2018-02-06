+++
title = "计算格林函数"
enable_toc = true
weight = 2
[menu.main]
parent = "procedures"
+++

使用 gCAP 反演震源机制的第一步是正确计算格林函数。默认情况下，本项目中 gCAP 将 `Glib` 作为格林函数库所在目录。

<!--more-->
## 文件说明

`Glib` 目录下包含了一个格林函数的示例： `run_fk.pl`、 `run_parallel.pl`、 `config.pm` 和 `model.fk`。

`run_fk.pl` 和 `run_parallel.pl`是计算格林函数的脚本，区别在于后者是进行的并行计算，这两个脚本都需要模块文件 `config.pm`。
在实际使用时，这三个文件不需要修改。至少，我的设计是不需要修改。`model.fk` 是示例计算格林函数所用的配置文件，里面包含了示例的一维地层等信息。

### 格林函数配置文件

`model.fk` 的内容如下：

    # FK configuration
    DIST: 135/145/5 210 235 260 280 300 415
    DEPTH: 5 10/20/5 25
    NT: 512
    DT: 0.2
    FLAT: NO               # NO or YES. if undefined, scripts will use YES

     5.5 3.18 5.50 2.53 600 1200
    10.5 3.64 6.30 2.79 600 1200
    16.0 3.87 6.70 2.91 600 1200
    90.0 4.50 7.80 3.27 900 1800

说明如下：

1. `#` 号后的内容对程序没有作用，只是注释；空行也没有作用
2. DIST 意义是指明哪些震中距的格林函数，其中 `135/145/5` 等于计算 135、140 和 145 三个震中距
3. DEPTH 是设置深度
4. NT 是设置数据点数
5. DT 是设置采样周期
6. FLAT 是设置是否需要展平变换
7. 以 `NT: 512` 这一行来说，其中的冒号是必须的，而且一定要是英文冒号，冒号后的空格可以不要。
8. 程序支持的参数是 DIST、DEPTH、NT、DT 和 FLAT，没有其他的。
9. 最下面几行是一维地层模型，这部分的格式为：地层厚度  S波速度  P波速度  密度  S波Q值  P波Q值，后三项可以省略。

如何正确设置计算的参数，请参考 [fk用法笔记](https://seisman.info/fk-notes.html)。

## 计算示例格林函数

运行脚本 `run_fk.pl` 即可计算得到示例所需的格林函数。

    $ perl run_fk.pl model.fk

如果安装了 Perl 的并行模块 Parallel::ForkManager，你可以使用脚本 `run_parallel.pl`，用并行的方式完成上述计算：

    $perl run_parallel.pl model.fk

## 格林函数库

计算完成后会得到 5 个名为 `model_xx`（`xx` 代表震源深度）的文件夹，其中包含了震源深度取不同值所对应的格林函数。

以 `model_05` 目录为例，其中包含了众多文件名为 `xxx.grn.x` 的格林函数文件。其中：

- `xxx` 代表震中距
- `x` 取0-8，对应双力偶震源的9个格林函数分量
- `x` 取a、b、c，对应爆炸源的3个格林函数分量

所有格林函数均是 SAC 格式，其中 `xxx.grn.0` 和 `xxx.grn.5` 的SAC 头段 `t1` 和 `t2` 中分别保存了初至 P 波和 初至 S 波的到时，SAC 头段 `user1` 和 `user2` 中保存了初至 P 波和初至 S 波的离源角（相对于垂直向下方向旋转的角度)。

gCAP 可以根据 SAC 头段 `t1` 和 `t2` 的值自动确定要截取的波形时间窗；若反演中需要使用震相初动极性信息，则需要从 SAC 头段 `user1` 和 `user2` 中读取震相离源角。