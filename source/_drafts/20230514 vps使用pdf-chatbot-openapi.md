 # 20230514vps使用pdf-chatbot-openai







#### Pinecone

- Visit [Pinecone](https://pinecone.io/) to create and retrieve   API keys, and also retrieve your environment and index name from the dashboard.

https://app.pinecone.io/

使用google账号登录，创建api key

| env                      | value                                |
| ------------------------ | ------------------------------------ |
| asia-southeast1-gcp-free | 0ea02c8f-76cb-4354-a9ce-1581191c0dba |



#### MongoDB

- Visit [MongoDB](https://mongodb.com/), create a database (free one is enough for these purposes) and retrieve your URI in Connect -> Drivers

create mongodb m1 

https://cloud.mongodb.com/

使用google账号登录，创建m1实例，用户名密码登录，登陆账户信息：

username: wanghongxing

pwd: tftRqhgEUcljfHcP

mongodb url：

```
MONGODB_URI=mongodb+srv://wanghongxing:tftRqhgEUcljfHcP@cluster0.fkmmpud.mongodb.net/?retryWrites=true&w=majority

```



#### JWT

- To generate your JWT_SECRET, you can run `openssl rand -base64 32` in terminal

```
JWT_SECRET=QuOVtWS0j6Xp+uGMHD7HkkmH51mxR4VKrsWXH+lsdDM=
```



#### Google OAuth (Client ID and secret)

- Read [this guide](https://support.google.com/cloud/answer/6158849?hl=en) from Google



```
GOOGLE_CLIENT_ID=188169271168-29g68velgfait43an7esvhhodoarrpcf.apps.googleusercontent.com

GOOGLE_CLIENT_SECRET=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


```







ip: 18.222.252.155



```
ssh -i ~/.ssh/whx1.pem   ubuntu@18.222.252.155

```





```


echo -e "\n"|ssh-keygen -t rsa -N "" >/dev/null 2>&1
#copy id_rsa_pub to github
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
nvm install 19.6.0


```



```
git clone git@github.com:wanghongxing/pdf-chatbot.git
cd pdf-chatbot
npm install
touch .env



```



final .env:

```
OPENAI_API_KEY=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

PINECONE_API_KEY=0ea02c8f-76cb-4354-a9ce-1581191c0dba
PINECONE_ENVIRONMENT=asia-southeast1-gcp-free
PINECONE_INDEX_NAME=whx
MONGODB_URI=mongodb+srv://wanghongxing:tftRqhgEUcljfHcP@cluster0.fkmmpud.mongodb.net/?retryWrites=true&w=majority

GOOGLE_CLIENT_ID=188169271168-29g68velgfait43an7esvhhodoarrpcf.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=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
NEXTAUTH_URL=http://pdf-chatbot.guyuai.com
JWT_SECRET=QuOVtWS0j6Xp+uGMHD7HkkmH51mxR4VKrsWXH+lsdDM=
```

