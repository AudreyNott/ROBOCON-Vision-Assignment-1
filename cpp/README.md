# C++ Task Starter

## 文件结构

```text
cpp/
├── CMakeLists.txt
├── include/
│   └── transform.hpp
└── src/
    ├── main.cpp
    └── transform.cpp
```

`CMakeLists.txt` 是完成本 Assignment 后新增的文件；`include/` 与 `src/` 下的源码为任务开始时已提供的内容。

> 本文件是任务开始前提供的起步说明。原文此处为「仓库中**没有 `CMakeLists.txt`**，这是 Assignment 的一部分，不是遗漏」——当时该文件确实尚未创建。完成本 Assignment 后它已存在，内容见根目录 `README.md` 第 6 章。

## 程序功能

程序读取 Project A 保存的原始 MP4，并输出三个并排画面：

```text
原始视频 | Otsu 二值化 | Canny 边缘
```

程序使用：

- OpenCV：视频读取/写入、灰度化、Gaussian blur、Otsu、Canny；
- Eigen：根据画面平均 BGR 计算场景亮度，并参与 Canny 阈值计算。

## 版本要求

```text
C++ standard: C++17
OpenCV:       >= 4.5, < 5.0
Eigen:        >= 3.3, < 4.0
GCC:          >= 9 recommended
Clang:        >= 10 recommended
CMake:        >= 3.16 recommended
```

Ubuntu 通常需要安装 C++ development packages，例如：

```text
libopencv-dev
libeigen3-dev
```

注意：Python 的 `opencv-python` 包不能替代 C++ 所需的 OpenCV development headers/libraries。

## 程序接口

编译得到可执行文件后，接口为：

```text
PROGRAM INPUT.mp4 [OUTPUT.mp4]
```

如果省略输出路径，默认输出：

```text
cpp_processed.mp4
```

学生需要先自行完成一次手动编译，再自行编写 `CMakeLists.txt`。这两步均已在仓库中完成：手工编译命令见根目录 `README.md` 第 5 章，`CMakeLists.txt` 的内容与构建过程见第 6 章。
