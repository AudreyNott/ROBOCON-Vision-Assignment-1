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

> 待完成。需要记录：`conda create` / `conda activate` / `python --version` / `which python` / `pip install` / `python camera.py` 的**实际执行命令与输出**；程序连续运行不少于 30 秒；退出后生成的 `raw_capture.mp4` 路径。
>
> 必须截图：原始图像 + 灰度图像 + 轮廓图像三个窗口同时来自运行中的 Project A（存入 `assets/python_a/`）。

---

## 3. Process Observation

> 待完成。需要记录：从系统中查找 Project A 进程的**完整查找过程**（用了哪些 `ps` / `pgrep` / 管道组合），以及程序自己打印的 PID 与查到的 PID 的核对结果。
>
> 至少确认：PID、PPID、CMD、CPU %、MEM %、运行时间。
>
> 必须截图：`htop`（能在图中看到整机使用情况与占用线程），存入 `assets/process/`。

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
