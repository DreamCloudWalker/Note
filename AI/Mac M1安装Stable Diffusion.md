

参考：https://zhuanlan.zhihu.com/p/620720236 （stable diffusion webui 更换 Python 版本）

如果使用一段时间后下次打开出现类似如下的错误，可以把stable-diffusion-webui 的目录下，找到 `__pycache__` 文件夹，将其删除。

以及删除 `venv` 文件夹里所有的东西。再重新./webui.sh  运行。

```shell
NK2M63R7VN:stable-diffusion-webui jian.deng$ ./webui.sh 

################################################################
Install script for stable-diffusion + Web UI
Tested on Debian 11 (Bullseye)
################################################################

################################################################
Running on jian.deng user
################################################################

################################################################
Repo already cloned, using it as install directory
################################################################

################################################################
Create and activate python venv
################################################################

################################################################
Launching launch.py...
################################################################
Python 3.10.13 (main, Aug 24 2023, 22:36:46) [Clang 14.0.3 (clang-1403.0.22.14.1)]
Version: v1.4.0
Commit hash: 394ffa7b0a7fff3ec484bcd084e673a8b301ccc8
Installing requirements


Checking roop requirements
Install insightface==0.7.3
Installing sd-webui-roop requirement: insightface==0.7.3
Install onnx==1.14.0
Installing sd-webui-roop requirement: onnx==1.14.0
Install onnxruntime==1.15.0
Installing sd-webui-roop requirement: onnxruntime==1.15.0
Install opencv-python==4.7.0.72
Installing sd-webui-roop requirement: opencv-python==4.7.0.72

Launching Web UI with arguments: --skip-torch-cuda-test --upcast-sampling --no-half-vae --use-cpu interrogate
Traceback (most recent call last):
  File "/Users/jian.deng/Github/AI/stable-diffusion-webui/launch.py", line 38, in <module>
    main()
  File "/Users/jian.deng/Github/AI/stable-diffusion-webui/launch.py", line 34, in main
    start()
  File "/Users/jian.deng/Github/AI/stable-diffusion-webui/modules/launch_utils.py", line 340, in start
    import webui
  File "/Users/jian.deng/Github/AI/stable-diffusion-webui/webui.py", line 28, in <module>
    import pytorch_lightning   # noqa: F401 # pytorch_lightning should be imported after torch, but it re-enables warnings on import so import once to disable them
  File "/opt/homebrew/lib/python3.10/site-packages/pytorch_lightning/__init__.py", line 35, in <module>
    from pytorch_lightning.callbacks import Callback  # noqa: E402
  File "/opt/homebrew/lib/python3.10/site-packages/pytorch_lightning/callbacks/__init__.py", line 14, in <module>
    from pytorch_lightning.callbacks.batch_size_finder import BatchSizeFinder
  File "/opt/homebrew/lib/python3.10/site-packages/pytorch_lightning/callbacks/batch_size_finder.py", line 24, in <module>
    from pytorch_lightning.callbacks.callback import Callback
  File "/opt/homebrew/lib/python3.10/site-packages/pytorch_lightning/callbacks/callback.py", line 25, in <module>
    from pytorch_lightning.utilities.types import STEP_OUTPUT
  File "/opt/homebrew/lib/python3.10/site-packages/pytorch_lightning/utilities/types.py", line 27, in <module>
    from torchmetrics import Metric
  File "/opt/homebrew/lib/python3.10/site-packages/torchmetrics/__init__.py", line 14, in <module>
    from torchmetrics import functional  # noqa: E402
  File "/opt/homebrew/lib/python3.10/site-packages/torchmetrics/functional/__init__.py", line 14, in <module>
    from torchmetrics.functional.audio._deprecated import _permutation_invariant_training as permutation_invariant_training
  File "/opt/homebrew/lib/python3.10/site-packages/torchmetrics/functional/audio/__init__.py", line 14, in <module>
    from torchmetrics.functional.audio.pit import permutation_invariant_training, pit_permutate
  File "/opt/homebrew/lib/python3.10/site-packages/torchmetrics/functional/audio/pit.py", line 22, in <module>
    from torchmetrics.utilities import rank_zero_warn
  File "/opt/homebrew/lib/python3.10/site-packages/torchmetrics/utilities/__init__.py", line 14, in <module>
    from torchmetrics.utilities.checks import check_forward_full_state_property
  File "/opt/homebrew/lib/python3.10/site-packages/torchmetrics/utilities/checks.py", line 25, in <module>
    from torchmetrics.metric import Metric
  File "/opt/homebrew/lib/python3.10/site-packages/torchmetrics/metric.py", line 30, in <module>
    from torchmetrics.utilities.data import (
  File "/opt/homebrew/lib/python3.10/site-packages/torchmetrics/utilities/data.py", line 22, in <module>
    from torchmetrics.utilities.imports import _TORCH_GREATER_EQUAL_1_12, _XLA_AVAILABLE
  File "/opt/homebrew/lib/python3.10/site-packages/torchmetrics/utilities/imports.py", line 50, in <module>
    _TORCHAUDIO_GREATER_EQUAL_0_10: Optional[bool] = compare_version("torchaudio", operator.ge, "0.10.0")
  File "/opt/homebrew/lib/python3.10/site-packages/lightning_utilities/core/imports.py", line 73, in compare_version
    pkg = importlib.import_module(package)
  File "/opt/homebrew/Cellar/python@3.10/3.10.13/Frameworks/Python.framework/Versions/3.10/lib/python3.10/importlib/__init__.py", line 126, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
  File "/opt/homebrew/lib/python3.10/site-packages/torchaudio/__init__.py", line 1, in <module>
    from torchaudio import (  # noqa: F401
  File "/opt/homebrew/lib/python3.10/site-packages/torchaudio/_extension.py", line 135, in <module>
    _init_extension()
  File "/opt/homebrew/lib/python3.10/site-packages/torchaudio/_extension.py", line 105, in _init_extension
    _load_lib("libtorchaudio")
  File "/opt/homebrew/lib/python3.10/site-packages/torchaudio/_extension.py", line 52, in _load_lib
    torch.ops.load_library(path)
  File "/opt/homebrew/lib/python3.10/site-packages/torch/_ops.py", line 643, in load_library
    ctypes.CDLL(path)
  File "/opt/homebrew/Cellar/python@3.10/3.10.13/Frameworks/Python.framework/Versions/3.10/lib/python3.10/ctypes/__init__.py", line 374, in __init__
    self._handle = _dlopen(self._name, mode)
OSError: dlopen(/opt/homebrew/lib/python3.10/site-packages/torchaudio/lib/libtorchaudio.so, 0x0006): Symbol not found: __ZN2at4_ops9_pad_enum4callERKNS_6TensorEN3c108ArrayRefIxEExNS5_8optionalIdEE
  Referenced from: <BE6EA463-573E-3D7C-9087-E19500344FCD> /opt/homebrew/lib/python3.10/site-packages/torchaudio/lib/libtorchaudio.so
  Expected in:     <59ED1CF5-3CD8-3592-A70B-3AB98E4C5F21> /opt/homebrew/lib/python3.10/site-packages/torch/lib/libtorch_cpu.dylib
```

