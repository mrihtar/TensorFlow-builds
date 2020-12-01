# TensorFlow builds

This is a collection of my custom TensorFlow builds (wheels) for Windows and Linux
with build notes.

## Available wheels

|OS|[TensorFlow]|[Python]|[CUDA]|[cuDNN]|[Compute Capability]|Optimization|Link|
|:---|:---|:---|:---|:---|:---|:---|:---|
|Windows|2.3.1|3.8 amd64|10.1|7.6.5|3.5, 7.0|/O2 (no AVX)|[tensorflow-2.3.1+C3.5-cp38-cp38-win_amd64.whl]|
|Windows|2.3.1|3.8 amd64|10.1|7.6.5|6.1, 7.5|/O2 (no AVX)|[tensorflow-2.3.1+C6.1-cp38-cp38-win_amd64.whl]|

<sub>3.5 (Kepler): GTX Titan, GT 640-GT 920</sub><br>
<sub>7.0 (Volta): TITAN V</sub><br>
<sub>6.1 (Pascal): GTX 10x0, MX110-MX350</sub><br>
<sub>7.5 (Turing): RTX 20x0, GTX 16xx, MX450</sub>

## Build on Windows

### Configuration

After setting up [msys2], [bazel v3.1.0], [MS Visual C++ Build Tools 2019] and
[the whole environment], the build can be done with this config file (`.tf_configure.bazelrc`):

```
build --action_env PYTHON_BIN_PATH="C:\\Python38\\python.exe"
build --action_env PYTHON_LIB_PATH="C:\\Python38\\lib\\site-packages"
build --python_path="C:\\Python38\\python.exe"
build --action_env TF_CUDA_VERSION="10"
build --action_env TF_CUDNN_VERSION="7"
build --action_env TF_CUDA_PATHS="C:\\Program Files\\NVIDIA GPU Computing Toolkit\\CUDA\\v10.1"
build --action_env CUDA_TOOLKIT_PATH="C:\\Program Files\\NVIDIA GPU Computing Toolkit\\CUDA\\v10.1"
build --action_env TF_CUDA_COMPUTE_CAPABILITIES="6.1,7.5"
build --action_env TF_CONFIGURE_IOS="0"
build --config=xla
build --config=cuda
build:opt --copt=/O2
build:opt --define with_default_optimizations=true
build --define=override_eigen_strong_inline=true
test --flaky_test_attempts=3
test --test_size_filters=small,medium
test:v1 --test_tag_filters=-benchmark-test,-no_oss,-no_windows,-no_windows_gpu,-no_gpu,-oss_serial
test:v1 --build_tag_filters=-benchmark-test,-no_oss,-no_windows,-no_windows_gpu,-no_gpu
test:v2 --test_tag_filters=-benchmark-test,-no_oss,-no_windows,-no_windows_gpu,-no_gpu,-oss_serial,-v1only
test:v2 --build_tag_filters=-benchmark-test,-no_oss,-no_windows,-no_windows_gpu,-no_gpu,-v1only
```
### Actual build

Machine with fast multithreaded multicore processor(s) is recommended for this task.

```
$ bazel clean --expunge
...
INFO: Found applicable config definition build:monolithic in file c:\...\tensorflow\.bazelrc: --define framework_shared_object=false
INFO: Starting clean.

$ bazel build --config=opt --config=short_logs --define=no_tensorflow_py_deps=true //tensorflow/tools/pip_package:build_pip_package
...
Target //tensorflow/tools/pip_package:build_pip_package up-to-date:
  bazel-bin/tensorflow/tools/pip_package/build_pip_package
  bazel-bin/tensorflow/tools/pip_package/build_pip_package.exe
INFO: Elapsed time: 24686.277s, Critical Path: 2661.70s
INFO: 10817 processes: 10817 local.
INFO: Build completed successfully, 16159 total actions
INFO: Build completed successfully, 16159 total actions

$ mkdir C:\tmp\final
$ bazel-bin\tensorflow\tools\pip_package\build_pip_package.exe C:\tmp\final
...
Tue Dec 1 15:24:15 CEST 2020 : === Output wheel file is in: C:\tmp\final
```

## Installation

- download and install [MS Visual C++ 2019 Redistributable]
- build your own wheel or download it from [here]
- install with pip  
  ```
  $ pip3 install --upgrade tensorflow-2.3.1+C6.1-cp38-cp38-win_amd64.whl
  ```

## Test

```
$ python3
Python 3.8.6 (tags/v3.8.6:db45529, Sep 23 2020, 15:52:53) [MSC v.1927 64 bit (AMD64)] on win32
...
>>> import tensorflow as tf
2020-12-01 16:27:23.146285: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library cudart64_101.dll
>>> print(tf.__version__)
2.3.1
>>> tf.constant([123]) + tf.constant([321])
2020-12-01 16:28:19.151489: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library nvcuda.dll
...
2020-12-01 16:28:20.741579: I tensorflow/compiler/xla/service/service.cc:176]   StreamExecutor device (0): GeForce GTX 1060 3GB, Compute Capability 6.1
<tf.Tensor: shape=(1,), dtype=int32, numpy=array([444])>
>>>
```

[TensorFlow]: https://github.com/tensorflow/tensorflow
[Python]: https://www.python.org/downloads/
[CUDA]: https://developer.nvidia.com/cuda-toolkit-archive
[cuDNN]: https://developer.nvidia.com/cuDNN
[Compute Capability]: https://en.wikipedia.org/wiki/CUDA#GPUs_supported
[tensorflow-2.3.1+C3.5-cp38-cp38-win_amd64.whl]: https://github.com/mrihtar/TensorFlow-builds/releases/download/v2.3.1/tensorflow-2.3.1+C3.5-cp38-cp38-win_amd64.whl
[tensorflow-2.3.1+C6.1-cp38-cp38-win_amd64.whl]: https://github.com/mrihtar/TensorFlow-builds/releases/download/v2.3.1/tensorflow-2.3.1+C6.1-cp38-cp38-win_amd64.whl
[msys2]: https://www.msys2.org
[bazel v3.1.0]: https://github.com/bazelbuild/bazel/releases/tag/3.1.0
[MS Visual C++ Build Tools 2019]: https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools
[the whole environment]: https://www.tensorflow.org/install/source_windows
[MS Visual C++ 2019 Redistributable]: https://aka.ms/vs/16/release/VC_redist.x64.exe
[here]: https://github.com/mrihtar/TensorFlow-builds/releases/tag/v2.3.1
