## <center>实验3 基于热力学的蛋白质工程</center>

实验报告需上交 RMSD 图（任务1）、突变自由能和计算自由能用到的所有能量(E)值（任务2）

### 任务1. 蛋白质动力学

1. 设置并运行 trpcage 小蛋白 (1L2Y) 在隐含溶剂中的分子动力学模拟。

  - 在 tinker\example 目录，删除除 trpcage.pdb 和 trpcage.key 以外的所有 trpcage.* 文件
  - 修改 trpcage.key 文件的内容如下：

  ``` bash
  parameters   ..\params\amber99sb.prm
  archive
  solvate   GBSA
  integrate   stochastic
  rattle   bonds
  OPENMP-THREADS   2   # 不要超过计算机 CPU 的实际线程数
  ```

  - 可以象前面两次实验一样，使用 FFE 将 trpcage.pdb 转换为 trpcage.xyz, 或者在 cmd 窗直接敲 %TINKDIR%\bin\pdbxyz trpcage.pdb 实现。
  - 使用 FFE 或命令行，最小化 (minimize) trpcage.xyz 并运行 MD 模拟。
    - 设置温度为 298K, 运行 20ps（或 10000 步），时间步长为 2fs, 每隔 1ps 转存一次结构。这个短时模拟实验仅用于示范，不可用于真实模拟。

2. 通过使用 FFE(VMD 也可以) 计算 MD 轨道相对初始结构的 RMSD, 检查结构的波动。

  - 装入 trpcage.arc 轨道文件。FFE 需要与结构文件正确配对的 key 文件，如果装入轨道后没有任何显示，通常是因为丢失了 key 文件，或者参数路径不对（也许你用了相对路径，却移走了参数文件）
  - 再装入 trpcage.xyz 晶体结构文件。当 trpcage 显现时，选择 Modeling Commands 卡片中的 Superpose 列表项。去掉氢原子进行比较，其它所有选项不用管。
  - 单击 Go 按钮，等日志文件生成后，其中可发现初始结构 trpcage.xyz 和 trpcage.arc 轨道每帧数据（共 20 帧）之间的 RMSD 值。
  - 记录所有 RMSD 值，并把它绘制成时间的函数曲线。记住，每 ps 转存每帧数据。当 Tinker 输出两个一样的 RMS 时，请移除多余数据。
    - 若使用 Mac 系统，可以先 cd 进入工作目录，然后在终端窗口敲 grep Root xxx.log 命令搜索并打印日志文件包含 Root 的所有行。
    - 若使用 Windows 系统，且安装了 notepad++ 文本编辑器，则可以使用 Search / Find 实现同样的功能。
    - 若在 Windows 系统内安装了 Unix 子系统，则可以使用 shell 和 grep 命令实现同样的功能。

### 任务2. 计算突变引起的自由能改变（$\text{TRP}\to\text{ALA}$）

  突变或配体绑定引起的自由能（稳定性或亲和力）改变，可以利用基于自由能微扰的炼金模拟计算：

  $\Delta A(X\to Y)=-RT\ln\left\langle\frac{e^{E_X-E_Y}}{RT}\right\rangle_X$

  其中，X 是指突变前的 trpcage，而 Y 指突变后 ($\text{TRP}\to\text{ALA}$) 的 trpcage. 系综平均 $\left\langle\cdots\right\rangle_X$ 是指大量构象态对突变前 trpcage 相空间的平均。典型情况需要好几个 ns 的 MD 模拟和 X 与 Y 之间的几个中间微扰步。因为实验的目的，我们只做 1 步从 X 到 Y 的微扰，仅用 trpcage.arc 初态、中间态和末态 3 个结构。

  (1) 用 archive.x 从 trpcage.arc 提取 1ps, 10ps 和 20ps 处的 3 个结构 (%TINKDIR%\bin\archive.exe trpcage.arc)，按指令提示进行操作。再找到提取出的结构，这里应该是 trpcage.001, trpcage.010 和 trpcage.020 (用 dir/w 命令可以检查三个文件是否真的存在)。

  (2) 追加 3 行到 trpcage.key 文件，定义一个 trp 不含 $\beta$ 碳的侧链群。你可以使用 FFE 打开 trpcage.xyz, 并选择残基 trp6 进行检查。$\lambda_{vdw}$ 和 $\lambda_{ele}$ 允许调节该群和群外之间的 vdw 和静电相互作用。$\lambda=1.0$ 意味着完全相互作用；$\lambda=0$ 意味着没有相互作用。使用该方法，移除 trp 侧链（但保留 $\beta$ 碳 CH2），该残基就变成了 ALA
  ```
  ligand -100 108, -111 116
  vdw-lambda 1.0
  ele-lambda 1.0
  ```

  (3) 在命令行制作 key 文件副本，或用文件浏览器将 trpcage.key 复制为 trpcagemut.key, 在后一文件内将两个 $\lambda$ 值都改为 0:

  ```
  ligand -100 108, -111 116
  vdw-lambda 0.0
  ele-lambda 0.0
  ```

  (4) 计算突变前后每个结构的总势能：

  ```
  $TINKDIR/bin/analyze trp.00x -k trpcage.key e
  $TINKDIR/bin/analyze trp.00x -k trpcagemut.key e
  ```
  3 个结构每个都记录 EX 和 EY 值 (上面x=1,2,3)，再用上面的方程计算自由能 $\Delta A$

  注意，在实践中，因为蛋白质的动态特性，需要好几个纳秒 MD 模拟出的成千上万个结构，而不是这里仅有的 3 个结构。而且我们也应逐渐改变 $\lambda$ 值 (1.0,0.9,0.8,...,0), 而不应该仅仅改变为这个实验中 2 步值，对每个 $\lambda$ 值执行 MD 模拟，然后计算总自由能改变为$\Delta A=\Delta A(1\leftrightarrow0.9)+\Delta A(0.9\leftrightarrow0.8)\dots$










