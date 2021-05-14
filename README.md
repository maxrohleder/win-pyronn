# PyroNN layers as a tf custom op

This repository contains the pyronn layers configured as a tf-custom ops for windows. 
Use the install instructions to install one of the provided builds, or follow the step-by-step guide to build for your own system.

## Install with pip

- create a python env with python 3.6 (I use conda for env management here, but any python 3.6 will do)

`conda create -n "pyronn" python=3.6`

- use the supplied requirements.txt file to install pyronn along with all necessary dependencies

`pip install -r https://raw.githubusercontent.com/maxrohleder/win-pyronn/master/requirements.txt`

- the approach above guarantees, that all deps are fitting. Alternatively, you can install the major dependecies by hand:

```
- python 3.6
- matplotlib 3.3.4
- tensorflow 2.4.1
- pyronn 0.1.0
Then:
- pip install https://github.com/maxrohleder/win-pyronn/releases/download/v0.1.0/pyronn_layers-0.1.0-cp36-cp36m-win_amd64.whl
```
## A step-by-step guide

As the [custom_ops readme](https://github.com/tensorflow/custom-op) offers very limited support for building a custom
layer on windows, I decided to create my own and thoroughly document the build process of 
the [pyroNN layers](https://github.com/csyben/PYRO-NN-Layers).

I spent a lot of time on this issue, so here are the exact steps I used to compile the framework. I used `powershell` with `Msys2` in my path to support the linux commands used by the build toolchain.

Make sure to use one of the tested combinations of build tools and dependencies listed [here](https://www.tensorflow.org/install/source_windows?hl=en#gpu).

| Tf version | Python version	|  Compiler	  | Build tools |   cuDNN	|  CUDA  |
| --- | --- | --- | --- | --- | --- |
| tensorflow_gpu-2.4.0	 |  3.6-3.8	      |    MSVC 2019	 |  Bazel 3.1.0	 |  8.0	|  11.0 |
| tensorflow_gpu-2.3.0	 |  3.5-3.8	     |     MSVC 2019	 |  Bazel 3.1.0	 |  7.6	| 10.1 |

**Note**: Despite this table, I had problems with the pre-built `tensorflow 2.3` from conda, so I tried out version 2.4.1 from pip (PyPi) and it works like a charm.

1. download bazel 3.1.0, unzip it to some folder and add it to path (its just one executable)
https://github.com/bazelbuild/bazel/releases/tag/3.1.0

2. install cuda 10.1 and cudnn 7.6.5
https://developer.nvidia.com/cuda-10.1-download-archive-base
https://developer.nvidia.com/rdp/cudnn-archive  <-- (have to make an nvidia dev account [direct link](https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.1_20191031/cudnn-10.1-windows10-x64-v7.6.5.32.zip))

3. unpack cudnn into the cuda folder
https://medium.com/vitrox-publication/deep-learning-frameworks-tensorflow-build-from-source-on-windows-python-c-cpu-gpu-d3aa4d0772d8

4. install MSVC 2019 Build tools and set ENV variables
https://visualstudio.microsoft.com/downloads/
Make sure to set these two environment variables and the specified paths exist on your system:
```
BAZEL_VC                   C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\VC
BAZEL_VC_FULL_VERSION      14.26.28801
```
   
5. create a python env with suitable tensorflow
```bash
conda create -n "pyronn" python=3.6
conda activate pyronn
pip install tensorflow-gpu==2.4.1
```
Test if tensorflow can reach the gpu:
```shell
python -c "import tensorflow as tf;tf.config.list_physical_devices('GPU')"
```

6. fix a tensorflow bug (reason for this https://github.com/maxrohleder/win-pyronn/issues/4)

Locate the `Tensor` file and delete the `#include <unistd.h>` line. For my conda python installation it was located here
`C:\Users\maxrohleder\Miniconda3\envs\tftest2\Lib\site-packages\tensorflow\include\unsupported\Eigen\CXX11\Tensor`

The line is located in line 74 and should not be included on Windows as it is posix...

7. build the layers directly from this repository (already includes some fixes)
```shell
git clone --recurse-submodules https://github.com/maxrohleder/win-pyronn win-pyronn
cd win-pyronn
bash configure.sh
bazel build --enable-runfiles build_pip_pkg --verbose_failures
```

8. create the pip archive `.whl` file and install it

```shell
bazel-bin/build_pip_pkg artifacts
pip install ./artifacts/<some-generated-name>.whl
```

### Troubleshooting and Changelog

Search the [issues section](https://github.com/maxrohleder/win-pyronn/issues?q=is%3Aissue) of this repository to find more answers.

#### cant find MSVC 

Make sure to set these two environment variables and the specified paths exist on your system:
```
BAZEL_VC                   C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\VC
BAZEL_VC_FULL_VERSION      14.26.28801
```

#### python not found error

If you see an error like this:
```shell
Python was not found; run without arguments to install from the Microsoft Store, or disable this shortcut from Settings > Manage App Execution Aliases.
Python was not found; run without arguments to install from the Microsoft Store, or disable this shortcut from Settings > Manage App Execution Aliases.
```

Change lines 128 and 129 of configure.sh:

```shell
TF_CFLAGS=( $(python3 -c 'import tensorflow as tf; print(" ".join(tf.sysconfig.get_compile_flags()))') )
TF_LFLAGS="$(python3 -c 'import tensorflow as tf; print(" ".join(tf.sysconfig.get_link_flags()))')"
```

to 

```shell
TF_CFLAGS=( $(python -c "import tensorflow as tf; print(' '.join(tf.sysconfig.get_compile_flags()))") )
TF_LFLAGS="$(python -c "import tensorflow as tf; print(' '.join(tf.sysconfig.get_link_flags()))")"
```

#### Cant find `_pyronn_layers_ops.so`

The tensorflow library loader gets confused, if you run the python script from within the build folder. 
Make sure to use a new project, such that tensorflow searches the `.so` file in the site-packages of your python env.

#### This wheel is not compatible with this plattform

The released binary was built against `python 3.6` and `tensorflow 2.4.1` you need to install that via pip, as conda
does not yet provide the latest tf version. (April 2021)





















