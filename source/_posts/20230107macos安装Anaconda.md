---
title: macos环境安装Anaconda，然后安装python3.7.10，使用 paddlepaddle-2.3.2
date: 2023-01-07 20:37:30
tags: 
---



以前使用pyenv来管理python版本，最近看到别人用 anaconda，就了解了一下，试着安装。

### 安装anaconda

从 anaconda 官网 https://www.anaconda.com/products/distribution#macos 下载可视化版本https://repo.anaconda.com/archive/Anaconda3-2022.10-MacOSX-x86_64.pkg，然后安装；安装完后系统多了一个 anaconda-navigator，安装目录是~/opt/anaconda3 ，打开后可以在environments下create新的环境，但是新建的时候python版本不能随便选，由于一直使用python-3.7.10来使用paddle，所以还是得自己手动创建。

```bash
 
conda create -n paddle232-py3710 python=3.7.10

#
# To activate this environment, use
#
#     $ conda activate paddle232-py3710
#
# To deactivate an active environment, use
#
#     $ conda deactivate

```

创建好后在可视化界面也可以看到。



### 然后命令行激活

```
(base) wanghongxing:~ whx$ python -V
Python 3.9.13
(base) wanghongxing:~ whx$ conda activate paddle232-py3710

(paddle232-py3710) wanghongxing:~ whx$ python -V
Python 3.7.10
```

可以看到，刚开始python是3.9.13，执行 `conda activate paddle232-py3710`后python版本变成3.7.10



### 安装paddle2.3.2

```bash
(paddle232-py3710) wanghongxing:~ python -m pip install paddlepaddle==2.3.2 -i https://pypi.tuna.tsinghua.edu.cn/simple

Looking in indexes: https://pypi.tuna.tsinghua.edu.cn/simple
Collecting paddlepaddle==2.3.2
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/ba/bf/bc6a1dd9a1126d8cd1467917d34bf12c9282152f99afc5b8cad29118eda4/paddlepaddle-2.3.2-cp37-cp37m-macosx_10_6_intel.whl (93.0 MB)
Collecting numpy>=1.13
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/32/dd/43d8b2b2ebf424f6555271a4c9f5b50dc3cc0aafa66c72b4d36863f71358/numpy-1.21.6-cp37-cp37m-macosx_10_9_x86_64.whl (16.9 MB)
Collecting decorator
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/d5/50/83c593b07763e1161326b3b8c6686f0f4b0f24d5526546bee538c89837d6/decorator-5.1.1-py3-none-any.whl (9.1 kB)
Collecting paddle-bfloat==0.1.7
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/c6/40/65edb5bf459317bdc808f884805784470c9b5ab38b81df23fac02e02f5b8/paddle_bfloat-0.1.7-cp37-cp37m-macosx_10_9_x86_64.whl (44 kB)
Collecting six
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/d9/5a/e7c31adbe875f2abbb91bd84cf2dc52d792b5a01506781dbcf25c91daf11/six-1.16.0-py2.py3-none-any.whl (11 kB)
Collecting protobuf<=3.20.0,>=3.1.0
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/be/f0/2633123b475c9ae6e9be25351c7ba6ca3adc223d73789ca2f6f5e4686723/protobuf-3.20.0-cp37-cp37m-macosx_10_9_x86_64.whl (961 kB)
Collecting astor
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/c3/88/97eef84f48fa04fbd6750e62dcceafba6c63c81b7ac1420856c8dcc0a3f9/astor-0.8.1-py2.py3-none-any.whl (27 kB)
Collecting Pillow
  Downloading https://pypi.tuna.tsinghua.edu.cn/packages/91/1d/57a09a69508a27c1c6caa4197ce7fac5be5b7d736889ba1a20931ff4efca/Pillow-9.4.0-1-cp37-cp37m-macosx_10_10_x86_64.whl (3.3 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 3.3/3.3 MB 14.4 MB/s eta 0:00:00
Collecting opt-einsum==3.3.0
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/bc/19/404708a7e54ad2798907210462fd950c3442ea51acc8790f3da48d2bee8b/opt_einsum-3.3.0-py3-none-any.whl (65 kB)
Collecting requests>=2.20.0
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/ca/91/6d9b8ccacd0412c08820f72cebaa4f0c0441b5cda699c90f618b6f8a1b42/requests-2.28.1-py3-none-any.whl (62 kB)
Collecting idna<4,>=2.5
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/fc/34/3030de6f1370931b9dbb4dad48f6ab1015ab1d32447850b9fc94e60097be/idna-3.4-py3-none-any.whl (61 kB)
Collecting urllib3<1.27,>=1.21.1
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/65/0c/cc6644eaa594585e5875f46f3c83ee8762b647b51fc5b0fb253a242df2dc/urllib3-1.26.13-py2.py3-none-any.whl (140 kB)
Collecting charset-normalizer<3,>=2
  Using cached https://pypi.tuna.tsinghua.edu.cn/packages/db/51/a507c856293ab05cdc1db77ff4bc1268ddd39f29e7dc4919aa497f0adbec/charset_normalizer-2.1.1-py3-none-any.whl (39 kB)
Requirement already satisfied: certifi>=2017.4.17 in ./opt/anaconda3/envs/paddle232-py3710/lib/python3.7/site-packages (from requests>=2.20.0->paddlepaddle==2.3.2) (2022.12.7)
Installing collected packages: paddle-bfloat, urllib3, six, protobuf, Pillow, numpy, idna, decorator, charset-normalizer, astor, requests, opt-einsum, paddlepaddle
Successfully installed Pillow-9.4.0 astor-0.8.1 charset-normalizer-2.1.1 decorator-5.1.1 idna-3.4 numpy-1.21.6 opt-einsum-3.3.0 paddle-bfloat-0.1.7 paddlepaddle-2.3.2 protobuf-3.20.0 requests-2.28.1 six-1.16.0 urllib3-1.26.13

(paddle232-py3710) wanghongxing:~ whx$ which paddle
/Users/whx/opt/anaconda3/envs/paddle232-py3710/bin/paddle
(paddle232-py3710) wanghongxing:~ whx$ paddle version
<stdin>:3: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
PaddlePaddle 2.3.2, compiled with
    with_avx: ON
    with_gpu: OFF
    with_mkl: OFF
    with_mkldnn: OFF
    with_python: ON


```

