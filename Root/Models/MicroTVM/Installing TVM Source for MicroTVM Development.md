---
tags:
  - csce/machine_learning
References:
  - https://tvm.apache.org/docs/install/from_source.html#enable-c-tests
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

>[!important] Make sure that `nproc` is on your system

>[!important] Make sure that `g++` is installed on the system with `which g++`

We need to make sure the following setup is done, in `build/config.cmake`
```cmake
set(BUILD_STATIC_RUNTIME OFF)
set(USE_SORT ON)
set(USE_MICRO ON)
set(USE_LLVM "llvm-config --ignore-libllvm --link-static")
set(HIDE_PRIVATE_SYMBOLS ON)
```

Build using the following flags:
```bash
cmake .. && cmake --build . --parallel $(nproc)
```

**Make Sure** that the `tvm-ffi` directory is within the root of the project not the one within the build directory.

```bash
cd 3rdparty/tvm-ffi; pip install .; cd ..
```

