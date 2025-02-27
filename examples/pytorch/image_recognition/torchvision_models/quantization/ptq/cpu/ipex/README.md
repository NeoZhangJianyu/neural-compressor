Step-by-Step
============

This document describes the step-by-step instructions for reproducing PyTorch ResNet50/ResNet18/ResNet101 tuning results with Intel® Neural Compressor.

# Prerequisite

## 1. Environment

We verified examples with IPEX backend on Python 3.8, recommended.

```shell
cd examples/pytorch/image_recognition/torchvision_models/quantization/ptq/cpu/ipex
pip install -r requirements.txt
```

## 2. Install pytorch and intel-pytorch-extension

refer [intel/intel-extension-for-pytorch at icx (github.com)](https://github.com/intel/intel-extension-for-pytorch/tree/v1.8.0)

1. install PyTorch and TorchVision

   refer [PyTorch install](https://pytorch.org/get-started/locally/)
   ```shell position-relative
    pip install requirements.txt
   ```
2. Get  Intel® Extension for PyTorch* source and install
    > **Note**
    >
    > GCC9 compiler is recommended
    >

   ```shell position-relative
   python -m pip install intel_extension_for_pytorch -f https://software.intel.com/ipex-whl-stable
   ```


   ```
   # build from source for IPEX 1.12
    git clone https://github.com/intel/intel-extension-for-pytorch/
    cd intel-extension-for-pytorch
    git submodule sync && git submodule update --init --recursive
    git checkout 1279c5824f1bcb61cd8990f4148abcadf3f214a4
    git apply ../patch.patch
    python setup.py install
   ```
   > Note: Intel® Extension for PyTorch* has PyTorch version > requirement. Please check more detailed information via > the URL below.
   >
   > GCC9 compiler is recommended
   >
   > Support IPEX version >= 1.8.0, 1.12.0 version need build from source and apply patch.

## 3. Prepare Dataset

Download [ImageNet](http://www.image-net.org/) Raw image to dir: /path/to/imagenet.  The dir include below folder:

```bash
ls /path/to/imagenet
train  val
```

# Run

> Note: All torchvision model names can be passed as long as they are included in `torchvision.models`, below are some examples.

### 1. ResNet18 With Intel PyTorch Extension

```shell
python main.py -t -a resnet18 --ipex --pretrained /path/to/imagenet
```
or
```shell
bash run_tuning.sh --topology=resnet18 --dataset_location=/path/to/imagenet
bash run_benchmark.sh --topology=resnet18 --dataset_location=/path/to/imagenet --mode=benchmark/accuracy --int8=true/false
```

### 2. ResNet50 With Intel PyTorch Extension

```shell
python main.py -t -a resnet50 --ipex --pretrained /path/to/imagenet
```
or
```shell
bash run_tuning.sh --topology=resnet50 --dataset_location=/path/to/imagenet
bash run_benchmark.sh --topology=resnet50 --dataset_location=/path/to/imagenet --mode=benchmark/accuracy --int8=true/false
```

### 3. ResNext101_32x16d With Intel PyTorch Extension

```shell
python main.py -t -a resnext101_32x16d_wsl --hub --ipex --pretrained /path/to/imagenet
```
or
```shell
bash run_tuning.sh --topology=resnext101_32x16d_wsl --dataset_location=/path/to/imagenet
bash run_benchmark.sh --topology=resnext101_32x16d_wsl --dataset_location=/path/to/imagenet --mode=benchmark/accuracy --int8=true/false
```


# Saving and Loading Model

* Saving model:
  After tuning with Neural Compressor, we can get neural_compressor.model:

```
from neural_compressor import PostTrainingQuantConfig
from neural_compressor import quantization
conf = PostTrainingQuantConfig(backend='ipex')
q_model = quantization.fit(model,
                            conf,
                            calib_dataloader=val_loader,
                            eval_func=eval_func)
```

Here, q_model is Neural Compressor model class, so it has "save" API:

```python
q_model.save("Path_to_save_configure_file")
```

* Loading model:

```python
from neural_compressor.utils.pytorch import load
quantized_model = load(os.path.abspath(os.path.expanduser(args.tuned_checkpoint)),
                        model,
                        dataloader=val_loader)
```

Please refer to [Sample code](./main.py).
