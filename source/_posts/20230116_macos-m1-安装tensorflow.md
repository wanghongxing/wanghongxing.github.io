---
title: macos m1 安装tensorflow
date: 2023-02-16 20:24:35
tags:
  - tensorflow
  - python
---





## 安装arm64 macos的[miniconda](https://so.csdn.net/so/search?q=miniconda&spm=1001.2101.3001.7020)

[miniconda下载](https://github.com/conda-forge/miniforge/#download)



```
 chmod +x ./Miniforge3-MacOSX-arm64.sh
 ./Miniforge3-MacOSX-arm64.sh
 source ~/miniforge3/bin/activate
 
 conda create -n tf python==3.9
 conda activate tf
  
```



## 安装tensorflow

默认安装2.8版本的tensorflow，也可以指定版本。最好默认

```
conda install -c apple tensorflow-deps
python -m pip install tensorflow-macos
python -m pip install tensorflow-metal

```

在这里看tensorflow入门，在这里就可以看文档：

https://colab.research.google.com/github/tensorflow/docs-l10n/blob/master/site/zh-cn/tutorials/keras/text_classification.ipynb?hl=zh-cn

告诉我可以用 google colab玩，就安装一下。



安装jupyter

```
conda install jupyter notebook

```

支持google的 colab



```
pip install jupyter_http_over_ws
jupyter serverextension enable --py jupyter_http_over_ws
jupyter notebook   --NotebookApp.allow_origin='https://colab.research.google.com'   --port=8888   --NotebookApp.port_retries=0

```

然后再右上角的connect种连接本地的jupyter。就可以在网页上运行python。

看看url地址貌似运行的github上的代码，我就把http://github.com/tensorflow/docs-i10n 这个代码仓库

clone了，然后就可以用 https://colab.research.google.com/github/wanghongxing/docs-l10n/blob/master/site/zh-cn/tutorials/keras/text_classification.ipynb?hl=zh-cn#scrollTo=6-tTFS04dChr 这个地址来学习tensorflow的文本分类例子。这时候是自己的代码仓库，感觉很厉害的样子。

执行之前别忘了安装 matplotlib，`pip install matplotlib`



