# Assignment 1

ROBOCON 视觉组 Assignment 2 作业仓库。

测试平台：Lenovo Legion R9000P（AMD Ryzen 9 8945HX + NVIDIA GeForce RTX 5060 Laptop GPU + AMD Radeon 集成显卡），Ubuntu 24.04.4 LTS。

---

## 1. System Information

### 1.1 环境信息汇总

| 项目 | 值 |
| --- | --- |
| Ubuntu 版本 | Ubuntu 24.04.4 LTS (Noble Numbat) |
| Kernel 版本 | 7.0.0-30-generic |
| CPU | AMD Ryzen 9 8945HX with Radeon Graphics，16 核 / 32 线程，最高 5462.71 MHz |
| GPU | NVIDIA GeForce RTX 5060 Laptop GPU（独显，`[10de:2d59]`，8151 MiB 显存）<br>AMD/ATI Raphael（集成显卡，`[1002:164e]`） |
| GPU 正在使用的内核驱动 | NVIDIA → `nvidia`；AMD 集成显卡 → `amdgpu` |
| 图形会话类型 | **X11** |
| NVIDIA Driver | 595.84 |
| CUDA Toolkit | **未安装（N/A）** |

### 1.2 Ubuntu 版本

```bash
cat /etc/os-release
```

```text
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
```

### 1.3 Kernel 版本

```bash
uname -r
```

```text
7.0.0-30-generic
```

### 1.4 CPU

```bash
lscpu
```

```text
架构：                     x86_64
CPU:                       32
厂商 ID：                  AuthenticAMD
  型号名称：               AMD Ryzen 9 8945HX with Radeon Graphics
    CPU 系列：             25
    型号：                 97
    每个核的线程数：       2
    每个座的核数：         16
    座：                   1
    CPU 最大 MHz：         5462.7109
    CPU 最小 MHz：         427.8030
Virtualization features:
  虚拟化：                 AMD-V
Caches (sum of all):
  L1d:                     512 KiB (16 instances)
  L2:                      16 MiB (16 instances)
  L3:                      64 MiB (2 instances)
```

> 完整输出还包含 `标记`（指令集标志位）与 `Vulnerabilities` 两段长内容，为保持可读性此处从略。物理核数 = 16 × 1 = 16，逻辑核数 = 16 × 2 = 32。

### 1.5 GPU

```bash
lspci | grep -Ei 'vga|3d|display'
```

```text
01:00.0 VGA compatible controller: NVIDIA Corporation Device 2d59 (rev a1)
05:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Raphael (rev d8)
```

```bash
lspci -nn | grep -Ei 'vga|3d|display'
```

```text
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:2d59] (rev a1)
05:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Raphael [1002:164e] (rev d8)
```

`lspci` 只认出 NVIDIA 卡的 PCI 设备号 `2d59`，未给出型号，需用 `nvidia-smi` 确认（见 1.8）。

### 1.6 GPU 正在使用的内核驱动

```bash
lspci -k | grep -EA3 'VGA|3D|Display'
```

```text
01:00.0 VGA compatible controller: NVIDIA Corporation Device 2d59 (rev a1)
	Subsystem: Lenovo Device 3e29
	Kernel driver in use: nvidia
	Kernel modules: nvidiafb, nouveau, nvidia_drm, nvidia
--
05:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Raphael (rev d8)
	Subsystem: Lenovo Raphael
	Kernel driver in use: amdgpu
	Kernel modules: amdgpu
```

NVIDIA 卡实际加载的是专有驱动 `nvidia`（`nouveau` 仅出现在候选模块列表中，未被启用）；AMD 集成显卡加载内核自带的 `amdgpu`。

### 1.7 图形会话类型

```bash
echo "$XDG_SESSION_TYPE"
echo "$DISPLAY"
echo "$WAYLAND_DISPLAY"
```

```text
x11
:1

```

当前为 **X11 会话**：`XDG_SESSION_TYPE` 给出类型，`DISPLAY` 为 `:1` 说明存在 X 服务器，`WAYLAND_DISPLAY` 为空说明没有 Wayland compositor 在运行。

### 1.8 NVIDIA Driver

```bash
nvidia-smi
```

```text
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 595.84                 Driver Version: 595.84         CUDA Version: 13.2     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 5060 ...    Off |   00000000:01:00.0 Off |                  N/A |
| N/A   40C    P4              8W /   50W |      15MiB /   8151MiB |      9%      Default |
+-----------------------------------------+------------------------+----------------------+
```

