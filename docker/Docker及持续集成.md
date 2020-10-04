

# Docker



## 一、初识 Docker

### 1、Docker 是什么

- Build,Ship and Run Any App,Anywhere — 一次封装，到处执行。
- 基于 Linux 的高效、敏捷、轻量级的容器(轻量虚拟)方案。

> 虚拟技术
>
> - 完全虚拟化 VMware Workstation ，VirtualBox
> - 硬件辅助虚拟化 InterVT AMD-V
> - 超虚拟化 Xen
> - 操作系统级 Docker LXC 容器

### 2、特点

- 高效的利用系统资源
- 快速的启动时间
- 一致的运行环境
- 持续交付和部署
- 更轻松的迁移

### 3、对比传统虚拟机总结

| 特性       | 容器               | 虚拟机     |
| ---------- | ------------------ | ---------- |
| 启动       | 秒级               | 分钟级     |
| 硬盘使用   | 一般为`MB`         | 一般为`GB` |
| 性能       | 接近原生           | 弱于       |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |



## 二、Docker安装

主机：ubuntu_18_04_x64



```shell
 # apt升级
sudo apt-get update

# 添加相关软件包
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
    
# 下载软件包的合法性，需要添加软件源的 GPG 密钥
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

# source.list 中添加 Docker 软件源 
sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
    
# 安装 Docker CE
sudo apt-get update
sudo apt-get install docker-ce

# 启动 Docker CE
sudo systemctl enable docker
sudo systemctl start docker

# 建立 docker 用户组(附加)
sudo groupadd docker
sudo usermod -aG docker $USER

# Helloworld测试
docker run hello-world
 
```



**镜像加速**

