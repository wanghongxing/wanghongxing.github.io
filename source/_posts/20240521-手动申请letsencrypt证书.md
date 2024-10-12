手动申请letsencrypt证书

系统安装docker，然后在执行如下命令

```
 docker run -it --rm --name certbot \
            -v "/etc/letsencrypt:/etc/letsencrypt" \
            -v "/var/lib/www:/www" \
            certbot/certbot certonly
```

其中/var/lib/www是域名静态根目录，期间提示号域名的根目录的时候输入 /www

正常的话会将证书放到/etc/letsencrypt目录下。