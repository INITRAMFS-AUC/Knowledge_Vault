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

