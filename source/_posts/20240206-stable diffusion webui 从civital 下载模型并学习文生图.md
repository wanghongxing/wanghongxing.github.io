---
title: 20240206-stable diffusion webui 从civital 下载模型并学习文生图
---

civital 下载模型并学习之二

# Meow Mix - Realistic & Versatile

 https://civitai-delivery-worker-prod.5ac0637cfd0766c97916cefa3764fbdf.r2.cloudflarestorage.com/158704/model/meowPrunedFp16.Jl2h.safetensors?X-Amz-Expires=86400&response-content-disposition=attachment%3B%20filename%3D%22meowMixRealistic_prunedFp16FIXED.safetensors%22&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=e01358d793ad6966166af8b3064953ad/20240205/us-east-1/s3/aws4_request&X-Amz-Date=20240205T111539Z&X-Amz-SignedHeaders=host&X-Amz-Signature=0a9b38d02cd05995a45f62c9bc0bcbb7d3313b7566b71e2bcb4180c0d6c91097



Prompt：

`Jim is a middle-aged man with short, graying hair and a rugged appearance. He has a rough, unshaven chin and deep wrinkles around his eyes. He is tall and muscular, with broad shoulders and thick arms. He wears a dirty, faded t-shirt and faded jeans that hang loosely on his frame. His demeanor is angry and bitter, with a constant scowl on his face. He is a character who has been through a lot of hardship and is struggling to come to terms with the changes in society. Despite his rough exterior, there is a hint of sadness in his eyes, indicating that he is deeply hurt by the loss of his job and his wife. Overall, Jim is a complex and troubled character who is struggling to find his place in a world that is rapidly changing around him.`

Negative Prompt:

`canvas frame, cartoon, 3d, ((disfigured)), ((bad art)), ((deformed)),((extra limbs)),((close up)),((b&w)), wierd colors, blurry, (((duplicate))), ((morbid)), ((mutilated)), [out of frame], extra fingers, mutated hands, ((poorly drawn hands)), ((poorly drawn face)), (((mutation))), (((deformed))), ((ugly)), blurry, ((bad anatomy)), (((bad proportions))), ((extra limbs)), cloned face, (((disfigured))), out of frame, ugly, extra limbs, (bad anatomy), gross proportions, (malformed limbs), ((missing arms)), ((missing legs)), (((extra arms))), (((extra legs))), mutated hands, (fused fingers), (too many fingers), (((long neck))), Photoshop, video game, ugly, tiling, poorly drawn hands, poorly drawn feet, poorly drawn face, out of frame, mutation, mutated, extra limbs, extra legs, extra arms, disfigured, deformed, cross-eye, body out of frame, blurry, bad art, bad anatomy, 3d render`



![00134-618633303](../images/sd/00134-618633303.png)

*Steps: 20, Sampler: Euler a, CFG scale: 8, Seed: 618633303, Size: 512x512, Model hash: 834d9e407e, Model: meowMixRealistic_prunedFp16FIXED, Version: v1.7.0*



试着自己改

Prompt：

`John is a middle-aged chinese man with graying short hair and a rugged appearance. He has a rough, unshaven chin and deep wrinkles around his eyes. He is tall and muscular, with broad shoulders and thick arms. He wears a dirty, faded t-shirt and faded jeans that hang loosely on his frame. His demeanor is angry and bitter, with a constant scowl on his face. He is a character who has been through a lot of hardship and is struggling to come to terms with the changes in society. Despite his rough exterior, there is a hint of sadness in his eyes, indicating that he is deeply hurt by the loss of his job and his wife. Overall, John is a complex and troubled character who is struggling to find his place in a world that is rapidly changing around him.`

<img src="../images/sd/20240206095935-14219338.png" alt="20240206095935-14219338" style="zoom:50%;" />