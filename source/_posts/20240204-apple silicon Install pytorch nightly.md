apple silicon Install pytorch nightly

https://developer.apple.com/metal/pytorch/



##### Anaconda

#### Setup

Apple silicon

```
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh
sh Miniconda3-latest-MacOSX-arm64.sh
```



#### Install

##### Anaconda

```
conda install pytorch torchvision torchaudio -c pytorch-nightly
```

##### pip

```
pip3 install --pre torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/nightly/cpu
```

#### Verify

You can verify `mps` support using a simple Python script:

```
import torch
if torch.backends.mps.is_available():
    mps_device = torch.device("mps")
    x = torch.ones(1, device=mps_device)
    print (x)
else:
    print ("MPS device not found.")
```

The output should show:

```
tensor([1.], device='mps:0')
```



