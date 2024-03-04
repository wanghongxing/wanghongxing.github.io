---
title: 20240205-Stable diffusion webui 使用sdXL_v10VAEFix.safetensors 模型时候提示  Downloading VAEApprox model TimeoutError
---

# Stable diffusion webui 使用sdXL_v10VAEFix.safetensors 模型时候提示  Downloading VAEApprox model TimeoutError

使用sdXL_v10VAEFix.safetensors 模型时候提示  Downloading VAEApprox model to:  stable-diffusion-webui/models/VAE-approx/vaeapprox-sdxl.pt，大致是下载vaeapprox-sdxl.pt 文件失败。

参考： https://www.stablediffusion-cn.com/sd/sd-knowledge/1339.html

究其原因是自动下载失败导致的，因为国内需要翻墙才能下载，但是启动 sd 的时候不能翻墙，导致下载失败。

该文件在 [Release v1.0.0-pre · AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui/releases/tag/v1.0.0-pre) ，自行下载并保存到models/VAE-approx目录下就行了