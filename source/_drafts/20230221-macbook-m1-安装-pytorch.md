---
title: 20230221 macbook m1 安装 pytorch
tags:
  - pytorch
---

下载 https://www.anaconda.com/products/distribution#Downloads



```bash
conda info

     active environment : base
    active env location : /Users/whx/anaconda3
            shell level : 1
       user config file : /Users/whx/.condarc
 populated config files :
          conda version : 23.1.0
    conda-build version : 3.23.3
         python version : 3.10.9.final.0
       virtual packages : __archspec=1=arm64
                          __osx=13.2.1=0
                          __unix=0=0
                       
                       
>> import platform
>> print(platform.platform())
macOS-13.2.1-arm64-arm-64bit                          


conda create -n pytorch python=3.10

pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple




```

去[PyTorch官网](https://link.zhihu.com/?target=https%3A//pytorch.org/get-started/locally/)获取命令。这里注意要选取**Nightly**版本，才支持GPU加速，Package选项中选择**Pip**。(这里若使用conda安装有一定概率无法安装到预览版，建议使用pip3安装)



https://pytorch.org/get-started/locally/



```
pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cpu


python
import torch
torch.__version__
torch.device("mps")

pip install transformers



```



