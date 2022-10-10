---
layout: post
title:  "Use C++17 with CUDA 10 and Clang"
date:   2022-10-10 21:20:51 +0800
categories: c++ cuda clang
---
Sometimes we are not able to use newer versions of *CUDA Toolkit* due to the NVIDIA driver version limit. But it may be attractive to use some new C++ features, which are added in new versions of `nvcc` compilers (for example, CUDA 11 starts to support C++17, but you only have CUDA 10.x), in our program. In this case, a solution is to use `clang++` instead of `nvcc` to compile our CUDA program. This can let you enjoy new language features while still targeting older CUDA runtime library.

## About `clang` as a CUDA compiler
The Clang compiler is one of the most famous C-family language compilers in the world. Many people use it to compile their C/C++ programs, but it is less known that Clang (or more precisely, the [LLVM compiler infrastructure][1]) supports CUDA compilation in recent versions. With Clang, when you are to compile your CUDA C/C++ programs, you still need to install one version of CUDA Toolkit that you want to use, but you will not use the `nvcc` compiler shipped with the toolkit.

It is worth mentioning that Clang is not fully compatible with NVCC when compiling CUDA. Details are documented [here][2] (for LLVM version 15). However, many commonly-used language extensions in NVCC are supported in Clang CUDA mode, so it is hopeful that a program can compile without modification.

Note that currently this feature is not supported on Windows.

## Installing Clang
You will need quite new versions of Clang to support targeting recent versions of CUDA Toolkit, so it is highly probable that your Linux distribution does not officially offer such package. LLVM project has prebuilt `deb` packages for Ubuntu/Debian. If you are using supported versions of these two OSes (for Ubuntu the current minimum is 18.04), then you can follow [https://apt.llvm.org/](https://apt.llvm.org/) for installation instructions. If you use OpenMP library, you need to additionally install the libomp package.

If you use relatively old distributions (like me, using Ubuntu 18.04), you are also recommended to upgrade your system's `libstdc++` (a component of GCC compiler) if you want to use new C++ standard library features, since Clang defaults to use system `libstdc++` implementation of standard library instead of bringing its own. If you don't know how, searching for *upgrade gcc linux* may be of some help.

## Use with CMake
CMake build system also supports using Clang as CUDA compiler. Set the following variables (change `/usr/bin/clang-15` and `/usr/bin/clang++-15` to the actual path of your compiler):
```
-DCMAKE_CUDA_COMPILER="/usr/bin/clang++-15" -DCMAKE_CXX_COMPILER="/usr/bin/clang++-15" -DCMAKE_C_COMPILER="/usr/bin/clang-15"
```
in your CMake configuration. If your installation is normal, CMake would automatically detect the compiler's identity and use proper flags for device code generation, etc.

## Some possible problems and workarounds
Some CUDA runtime headers may cause errors in compilation. That is result of the differences between compilers, and also compiler bugs. I'll describe some below.

One is that the compiler failed to find a special overload of the memory deallocation function. Adding `-fsized-deallocation` will enable this overload in Clang (disabled by default).

Another lies in the Thrust library. In one segment of code it checks predefined macros for selecting code according to the compiler and its version. However, it does not behave correctly under Clang. A workaround is to add `-DCUB_USE_COOPERATIVE_GROUPS` flag, which forces the selection into correct branch.

[1]: https://www.llvm.org/
[2]: https://releases.llvm.org/15.0.0/docs/CompileCudaWithLLVM.html