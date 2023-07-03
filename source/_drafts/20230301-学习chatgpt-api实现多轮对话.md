



看一些例子





OpenAI 开放了两个新模型的api接口，专门为聊天而生的 gpt-3.5-turbo 和 gpt-3.5-turbo-0301。



源代码：[XksA-me/ChatGPT-3.5-AP](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FXksA-me%2FChatGPT-3.5-API)



**首先**你需要有一个 openai 账号，如何注册我就不多说了，网上教程很多，而且很详细，如果有问题可以加我微信：pythonbrief，**添加通过后请直接描述你的问题+问题截图**。

访问下面页面，登录 openai 账号后，创建一个 api keys。

```bash
# api keys 创建页面
https://platform.openai.com/account/api-keys
复制代码
```

**接下来**很简单了，安装 openai 官方的 Python SDK，这里需要注意的是得安装最新版本 openai，官方推荐的是 0.27.0 版本。

```bash
pip install openai==0.27.0
 
```

 

**直接上**请求代码：

```python
import openai
import json

# 目前需要设置代理才可以访问 api
os.environ["HTTP_PROXY"] = "自己的代理地址"
os.environ["HTTPS_PROXY"] = "自己的代理地址"


def get_api_key():
    # 可以自己根据自己实际情况实现
    # 以我为例子，我是存在一个 openai_key 文件里，json 格式
    '''
    {"api": "你的 api keys"}
    '''
    openai_key_file = '../envs/openai_key'
    with open(openai_key_file, 'r', encoding='utf-8') as f:
        openai_key = json.loads(f.read())
    return openai_key['api']

openai.api_key = get_api_key()

q = "用python实现：提示手动输入3个不同的3位数区间，输入结束后计算这3个区间的交集，并输出结果区间"
rsp = openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[
        {"role": "system", "content": "一个有10年Python开发经验的资深算法工程师"},
        {"role": "user", "content": q}
    ]
)
 
```

 

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcd3e3f3999e4db8b3a8725bd7b8f456~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

## 实现多轮对话

如何实现多轮对话？

gpt-3.5-turbo 模型调用方法 openai.ChatCompletion.create 里传入的 message 是一个列表，列表里每个元素是字典，包含了角色和内容，我们只需将每轮对话都存储起来，然后每次提问都带上之前的问题和回答即可。



 



---


这个貌似做了个是啥

https://github.com/acheong08/ChatGPT/

----



这个貌似做了一个桌面应用，包装了chatgpt网站

https://github.com/lencx/ChatGPT

下面这个

https://github.com/f/awesome-chatgpt-prompts

