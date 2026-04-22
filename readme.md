# logs

## Apr 22, 2026

### 哔哩哔哩直播间电池计数器

**写在前面：该项目受服务器性能限制无法顺利运行。**

本项目源于我的另一个仓库的"哔哩哔哩直播间弹幕互动盲盒查询工具" ([boxlive](https://github.com/MrOxOrOxen/yqz/tree/boxlive))。其能够自动统计直播间用户的电池投喂情况（礼物、盲盒、SC、大航海），并在网页端实时展示排名、用户名、总投喂电池数量及礼物详细信息。

**共包含以下文件：**

- boxlive_v2.py: 监听直播间电池投喂情况并生成json文件
- api_server.py: 后端接口程序，读取json文件并将结果通过HTTP接口暴露给前端
- index.html: 前端程序，HTML5+CSS3+Javascript
- gift_ledger.json: 存储所有用户电池汇总信息的动态数据库
- user_stats.json: 由boxlive_v2.py生成的另一个数据库，与本项目无关

**本项目前端部分完全由Gemini编写，后端部分介绍如下：**

后端部分通过uvicorn接受HTTP请求并将python产生的json数据传递出去。

在html代码中，采用以下方式处理已传递的数据：

```html
const response = await fetch(API_URL);
const data = await response.json();
```

此后，通过Javascript到HTML页面的动态渲染，实现完整的数据传输。

（信息的传输过程我不太懂，以上来自Gemini总结）

**本项目在阿里云服务器上运行，以下为服务器相关细节：**

需开放服务器入方向的8000, 80(HTTP)以及443(HTTPS)的端口权限，其中port 8000为自定义通讯接口。

为保证程序在服务器上保持活跃，服务器端需通过nohup后台运行：

```
nohup python3 -u boxlive_v2.py > box.log 2>&1 &
nohup python3 -u api_server.py > api.log 2>&1 &
```

日志会被存放在box.log与api.log中。-u指令用来防止日志缓冲，保证数据的及时输出。

**由于index.html通过github.io直接生成的网页为HTTP协议，无法顺利在HTTPS协议下显示，因此需要向服务器中导入网站证书。以下为具体流程（需提前在域名网站控制台关联服务器信息）：**

先安装Nginx:

```
sudo apt install nginx
```

前往域名注册网址下载证书文件(.crt和.key)。将证书文件上传至/etc/nginx/ssl/.

配置Nginx使其支持HTTPS:

```
server {
    listen 443 ssl;
    server_name yqz.mroxoroxen.com;

    ssl_certificate /etc/nginx/ssl/yqz.mroxoroxen.com_bundle.crt; 
    ssl_certificate_key /etc/nginx/ssl/yqz.mroxoroxen.com.key; 

    ssl_session_timeout 5m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        add_header 'Access-Control-Allow-Origin' '*';
        
        root /root/bili;
        index index.html;

        try_files $uri $uri/ =404;
    }

    location /data {
        proxy_pass http://127.0.0.1:8000/data;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name yqz.mroxoroxen.com;
    return 301 https://$host$request_uri;
}
```

其中，location /指令是控制ui界面的，location /data指令用来访问json数据。也就是说：

以下网址可以访问到ui界面：

https://yqz.mroxoroxen.com

以下网址可以访问到底层的json文件：

https://yqz.mroxoroxen.com/data

其中，/data为人为规定好的python, Nginx与HTML的通讯词。


激活配置：

```
nginx -t
systemctl restart nginx
```

**有关证书自动更新：**

由于国内SSL证书手动申请只有90天的有效期，需部署Certbot以实现证书自动更新。安装Certbot:

```
sudo apt install snapd -y
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

一键获取证书并配置Nginx:

```
sudo certbot --nginx -d yqz.mroxoroxen.com
```

测试自动续期：

```
sudo certbot renew --dry-run
```

请注意，certbot要求固定的证书路径，即Nginx的配置文件中ssl_certificate的路径需要修改：

```
ssl_certificate /etc/letsencrypt/live/yqz.mroxoroxen.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/yqz.mroxoroxen.com/privkey.pem;
```