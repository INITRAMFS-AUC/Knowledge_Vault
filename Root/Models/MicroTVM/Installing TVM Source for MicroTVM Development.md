---
tags:
  - csce/machine_learning
References:
  - https://tvm.apache.org/docs/install/from_source.html#python-package-installation
---
This is more or less steps to setup MicroTVM

# MiniConda Setup

1. Installing MiniConda3 from Conda's website
2. use `bash Miniconda3-latest-linux-x86_64.sh`
	1. Follow the prompts


# MicroTVM setup

This setups the build environment for microTVM.
```bash
# create the conda environment with build dependency
conda create -n tvm-build-venv -c conda-forge \
    "llvmdev>=15" \
    "cmake>=3.24" \
    git \
    python=3.11
# enter the build environment
conda activate tvm-build-venv
```

Clone the microTVM repo from source:
```
git clone --recursive  https://github.com/apache/tvm tvm
```

Enter the repo and make a build directory
Copy the ``config.cmake`` into the build directory, this is a template and we will modify it later.
```bash
cd tvm
rm -rf build && mkdir build && cd build
# Specify the build configuration via CMake options
cp ../cmake/config.cmake .
```
## Build Configurations for MicroTVM

>[!important] 
>These build configs are mostly based on (this repo)[https://github.com/guberti/tvm-arduino-demos] of one of the maintainers on TVM. 
>> The repo is mainly made for using microTVM on arduino, so it is a good start for trying bare metal microTVM.

We need to make sure the following setup is done, in `build/config.cmake`
```cmake
set(BUILD_STATIC_RUNTIME OFF)
set(USE_SORT ON)
set(USE_MICRO ON)
set(USE_LLVM ON)
set(CMAKE_CXX_COMPILER g++)
set(CMAKE_CXX_FLAGS -Werror)
set(HIDE_PRIVATE_SYMBOLS ON)
```

From Installation guide we also needs to run these commands
```bash
# controls default compilation flags (Candidates: Release, Debug, RelWithDebInfo)
echo "set(CMAKE_BUILD_TYPE RelWithDebInfo)" >> config.cmake

# LLVM is a must dependency for compiler end
echo "set(USE_LLVM \"llvm-config --ignore-libllvm --link-static\")" >> config.cmake
echo "set(HIDE_PRIVATE_SYMBOLS ON)" >> config.cmake
```