NVIDIA Driver 版本为 **595.84**。上表默认输出会截断较长的卡名，改用 `--query-gpu` 取完整信息：

```bash
nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv
```

```text
name, driver_version, memory.total [MiB]
NVIDIA GeForce RTX 5060 Laptop GPU, 595.84, 8151 MiB
```

### 1.9 CUDA Toolkit

```bash
nvcc --version
```

```text
找不到命令 “nvcc”，但可以通过以下软件包安装它：
sudo apt install nvidia-cuda-toolkit
```

**本机未安装 CUDA Toolkit。** 本作业的 C++ 部分只需 OpenCV 与 Eigen，不需要 CUDA，因此无需安装。

### 1.10 关于 CUDA 版本的重要区分

`nvidia-smi` 输出中的 `CUDA Version: 13.2` **不能**理解为「本机安装了 CUDA 13.2 Toolkit」，二者含义不同：

| | 含义 | 本机情况 |
| --- | --- | --- |
| `nvidia-smi` 中的 `CUDA Version` | 该 **NVIDIA 驱动所支持的最高 CUDA 运行时版本**，是驱动能力的上限，随驱动一起提供 | 13.2 |
| CUDA Toolkit | **实际安装的开发工具链**（含 `nvcc` 编译器、头文件、运行时库），需单独安装 | **未安装（N/A）** |

即：本机驱动有能力运行面向 CUDA 13.2 及以下版本构建的程序，但因未安装 Toolkit，**无法在本机编译** CUDA 代码——`nvcc` 命令不存在即为直接证据。

---

## 2. Python Project A

Project A 代码位于 `python_A/`，功能为：打开摄像头 → 持续读取图像 → 显示原始图像 → 灰度化 → 轮廓提取 → 持续运行 → `q` / `ESC` 退出 → 保存未经处理的原始摄像头视频。

### 2.1 环境创建与激活

项目 `pyproject.toml` 声明 `requires-python = ">=3.9,<3.11"`，因此选择 Python 3.10。**未修改 `pyproject.toml` 中的任何约束。**

```bash
conda create -n robocon-a python=3.10 -y
conda activate robocon-a
```

```text
Preparing transaction: done
Verifying transaction: done
Executing transaction: done

To activate this environment, use
    $ conda activate robocon-a
```

### 2.2 解释器确认

```bash
python --version
which python
```

```text
Python 3.10.21
/home/audrey/miniconda3/envs/robocon-a/bin/python
```

`which python` 指向 `envs/robocon-a/bin/python`，确认解释器来自新建的 Conda 环境，而不是 `base`。

### 2.3 依赖安装

```bash
cd ~/ROBOCON-Vision-Assignment-1/python_A
python -m pip install -e . -i https://pypi.tuna.tsinghua.edu.cn/simple
```

> 命令末尾的 `-i` 指定了清华 PyPI 镜像以加快下载；`. ` 表示安装当前目录的项目，`-e` 为可编辑安装。加上 `.` 之后 pip 会读取 `pyproject.toml` 并强制执行其中声明的 `requires-python` 与依赖版本范围。

```text
Obtaining file:///home/audrey/ROBOCON-Vision-Assignment-1/python_A
Collecting numpy<2.0,>=1.26 (from robocon-vision-assignment1-project-a==1.0.0)
Collecting opencv-python<5.0,>=4.9 (from robocon-vision-assignment1-project-a==1.0.0)
Successfully installed numpy-1.26.4 opencv-python-4.11.0.86 robocon-vision-assignment1-project-a-1.0.0
```

安装过程中 pip 依次尝试了 opencv-python 的 4.14.0.94、4.13.0.92、4.13.0.90、4.12.0.88 四个版本，最终回退到 **4.11.0.86**。原因是 OpenCV 4.12 及以上要求 `numpy>=2`，与项目声明的 `numpy>=1.26,<2.0` 冲突。这说明 pip 确实在读取并服从 `pyproject.toml` 的约束，而不是绕过它。

验证安装结果：

```bash
python -c "import cv2, numpy; print(cv2.__version__, numpy.__version__)"
```

```text
4.11.0 1.26.4
```

### 2.4 运行

```bash
python camera.py
```

