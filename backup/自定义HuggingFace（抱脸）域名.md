# 闲聊
之前看到了有人分享了搭建始皇同款网站监控面板[uptime-kuma](https://linux.do/t/topic/31141)的文章。
我对其中huggingface分配的域名不仅难看、难记还不能自定义不好用的这个观点非常认同。所以我下面可以提供另一种在huggingface上自定义域名的方法以供大家参考。
# 正片
其实就是在部署[NB](https://github.com/Harry-zklcdc/go-proxy-bingai)这个项目的时候看到[讨论338](https://github.com/Harry-zklcdc/go-proxy-bingai/discussions/338)中作者提供了利用Cloudflare Tunnel来对HuggingFace和 CodeSandbox进行自定义域名,虽然该项目本身已经被HuggingFace 封杀掉了，但是不影响我们使用其中的Cloudflare Tunnel的穿透服务。
# 修改及配置
## CLOUDFLARE
首先进入 Zero Trust 后台
![index](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/e82fe509-5c0a-468f-9095-06b300153a04)
选择「Network」-> 「Tunnel」-> 「Create a Tunnel」
![tunnel](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/bd880c4e-5cee-4367-bec3-0e5b8fc01836)
选择「Cloudflared」
![choose](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/2690f500-927d-494a-877a-2fbd9fddf962)
填入该隧道的注释名称
![name](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/376cceaa-4d37-4397-9323-4233ff8d51b9)
点击右边方框内的内容, 复制安装命令
![token](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/d524b19c-fd10-41e0-a44d-1b6b94b6cf82)
只需要框出来的这个，将后面的`eyJhIjoiMmxxxxxxxxxxxxxxxxxxxxx`复制出来保存。
## HuggingFace
在huggingface的uptime-kuma仓库设置中添加秘密环境变量
Name为`CF_ZERO_TRUST_TOKEN`
Value为`eyJhIjoiMmxxxxxxxxxxxxxxxxxxxxx`,即之前在cloudflare中获取的那个。
![环境变量](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/0e8c966e-3cf0-429f-a2af-775c8097c932)
然后重启 Space
![restart](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/2509ec60-9758-4b1b-8937-0d863b8585d6)
然后对上面部署uptime-kuma项目的Dockerfile进行如下修改
```
FROM alpine AS builder
RUN apk add --no-cache nodejs npm git

RUN npm install npm -g

RUN adduser -D app

USER root
RUN apk --no-cache add curl supervisor
# 创建 Supervisor 日志目录并设置权限
RUN mkdir -p /var/log/supervisor/ && \
    chown -R app:app /var/log/supervisor/

# 创建 Supervisor 运行目录并设置权限
RUN mkdir -p /var/run/supervisor/ && \
    chown -R app:app /var/run/supervisor/

USER app
WORKDIR /home/app

RUN git clone https://github.com/louislam/uptime-kuma.git
WORKDIR /home/app/uptime-kuma
RUN npm run setup

USER root
WORKDIR /home/app
RUN curl -LO https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 && \
    chmod +x cloudflared-linux-amd64 && \
    mv cloudflared-linux-amd64 /usr/local/bin/cloudflared

COPY ../supervisor.conf /etc/supervisor/supervisord.conf
COPY ../start.sh /usr/bin/start.sh

WORKDIR /home/app/uptime-kuma
USER app

EXPOSE 3001
#CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf", "&&", "node", "server/server.js"]
#CMD ["node", "server/server.js"] dockerfile
#CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
CMD ["sh", "/usr/bin/start.sh"]
```
这是我自己修改的，**有些地方可能有些混乱**，但毕竟是我自己用的，只要能运行，我就懒得优化了 :stuck_out_tongue_winking_eye: 。

然后再huggingface继续创建`supervisor.conf`配置文件。该配置文件也是来自于NB项目[supervisor](https://github.com/Harry-zklcdc/go-proxy-bingai/blob/master/docker/supervisor.conf)，我只使用到了我用到的文件（大概？）
```
[supervisord]
logfile=/var/log/supervisor/supervisord.log
logfile_maxbytes=10MB
logfile_backups=10
loglevel=info
pidfile=/var/run/supervisor/supervisord.pid
nodaemon=true
childlogdir=/var/log/supervisor

[inet_http_server]
port=127.0.0.1:9005

[rpcinterface:supervisor]
supervisor.rpcinterface_factory=supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=http://127.0.0.1:9005

[program:cloudflared]
command=/usr/local/bin/cloudflared tunnel --no-autoupdate run --token %(ENV_CF_ZERO_TRUST_TOKEN)s
directory=/
stdout_logfile=/dev/stdout
stderr_logfile=/dev/stderr
autostart=true
autorestart=true
startsecs=5
stopwaitsecs=5
killasgroup=true
```
原本的Dockerfile文件中最后一行的`CMD ["node", "server/server.js"] dockerfile`（已经被注释掉放进了sh脚本）我是打算配置到`supervisor`服务中的，奈何我问了ChatGPT也没能解决没有访问权限的问题，所以就丢进sh脚本一起运行了，下面就是在huggingface中创建这个sh脚本文件，`start.sh`
```sh
#!/bin/bash

# 启动 supervisord
/usr/bin/supervisord -c /etc/supervisor/supervisord.conf &

# 启动 Node.js 应用
node server/server.js
```
这样自动部署后就应该正常运行能访问了，即使HuggingFace的仓库本身不公开即（private），也能通过cloudflare tunnel中的自定义域名来访问。
另附`README.md`文件内容，如果是安装LINUX DO的那个帖子部署的，应该是没问题的
```
---
title: Uptime Kuma
emoji: 🏢
colorFrom: indigo
colorTo: gray
sdk: docker
pinned: false
license: mit
app_port: 3001
---

Check out the configuration reference at https://huggingface.co/docs/hub/spaces-config-reference
```
## END
最后回到cloudflare tunnel界面，看到绿色的Status中显示HEALTHY
![tunnel](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/cb2ce660-f9b6-469d-9c92-d9e577371420)
就表示已经连接成功了。然后点击配置进入下面的页面
![public](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/f53e5df5-71dc-4db9-9d78-92d6ddaaade6)
然后在下一个页面中填入如图所示内容
![example](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/fe692a7a-d595-4ecb-aaa0-e3a2ea9c1463)

>`Subdomain` 是指子域名, 自定义
>
>`Domain`选择绑定到 Cloudflare 的域名, 自定义
>
>`Type`选择 `HTTP`, 不要改
>
>`URL` 填入 `localhost:3001`，即你在huggingface的README.MD中`app_port: 3001`暴露的端口
最后，你就可以用刚才填入的域名去访问了。