安装paddle

```bash

(paddle232-py3710) wanghongxing:guyuai-emotion-train whx$ pip3 install paddle
Collecting paddle
  Using cached paddle-1.0.2.tar.gz (579 kB)
  Preparing metadata (setup.py) ... error
  error: subprocess-exited-with-error
  
  × python setup.py egg_info did not run successfully.
  │ exit code: 1
  ╰─> [8 lines of output]
      Traceback (most recent call last):
        File "<string>", line 36, in <module>
        File "<pip-setuptools-caller>", line 34, in <module>
        File "/private/var/folders/6d/lxb9s6557cx2wq_2fs6s2_bm0000gr/T/pip-install-h1m6_55m/paddle_d36d8dad2117474f8fc2aaa140c6ac30/setup.py", line 3, in <module>
          import paddle
        File "/private/var/folders/6d/lxb9s6557cx2wq_2fs6s2_bm0000gr/T/pip-install-h1m6_55m/paddle_d36d8dad2117474f8fc2aaa140c6ac30/paddle/__init__.py", line 5, in <module>
          import common, dual, tight, data, prox
      ModuleNotFoundError: No module named 'common'
      [end of output]
  
  note: This error originates from a subprocess, and is likely not a problem with pip.
error: metadata-generation-failed

× Encountered error while generating package metadata.
╰─> See above for output.

note: This is an issue with the package mentioned above, not pip.
hint: See above for details.

```

然后手动安装依赖包

```bash
pip install common
pip install dual
pip install tight
pip install data
pip install prox

(paddle232-py3710) wanghongxing:guyuai-emotion-train whx$ pip3 install paddle==1.0.2
Collecting paddle==1.0.2
  Using cached paddle-1.0.2.tar.gz (579 kB)
  Preparing metadata (setup.py) ... done
Building wheels for collected packages: paddle
  Building wheel for paddle (setup.py) ... done
  Created wheel for paddle: filename=paddle-1.0.2-py3-none-any.whl size=33366 sha256=63916ce0eea092ef9e690f3e992b51f32f955eabc66070be6e9feba60d672973
  Stored in directory: /Users/whx/Library/Caches/pip/wheels/e2/38/0e/382f68d54c6949b370f1438aa96172ff44a8ed367134cce32e
Successfully built paddle
Installing collected packages: paddle
Successfully installed paddle-1.0.2


```

安装完后仍然执行异常，没办法，删除`rm -rf ~/opt/anaconda3`，重新下载命令行版本，下载地址`https://repo.anaconda.com/archive/Anaconda3-2022.10-MacOSX-x86_64.sh`。



```
bash Anaconda3-2022.10-MacOSX-x86_64.sh
conda create -n paddle232-py3710 python=3.7.10
conda activate paddle232-py3710
pip install paddle==1.0.2
python -m pip install paddlepaddle==2.3.2 -i https://pypi.tuna.tsinghua.edu.cn/simple

```

这时候 paddle2.3.2 环境正常。

```
arch -x86_64 bash

```