```text
================================================================
ROBOCON Vision Assignment 1 - Python Project A
PID:          28680
PPID:         9628
Python:       /home/audrey/miniconda3/envs/robocon-a/bin/python
Python ver.:  3.10.21
Camera index: 0
Raw output:   /home/audrey/ROBOCON-Vision-Assignment-1/python_A/raw_capture.mp4
Keep this process running and inspect it from another terminal.
Press q or ESC in an OpenCV window to exit.
================================================================
Actual stream: 1280x720, writer FPS=10.00
Saved raw video: /home/audrey/ROBOCON-Vision-Assignment-1/python_A/raw_capture.mp4
Captured frames: 911
Elapsed time:    93.3 s
Loop rate:       9.8 frame/s
```

程序连续运行 **93.3 秒**（要求 ≥30 秒），按 `q` 退出。

### 2.5 输出视频

```bash
ls -lh raw_capture.mp4
```

```text
-rw-rw-r-- 1 audrey audrey 27M  9月 26 16:42 raw_capture.mp4
```

输出为**未经任何处理的原始摄像头视频**：代码中 `writer.write(frame)` 在 `process_frame()` 之前执行，写入的是 `capture.read()` 拿到的原始帧。

### 2.6 图像结果截图

![Project A 原始图像 / 灰度图像 / 轮廓图像](assets/python_a/three_windows.png)

三个窗口（Original / Grayscale / Contours）同时来自正在运行的 Project A。

---

## 3. Process Observation

Project A 在启动时会打印自己的 PID。本节的做法是：让程序在**终端 1** 中持续运行，另开**终端 2**，完全从系统层面独立找出这个进程，再与程序自己打印的 PID 核对。

### 3.1 查找过程

**第一步：用 `pgrep` 按完整命令行匹配**

```bash
pgrep -af "python camera.py"
```

```text
29552 python camera.py
```

`-f` 让 pgrep 匹配**完整命令行**而非仅匹配进程名。这一步是必需的：该进程的进程名实际是 `python`，若不加 `-f` 而直接搜 `camera.py`，不会有任何结果。`-a` 顺带把命令行打印出来，便于核对。

> 匹配串写成 `"python camera.py"` 而不是裸的 `"camera.py"`，是因为 `-f` 做的是纯文本匹配，任何命令行中碰巧含有该字符串的进程都会被命中——实测中 `pgrep -af "camera.py"` 会把执行该命令的 shell 自身也匹配出来。

**第二步：用 `ps` 一次取出全部所需字段**

```bash
ps -o pid,ppid,cmd,%cpu,%mem,etime -p $(pgrep -f "python camera.py")
```

```text
    PID    PPID CMD                         %CPU %MEM     ELAPSED
  29552    9628 python camera.py             139  0.7       05:59
```

`$(...)` 是命令替换：先执行内层 `pgrep` 得到 PID，再把结果填入 `ps` 的 `-p` 参数位置。`-o` 指定输出列：

| 列 | 含义 | 实测值 |
| --- | --- | --- |
| `pid` | PID | **29552** |
| `ppid` | PPID（父进程 PID） | **9628** |
| `cmd` | CMD（完整命令行） | **python camera.py** |
| `%cpu` | CPU 占用百分比 | **139** |
| `%mem` | 物理内存占用百分比 | **0.7** |
| `etime` | 已运行时间（elapsed time） | **05:59** |

**第三步：用 `pstree` 查看该进程下的线程**

```bash
pstree -p $(pgrep -f "python camera.py")
```

```text
python(29552)─┬─{python}(29553)
              ├─{python}(29554)
              ├─{python}(29555)
              ├─{python}(29556)
              │      ...
              ├─{python}(29614)
              └─{python}(29615)
```

`-p` 显示各节点的 PID，花括号 `{}` 包裹的就是线程。该进程共 **63 个线程**，线程 PID 从 29553 连续排到 29615。

### 3.2 PID 核对

| 来源 | PID |
| --- | --- |
| 程序自己打印（终端 1） | 29552 |
| 从系统中查出（终端 2 的 `pgrep`） | 29552 |

**两者一致。**

### 3.3 htop 截图

![htop 进程与线程观察](assets/process/htop.png)

截图前的操作：按 `F4` 输入 `camera.py` 过滤出目标进程，再按 `H` 打开线程显示。

图中可确认两项内容：

- **整机使用情况**：32 个逻辑核各自的占用条、内存 `8.10G/30.6G`、`Tasks: 179, 1772 thr`、`Load average: 2.20 1.80 1.10`、`Uptime: 05:27:19`。
- **该进程占用的线程**：主进程 `PID 29552`（`TIME+` 为 `3:14.93`），其下 63 个线程缩进排列，每个线程单独列出 `CPU%` 与 `TIME+`（约 `0:23`）。

### 3.4 CPU 139% 与 63 个线程的来源