- [Azure中国镜像](https://dockerhub.azk8s.cn)
- [阿里云加速器(需登录账号获取)](https://cr.console.aliyun.com/)
- [七牛云加速器](https://reg-mirror.qiniu.com)

```shell
# /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com"
	] 
}

sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 三、简单Nginx服务

```shell
# 拉取官方镜像 - 面向docker的只读模板 
docker pull nginx

# 查看
docker images nginx

# 启动镜像
mkdir www
echo 'hello docker!!' >> www/index.html

# 启动
# www目录里面放一个index.html
docker run -p 8000:80 -v $PWD/www:/usr/share/nginx/html nginx

# 后台启动
docker run -p 80:80 -v $PWD/www:/usr/share/nginx/html -d nginx

# 停止
docker stop ff6

# 查看进程
docker ps
docker ps -a // 查看全部

# 伪终端 ff6容器的uuid
docker exec -it ff6 /bin/bash

# 删除镜像 
docker rm ff6
 
```



## 四、Docker运行过程



- 镜像（Image）

  面向Docker的只读模版

- 容器（Container）

  镜像的运行实例

- 仓库（Registry）

  存储镜像的服务器

  

## 五、Dockerfile定制镜像

> 镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的 命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的 问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

- 定制自己的Web服务器

```dockerfile
#Dockerfile
FROM nginx:latest
RUN echo '<h1>Hello, Kaikeba!</h1>' > /usr/share/nginx/html/index.html
```

```shell
# 定制镜像
docker build -t nginx:kaikeba .
# 运行
docker run -p 80:80 nginx:kaikeba
```



## 六、定制NodeJS镜像

- 定制一个程序NodeJS镜像

```shell
npm init -y
npm i koa -s
```

```json
 // package.json
{
  "name": "myappp",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": "myappp",
  "dependencies": {
    "koa": "^2.7.0"
  }
}
```

```javascript
 // app.js
const Koa = require('koa')
const app = new Koa()
app.use(ctx => {
    ctx.body = 'Hello Docker'
})
app.listen(3000, () => {
    console.log('app started at http://localhost:3000/')
})
```

```dockerfile
#Dockerfile

#制定node镜像的版本
FROM node:10-alpine 

#移动当前目录下面的文件到app目录下 
ADD . /app/ 

#进入到app目录下面，类似cd 
WORKDIR /app

#安装依赖
RUN npm install

#对外暴露的端口
EXPOSE 3000

#程序启动脚本
CMD ["node", "app.js"]
```

```shell
# 定制镜像
docker build -t mynode .

# 运行
docker run -p 3000:3000 -d mynode
```



## 七、定制PM2镜像

PM2-利用多核资源

- 内建负载均衡（使用Node cluster集群模块、子进程，可以参考朴灵的《深入浅出node.js》一书第九章）
- 线程守护，keep alive
- 0秒停机重载，维护升级的时候不需要停机
- 现在``Linux(stable) && Mac OSX(stable) && windows(stable)`` 多平台支持
- 停止不稳定的进程（避免无限循环）
- 控制台检测  https://id.keymetrics.io/api/oauth/login#/register
- 提供 ``HTTP API``

**配置**

```shell
npm install -g pm2
pm2 start app.js --watch -i 2 // watch 监听文件变化 -i 启动多少个实例

pm2 stop all
pm2 list
pm2 start app.js -i max # 根据机器CPU核数，开启对应数目的进程
```

**pm2配置开机启动**

```
pm2 startup
```

```javascript
# .dockerignore
node_modules
```

**定制PM2镜像**

```yml
 // process.yml
apps:
  - script : app.js
    instances: 2
    watch  : true
    env    :
      NODE_ENV: production
 
```

```dockerfile
# Dockerfile
FROM keymetrics/pm2:latest-alpine
WORKDIR /usr/src/app
ADD . /usr/src/app
RUN npm config set registry https://registry.npm.taobao.org/ && \
    npm i
EXPOSE 3000

#pm2在docker中使用命令为pm2-docker
CMD ["pm2-runtime", "start", "process.yml"]
```

```shell
 # 定制镜像
docker build -t mypm2 .

# 运行
docker run -p 3000:3000 -d mypm2
```



## 八、Compose

**简介**

Compose项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。

```shell
#安装
apt install docker-compose
```

```shell
mkdir hello-world && cd $_

vi helloword-compose.yml

# helloworld-compose.yml
version: "3.1"
services:
	hello-world:
		image: hello-world
```

```shell
docker-compose up
```

- 使用docker安装``mongo``l以及``mongo-express``

```shell
mkdir mongo && cd $_
vi docker-compose.yml

#docker-compose.yml
version: '3.1'
services:
  mongo:
    image: mongo
    restart: always
    ports:
        - 27017:27017
  mongo-express:
    image: mongo-express
    restart: always
    ports:
				- 8000:8081

# 启动
docker-compose up

#浏览器打开
http://{url}:8000/
 
```

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20200107213005.png)



## 九、构建前端应用

**文件结构**

<img src="https://raw.githubusercontent.com/super-lin0/pic/master/img/20200108075605.png" />

- 安装deploy插件（主要用于将代码上传到服务器上）

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20200108073136.png)

```json
#.vscode/settings.json
{
  "deploy": {
    "packages": [
      {
        "files": ["**/*"],

        "exclude": [
          "node_modules/**",
          ".git/**",
          ".vscode/**",
          "**/node_modules/**"
        ],
        "deployOnSave": false	// 在保存时上传
      }
    ],
    "targets": [
      {
        "type": "sftp",	// Linux默认文件上传方式
        "name": "AliyunServer",
        "dir": "/root/source/docker_ci",	// 服务器文件夹目标地址
        "host": "xx.xx.xx.xx",	// 主机地址
        "port": 22,	// 主机端口
        "user": "zhngsan",	// 主机用户名
        "privateKey": "/Users/xx/.ssh/xx.pem"	// 本机私钥地址
      }
    ]
  }
}

```

- 配置``nginx/conf.d/docker.conf``

```nginx
server {
    listen       80;
    location / {
        root   /var/www/html;
        index  index.html index.htm;
		}
    location ~ \.(gif|jpg|png)$ {
        root /static;
        index index.html index.htm;
    }
}
```

- 配置 ``docker-compose.yml``

```yaml
version: "3.1"
services:
  nginx:
    restart: always
    image: nginx
    ports:
      - 8091:80	// 将容器中的80端口映射到实体机的8091端口
    volumes:
      - ./nginx/conf.d/:/etc/nginx/conf.d		// 配置文件映射
      - ./frontend/dist:/var/www/html/			// 将/var/.. 目录映射到实体机打包dist文件夹
      - ./static/:/static/			// 静态文件映射
```

- 启动

```shell
docker-compose up
```



## 十、构建NodeJs

- 配置``process.yml``

```yaml
 apps:
  - script : server.js
    instances: 2
    watch  : true
    env    :
      NODE_ENV: production
```

- 配置``.dockerignore``

```
node_modules
```

- 配置``Dockerfile``

```dockerfile
FROM keymetrics/pm2:latest-alpine
WORKDIR /usr/src/app
ADD . /usr/src/app
RUN npm config set registry https://registry.npm.taobao.org/ && \  
    npm i
# RUN npm i
EXPOSE 3000
#pm2在docker中使用命令为pm2-docker
# CMD ["pm2-runtime", "start", "--json", "process.json"]
CMD ["pm2-runtime", "start",  "process.yml"]
```

- 修改数据库配置

```javascript
// conf.js
module.exports = {
    url: "mongodb://mongo:27017",
    dbName: 'taro',
}
```

- 修改 ``nginx/conf.d/conf``

```nginx
    location /api {
            proxy_pass  http://app-pm2:3000;
            proxy_redirect     off;
            proxy_set_header   Host             $host;
            proxy_set_header   X-Real-IP        $remote_addr;
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
```

- 修改 ``docker-compose``文件

```dockerfile
version: "3.1"
services:
  app-pm2:
    container_name: app-pm2
    #构建容器
    build: ./backend
    ports:
      - "3000:3000"
  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
```



## 十一、Webhook

- 新建一个Webhooks

  [一个简单的例子学会github repository的webhook](https://www.jianshu.com/p/7cbda59a191f)

- Webhook.js

```javascript
var http = require('http')
var createHandler = require('github-webhook-handler')
var handler = createHandler({ path: '/webhooks', secret: 'myHashSecret' })
// 上面的 secret 保持和 GitHub 后台设置的一致

function run_cmd(cmd, args, callback) {
    var spawn = require('child_process').spawn;
    var child = spawn(cmd, args);
    var resp = "";

    child.stdout.on('data', function (buffer) { resp += buffer.toString(); });
    child.stdout.on('end', function () { callback(resp) });
}
// debug用
// run_cmd('sh', ['./deploy-dev.sh'], function(text){ console.log(text) });

http.createServer(function (req, res) {

    handler(req, res, function (err) {
        res.statusCode = 404
        res.end('no such location')
    })
}).listen(7777,() =>{
    console.log('WebHooks Listern at 7777');
})

handler.on('error', function (err) {
    console.error('Error:', err.message)
})


handler.on('*', function (event) {
    console.log('Received *', event.payload.action);
    //   run_cmd('sh', ['./deploy-dev.sh'], function(text){ console.log(text) });
})
 
handler.on('push', function (event) {
    console.log('Received a push event for %s to %s',
        event.payload.repository.name,
        event.payload.ref);
        // 分支判断
        if(event.payload.ref === 'refs/heads/master'){
            console.log('deploy master..')
            run_cmd('sh', ['./deploy-dev.sh'], function(text){ console.log(text) });

        }
})


handler.on('issues', function (event) {
    console.log('Received an issue event for % action=%s: #%d %s',
        event.payload.repository.name,
        event.payload.action,
        event.payload.issue.number,
        event.payload.issue.title)
})


```



## 十二、实现持续集成

```sh
# deploy-dev.sh
echo Deploy Project
# docker-compose up -d --force-recreate --build

# 获取最新版代码
git pull

# 强制重新编译容器
docker-compose down
docker-compose up -d --force-recreate --build


# 定制镜像
# docker build -t myapp:pm2 ./backend

# 重启启动容器
# docker stop myapp
# docker rm myapp
# docker run --name myapp -p 3000:3000  -d myapp:pm2
```



## 十三、参考阅读

1、[使用pm2+nginx部署koa2(https)](https://www.zhaofinger.com/detail/5)

2、[详解 Node.Js 中实现端口重用原理](http://www.sohu.com/a/247732550_796914)

3、[PM2控制台检测](https://id.keymetrics.io/api/oauth/login#/register)