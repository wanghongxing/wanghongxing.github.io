---
title: 20240204-apple silicon install stable diffusion webui速度慢的解决办法
---


apple silicon install stable diffusion webui 后运行速度特别慢，参照 https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/7453

下载新的webui-user.sh文件，其实就是增加了三行

```
export COMMANDLINE_ARGS="--skip-torch-cuda-test --upcast-sampling --opt-sub-quad-attention --use-cpu interrogate"
venv_dir="venv-torch-nightly"
export TORCH_COMMAND="pip install --pre torch torchvision -f https://download.pytorch.org/whl/nightly/cpu/torch_nightly.html"

```

然后重新执行webui.sh 后系统速度暴增。