`ps` 报告的 `%CPU` 为 139%，**超过 100%**。这是因为该列是跨所有核心的累计值：本机有 32 个逻辑核，单线程任务满负荷也只有 100%，139% 说明该进程同时在多个核心上运行。

线程数 63 则来自 OpenCV 与 Qt：

```bash
python -c "import cv2; print(cv2.getNumThreads())"
```

```text
32
```

```bash
python -c "import cv2; print(cv2.getBuildInformation())" | grep -E 'GUI|Parallel'
```

```text
  GUI:                           QT5
  Parallel framework:            pthreads
```

OpenCV 使用 pthreads 作为并行后端，默认按 CPU 数量（32）创建工作线程池；Qt5 GUI 后端又为三个 `cv2.imshow` 窗口另开事件线程——两者相加即 `pstree` 中看到的 63 个线程，也是该进程 CPU 时间的主要来源。

---

## 4. Python Project B

> 待完成。需要记录：Project B 的 `conda create` / `conda activate` / `python --version` / `which python` / `pip install` / `python analyze_video.py` 实际命令与输出，以及输出 MP4 的路径。
>
> 还需说明：Project A 使用的 Conda 环境与 Python 版本、Project B 使用的 Conda 环境与 Python 版本、以及**为什么两个项目不能共用一个环境**（两个 `pyproject.toml` 声明的 Python 版本范围不兼容）。

---

## 5. C++ Manual Build

> 待完成。需要原样记录**实际成功使用的那一条完整 `g++` 命令**，并回答：
>
> - `-I` 的作用是什么？
> - 为什么 `transform.hpp` 不单独作为一个 cpp 文件编译？
> - 为什么只写 `main.cpp` 往往无法得到完整程序？
> - 编译成功后产生的文件是什么？

---

## 6. CMake Build

> 待完成。需要记录：自写的 `CMakeLists.txt` 完整内容、`cmake -S . -B build`、`cmake --build build`、从 `build/` 运行可执行文件的命令与运行结果。
>
> 还需回答：手工 `g++` 命令和 CMake 的关系是什么？

---

## 7. Git / GitHub

> 待完成。需要记录实际执行过的关键命令：`git status` / `git add` / `git commit` / `git branch` / `git switch` / `git push` / `git log --oneline --graph --all`。
>
> 要求：至少 3 个有意义的 commit；至少创建并使用过 1 个非 main 分支，且该分支的修改最终回到主分支；不得提交 Conda 环境目录、`build/`、大型原始依赖库。

---

## 8. Problems and Notes

- `lspci` 把 NVIDIA 显卡识别为 `Device 2d59` 而非具体型号，原因是本机 `pci.ids` 硬件数据库尚未收录该型号（RTX 5060 Laptop GPU 较新）。准确的型号来源是 `nvidia-smi --query-gpu=name --format=csv`。这属于数据库未更新，不是硬件或驱动故障。
- 默认的 `nvidia-smi` 表格输出会因列宽限制截断较长的 GPU 名称，取完整名称需使用 `--query-gpu` 参数。
- `nvidia-smi` 报告的 GPU Bus-Id 为 `00000000:01:00.0`，与 `lspci` 报告的 `01:00.0` 一致，可确认二者指向同一块物理显卡。
- 本机为双显卡（NVIDIA 独显 + AMD 集成显卡）且运行在 X11 下，后续 Part II 调用摄像头时需留意默认渲染设备。
- README 编写过程中，`cat` 曾出现「找不到命令」的报错，实际原因是命令输入时的拼写/大小写问题，系统本身 `cat` 位于 `/usr/bin/cat` 且 `/usr/bin` 在 `PATH` 中，与系统环境无关。
- **Project A 首次运行时三个窗口全黑。** 程序输出 `Captured frames: 646`、`Elapsed time: 113.4 s`，即摄像头在按 10 fps 正常吐帧；但生成的 `raw_capture.mp4` 仅 791 KB，约合 **1.2 KB/帧**，而 1280×720 的真实画面每帧压缩后应有几十 KB，说明帧内容是近乎全黑的底噪。排查过程：`/sys/class/video4linux/video0/name` 与 `udevadm info` 确认设备为 `174f:246f Syntek Integrated Camera` 且 `ID_V4L_CAPABILITIES=:capture:`，UVC 驱动已正常绑定，`fuser` 确认无其他进程占用——硬件、驱动、代码均无问题，最终确认是**摄像头隐私快门未打开**。打开快门后重录，得到 27 MB / 911 帧的正常视频。
