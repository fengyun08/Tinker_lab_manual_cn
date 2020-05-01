# BME 346/383J
# <center>Tinker 实验指南</center>

[TOC]

## <center>实验1 用 Tinker 做分子模拟 (1)</center>

请记录（至少20行）每次计算的文本输出，并做好 VMD 截图，然后放到实验报告中。

重点：Tinker 通常被安装在你不能写入的实验室系统目录。在你开始学习教程前，请先将 Tinker 软件 bin, params, test, 和 example 目录复制到自己的工作目录（例如Document）。如果你想保存文件随时随地用他们工作，可以拷到闪存或移动盘中。

1. 关于 Tinker

  Tinker 是华盛顿大学医学院开发的分子模拟包。可以从[他们的网站](http://dasher.wustl.edu)免费下载所需的 Mac, Windows 或 Linux 平台版 Tinker 软件安装包。

  Tinker 运行在命令行下，因此很便于自动化脚本运行。可以用 VMD, PyMol 或 FFX 可视化分子结构。

  这是 Windows 版下载地址：http://biomol.bme.utexas.edu/~pren/courses/bme346-soft

2. 使用 Windows 的 DOS 命令提示

  如果使用 Mac, 可以在“终端”执行相似的操作

  单击开始按钮，键入 cmd 命令并回车

  - 改变目录，用 cd 命令：例如，cd /d C: 或 cd test 或 cd ../ 之类命令。
  - 列出文件夹中的文件和目录：例如，dir 或 dir /w 等等。
  - 执行命令（可执行二进制文件）：例如，analyze.exe test.xyz
    上述例子中，analyze.exe 和 test.xyz 不在工作目录和搜索路径上，因此一般要输入完整路径。
  - 浏览和编辑文本文件，使用记事本或其他文本编辑器：例如，notepad filename
    如果是有格式文件，则使用“写字板”打开即可。例如，write filename

3. 用 VMD 或 PyMol 可视化结构

  - VMD 可以可视化 Tinker 的 xyz 文件：
    - 先单击菜单 File | New Molecule
    - 点 Load files 装入新分子
    - 浏览并选取你的 xyz 文件
    - 确定文件类型，必须选择 Tinker
    - Load 装入
  - VMD 也能装入 Tinker 的 arc 档案文件（包含很多结构，用于分析 MD 轨道和制作模拟动画）。只要装入完成，就可以点击 VMD 主窗口底部的箭头按钮，播放模拟动画。
  - PyMol 只能打开 xyz 文件，不能打开 arc 档案文件。
  - 可以尝试从 tinker/example 目录装入一个 xyz 文件。

4. 用 Tinker 做计算

  - 打开 cmd 命令窗口
  - 用 cd 切换到工作目录到 example
  - 在开始下节内容前，先读完前节的内容

  ```
  重要信息
  提示：*.xyz 和 *.key 两个文件是做 Tinker 计算必须的。例如，可以打开 butane.xyz 和 butane.key 两个文件观看文件内容。

  *.xyz 文件:
        第1列是索引列
        第2列是原子符号
        3-5 列是 x,y,z 坐标，单位是埃
        第6列是定义在 key 文件中的原子类型
        第7列至最后列，是连接到当前原子的原子列表
  *.key 文件:
        key 文件可以直接包含所有参数，或者在第1行连接多实际的参数文件 (*.prm)。参数基于原子类型，规定了原子之间的键、角、扭角、范德华和静电相互作用。
  如果看到有关的 OMP 错误，请在 key 文件中将 OPEN-THREADS 设置为一个小于 CPU 核心数的值，该变量设置并行计算中使用的 CPU 核心数。
  ```

  4.1 能量分析 (analyze.exe)

  - 在 Windows 中运行 cmd 命令，弹出命令窗口
  - 键入 cd xxxx 进入拷贝的 example 工作目录
  - 键入下列命令，运行 analyze.exe 程序：
    ```bash
    ..\bin\analyze.exe bearing.xyz```
  - 程序提问时选择不同的应答选项（可以组合多项选择）：
    - E 指能量
    - P 列出所用参数
    - M 列出分子电矩
    - 或上面选项的任意组合
  - 尝试所测试目录的另一分子
  - 转存结果到文本形式的日志文件，以便以后在文本编辑器中观看内容。命令如下：
    ```bash
    ..\bin\analyze.exe bearing.xyz e > bearing.log```

  4.2 能量最小化 (minimize.exe)

  - 重复上面前两步，然后运行：
    ```bash
    ..\bin\minimize.exe bearing.xyz 0.0001```
  注意，在最小化后会生成一个新的 bearing.xyz_2 文件.任何操作 bearing.xyz 的新命令，都应该使用这个最新生成的bearing.xyz_2（只要没删除它，后续执行最小化会依次产生如 bearing.xyz_3 之类后缀编号递增的新文件，新命令依次也要改为使用更新的后续文件）。最后，使用 VMD 观看最小化后的结构文件（文件类型要指定为 Tinker）。

  ```
  Tinker 中还有可以替代的能量最小化命令：optimize.exe, newton.exe.
  "Minimize" 使用低存储BFGS非线性优化算法 (J.Nocedal and S. J. Wright, "Numerical Optimization", Springer-Verlag New York, 1999, Section 9.1, http://en.wikipedia.org/wiki/Limited-memory_BFGS)
  "Optimize" 使用最优条件可变度量方法 (D. F. Shanno and K-H. Phua, Journal of Optimization Theory and Applications, 25, 507-518, 1978)
  "Newton" 使用拟牛顿方法 (R. S. Dembo and T. Steihaug,......
  ```

  4.3 简振模分析 (NMA)

  - 使用前面生成的 bearing.xyz_2 文件（最小化到 0.0001）
  - 删除名为 bearing.001, bearing.002 之类的文件，应该只留下 bearing.xyz, bearing.xyz_2 和 bearing.key 文件。
  - 使用 "vibrate.exe" 计算 Hessian 矩阵和简振模频率。
    注意，因为分析的是最小化后的结构，所以前 6 个本征值应该为 0 (如果没做最小化，你会得到负数值)。
    你可以指定要输出的振动模结构。例如，指定第 9 或 20 振动模，那么 vibrate 将创建的文件名分别为 bearing.009 和 bearing.020. 也可以通过输入 "9 20" 同时指定多个振动模。在 VMD 中这些文件将会播放成动画，演示特定振动模的运动。
    如果还要将屏幕显示的文本输出转存到文件，可以这样敲命令：
    ```bash
    ..\bin\vibrate.exe bearing.xyz_2 9 > bearing.log```
    输出的 bearing.log 包含了简振模分析中的全部振动频率。选用的数字 9 会记录第 9 振动模，但是你可以再尝试其他任意振动模。

  4.4 分子动力学模拟 (dynamic.exe)

  - 使用前面生成的 bearing.xyz_2 文件（最小化到 0.0001）
  - 删除除了 bearing.xyz, bearing.xyz_2 和 bearing.key 之外形如 bearing.* 名字的任何文件。
  - 打开 bearing.key 文件（用 write bearing.key 命令），在文件底部加入关键字 archive 到单独一行，保存后退出。
  - 从最新的最小化结构 bearing.xyz_2 开始，执行常温 MD 模拟。可以简单运行 <font color="red"><b>..\bin\dynamic.exe bearing.xyz_2</b></font>, 然后按提示交互输入参数；也可以运行 <font color="red"><b>..\bin\dynamic.exe bearing.xyz_2 10000 1.0 0.1 2 500 > mydynamic.log</b></font> 一次性输入全部命令参数。（只要按交互模式运行命令，就会了解这些命令参数的含义）。
    即，使用时间步长为 1fs, 模拟时间 10000 步（即 10ps），然后每隔 0.1ps 保存输出结构，选择常温选项，并设定模拟温度为 500K.
  - 在 VMD 中可视化轨道和播放动画 (bearing.arc 文件)

  ```
  如要用广义玻恩方法模拟隐含溶剂效应，则要用 wordpad 打开 bearing.key 文件加入下列行
          solvate GBSA
  ```
