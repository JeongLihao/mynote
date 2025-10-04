# docker容器实践

## 0. 前言

### 1. docker的基本架构和优势

1. 基本架构

   1. docker daemon (守护进程)
      + 负责管理 Docker 容器、镜像和网络等资源
      + 负责监听客户端的请求并响应操作，如容器的创建、运行、停止和删除等
   2. docker cli (命令行界面)
      + 是用户与 Docker 交互的工具，它是客户端的主要接口
   3. docker images (镜像)
      + 用于创建 Docker 容器的模板
   4. docker containers (容器)
      + 镜像的运行实例
   5. docker registry (镜像仓库)
      + 用于存储和分发 Docker 镜像的服务
   6. docker network (网络)
      + 提供了多种网络模式来管理容器之间的通信
   7. docker volumes (卷)
      +  用于持久化容器中的数据

2. 如图所示

   ```lua
   +------------------+    +-----------------+
   |    Docker CLI    | <--->| Docker Daemon   |
   |  (客户端接口)    |    | (守护进程)      |
   +------------------+    +-----------------+
                             ^
                             |
                    +---------------------+
                    | Docker Containers   |
                    | (容器实例)          |
                    +---------------------+
                             ^
                             |
              +---------------------------+
              | Docker Images (镜像)       |
              +---------------------------+
                             ^
                             |
              +-----------------------------+
              | Docker Registry (镜像仓库) |
              +-----------------------------+
   ```

3. docker的优势
   + 一致性和可移植性
   + 轻量级与高效性
   + 简化的应用部署和管理
   + 可扩展性和高可用性
   + 隔离性与安全性
   + 简化的开发环境

### 2. docker的基础操作

#### 1. 镜像管理

1. 查看已下载的镜像

   ```sh
   docker images
   ```

2. 拉取docker镜像

    ```sh
    docker pull <image_name>
    如
    docker pull ubuntu
    ```

3. 删除本地镜像

    ```sh
    docker rmi <image_name>
    如
    docker rmi ubuntu
    ```

4. 查找docker镜像

    ```sh
    docker search <image_name>  
    如
    docker search nginx
    ```

5. 构建docker镜像

    ```sh
    docker build -t <image_name> .
    如
    docker build -t myapp .
    ```

#### 2. 容器管理

1. 运行一个docker容器

    ```sh
    docker run <options> <image_name>
    如
    docker run -it ubuntu
    ```

2. 运行一个后台容器

    ```sh
    docker run -d <image_name>
    如
    docker run -d nginx
    ```

3. 查看正在运行的容器

    ```sh
    docker ps 
    ```

4. 查看所有容器（包括已停止的）

    ```sh
    docker ps -a
    ```

5. 停止容器

    ```sh
    docker stop <container_id>
    如
    docker stop abcdef12345
    ```

6. 启动已停止的容器

    ```sh
    docker start <container_id>
    ```

7. 删除容器

    ```sh
    docker rm <container_id>
    如
    docker rm abcdef12345
    ```

8. 进入正在运行的容器

    ```sh
    docker exec -it <container_id> /bin/bash
    如
    docker exec -it abcdef12345 /bin/bash
    ```

9. 查看容器日志

    ```sh
    docker logs <container_id>  
    ```

10. 停止并删除容器

     ```sh
     docker stop <container_id>
     docker rm <container_id> 
     ```

#### 3. 容器与镜像管理

1. 将容器的端口映射到宿主机的端口

    ```sh
    docker run -d -p <host_port>:<container_port> <image_name>
    如
    docker run -d -p 8080:80 nginx
    ```

2. 为容器挂载本地目录（持久化数据）

    ```sh
    docker run -d -v /path/on/host:/path/in/container <image_name> 
    ```

3. 查看容器资源使用情况

    ```sh
    docker stats
    ```

4. 获取容器的详细信息

    ```sh
    docker inspect <container_id>  
    ```

#### 4. 网络管理

1. 查看docker网络

    ```sh
    docker network ls 
    ```

2. 创建docker网络

    ```sh
    docker network create <network_name> 
    ```

3. 连接容器到网络

    ```sh
    docker network connect <network_name> <container_id>
    ```

4. 从网络中断开容器

    ```sh
    docker network disconnect <network_name> <container_id>  
    ```

#### 5. 容器的清理

1. 清理未使用的镜像

    ```sh
    docker image prune  
    ```

2. 清理所有未使用的容器、网络和镜像

    ```sh
    docker system prune -a 
    ```

### 3. Docker 与传统虚拟机的对比

| 特性           | Docker                                 | 传统虚拟机                             |
| -------------- | -------------------------------------- | -------------------------------------- |
| **架构**       | 基于操作系统级虚拟化（共享内核）       | 基于硬件虚拟化（独立操作系统）         |
| **启动速度**   | 快（几秒钟）                           | 慢（需要加载操作系统）                 |
| **资源占用**   | 轻量级，占用少                         | 较重，资源占用高                       |
| **隔离性**     | 应用级隔离，共享宿主机内核             | 完全隔离，各自拥有独立的操作系统和内核 |
| **性能**       | 高性能，几乎没有额外开销               | 性能损失较大，由于虚拟化带来的额外开销 |
| **可移植性**   | 高（跨平台一致性好）                   | 低，迁移较为复杂                       |
| **管理和运维** | 容器编排工具如 Kubernetes 提供易管理性 | 管理平台如 VMware，管理更复杂          |
| **持久化存储** | 需要使用卷挂载实现持久化存储           | 虚拟机内的虚拟磁盘自带持久化存储       |
| **适用场景**   | 微服务、快速部署、持续集成/持续交付    | 传统应用、需要操作系统虚拟化的应用     |

## 1. docker的安装

1. 安装

   ```sh
   apt install docker.io
   ```

2. 修改源（如果pull失败的话）

   + `vim /etc/docker/daemon.json`

   ```ini
    { 
     	"registry-mirrors": [ "https://docker.xuanyuan.me" ] 
    } 
   ```

3. 验证

   ```sh
   docker --version
   docker run hello-world
   ```

   ![image-20250728142109591](https://img.lhjeong.cn/20250728142109794.png)

## 2. docker基础操作

1. 拉取一个镜像（如nginx）

   ```sh
   docker pull nginx 
   ```

   ![image-20250728142259360](https://img.lhjeong.cn/20250728142259459.png)

2. 启动这个镜像并访问它

   ```sh
   docker run --name mynginx -d -p80:80 nginx
   
   // docker ps  查看当前容器
   // docker ps -a 查看所有容器
   ```

   ![image-20250728143840760](https://img.lhjeong.cn/20250728143840833.png)

   ![image-20250728143858143](https://img.lhjeong.cn/20250728143858180.png)

3. 停止并删除容器

   ```sh
   // 停止
   root@debian:~# docker stop mynginx
   // 删除
   root@debian:~# docker rm mynginx 
   ```

4. 删除本地镜像

   ```sh
   // 查看镜像
   root@debian:~# docker images
   // 删除镜像
   root@debian:~# docker rmi nginx:latest 
   ```

   ![image-20250728144514988](https://img.lhjeong.cn/20250728144515120.png)

## 3. dockerfile 编写与镜像构建

1. 编写一个简单的dockerfile

   ```dockerfile
   FROM nginx:alpine
   COPY index.html /opt/test/html
   CMD [ "nginx","-g","daemon off;"]
   ```

2. 创建`index.html`文件

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>hello docker</title>
   </head>
   <body>
       <h1>hello docker</h1>
   </body>
   </html>
   ```

3. 构建镜像

   ```sh
   docker build -t hello-docker .
   ```

4. 运行镜像

   ```sh
   docker run -d -p 8080:80 hello-docker
   ```

   ![image-20250728151700771](https://img.lhjeong.cn/20250728151700886.png)

5. yaml文件格式

   ```yaml
   version: '3'
   services:
     web:
       image: nginx
       ports:
         - "8080:80"
   ```

## 4. docker compose基础

1. 作用和优势

   1. 简化配置：可以在一个文件中定义多个服务，并一键启动

   2. 服务之间的连接：可以通过服务名而非 IP 地址来让容器之间互相通信

   3. 方便管理：提供了简单的命令来启动、停止、查看日志等

2. 编写一个简单的docker compose文件

   ```yaml
   version: '3'
   services:
     portainer:
       image: portainer/portainer-ce:latest
       container_name: portainer
       restart: always
       ports:
         - "9000:9000"
       volumes:
         - /var/run/docker.sock:/var/run/docker.sock
         - portainer_data:/data
   
   volumes:
     portainer_data:
   ```

3. 安装docker compose

   ```sh
   apt update
   apt install docker-compose
   ```

4. 启动

   ```sh
   docker-compose up -d
   docker-compose [项目名] up -d
   ```

   ![image-20250728154405921](https://img.lhjeong.cn/20250728154406310.png)

   ![image-20250728154431212](https://img.lhjeong.cn/20250728154431299.png)

5. 移除

   ```sh
   // 1. 移除所有
   docker-compose down
   // 2. 移除单个
   docker-compose -f ghostblog.yml down
   ```

## 5. gitee开源项目部署

1. 使用git拉取代码

   ```sh
   git clone https://gitee.com/mirrors/FastAPI.git
   ```

2. 编写`main.py`文件

   ```python
   from fastapi import FastAPI
   
   app = FastAPI()
   
   @app.get("/")
   def read_root():
       return {"Hello": "This is a FastAPI app running in Docker!", "source": "built from /opt/FastAPI source"}
   
   @app.get("/items/{item_id}")
   def read_item(item_id: int, q: str = None):
       return {"item_id": item_id, "q": q}
   ```

3. 编写dockerfile

   ```dockerfile
   FROM python:3.9-slim
   
   WORKDIR /app
   
   COPY . /app
   
   RUN pip install --no-cache-dir -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
   
   EXPOSE 8000
   
   CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
   ```

4. 构建并运行

   ```sh
   // 构建
      docker build -t dockerfile .
   // 运行
      docker run --name fast -p 7777:8000 dockerfile
   // 访问
      http://192.168.231.2:7777/docs#/default
      http://192.168.231.2:7777/redoc
   ```

5. 查看

   ```sh
   docker ps 
   ```

   ![image-20250729102507273](https://img.lhjeong.cn/20250729102507407.png)

6. 项目展示

   ![image-20250729102603228](https://img.lhjeong.cn/20250729102603336.png)

## 6. 原生方式向docker方式迁移

### 1. 项目一：dedecms

1. 编写dockerfile

   ```dockerfile
   # 使用官方 PHP 镜像作为基础镜像
   FROM php:7.4-fpm
   # 替换为国内镜像源
   RUN sed -i 's|http://deb.debian.org|http://mirrors.ustc.edu.cn|g' /etc/apt/sources.list
   # 安装 Nginx
   RUN apt-get update && apt-get install -y nginx && apt-get clean && rm -rf /var/lib/apt/lists/*
   # 安装 PHP 扩展所需依赖和扩展
   RUN apt-get update && apt-get install -y \
       libpng-dev \
       libjpeg-dev \
       libfreetype6-dev \
       default-libmysqlclient-dev \
       && docker-php-ext-configure gd --with-freetype --with-jpeg \
       && docker-php-ext-install -j$(nproc) \
           gd \
           mysqli \
           pdo_mysql \
           zip \
       && apt-get clean \
       && rm -rf /var/lib/apt/lists/*
   # 配置 Nginx
   COPY ./dedecms.conf /etc/nginx/conf.d/
   # 将 DedeCMS 代码拷贝到容器
   COPY ./uploads /var/www/html/
   # 设置工作目录
   WORKDIR /var/www/html
   # 设置适当的权限
   RUN chown -R www-data:www-data /var/www/html
   # 端口
   EXPOSE 9020
   # 启动 Nginx 和 PHP-FPM
   CMD service nginx start && php-fpm
   ```

2. 编写nginx文件

   ```ini
   server {
       listen 80;
       server_name localhost;
   
       root /var/www/html;
       index index.php index.html;
   
       location / {
           try_files $uri $uri/ /index.php?$args;
       }
   
       location ~ \.php$ {
           include fastcgi_params;
           fastcgi_pass 127.0.0.1:9020;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           include fastcgi_params;
       }
   }
   ```

3. 构建

   ```sh
   docker build -t dedecms .
   docker run -d -p 9999:80 --name dede dedecms
   ```

4. 进入容器编写`common.inc.php`

   `docker exec -it dede /bin/bash`

   ```php
   <?php
   $cfg_dbhost = 'mysql.mxdx.com';
   $cfg_dbname = 'dedecms';
   $cfg_dbuser = 'su';
   $cfg_dbpwd = '123456';
   $cfg_dbprefix = 'dede_';
   $cfg_db_language = 'utf8mb4';
   ?>
   ```

5. 展示

   ![image-20250730101408431](https://img.lhjeong.cn/20250730101415722.png)

   ![image-20250730101431587](https://img.lhjeong.cn/20250730101431852.png)

6. 假设mysql扩展没有成功安装

   1. 确认能否编译安装

      ```sh
      which phpize
      which php-config
      phpize --version
      
      // 应输出
      /usr/bin/phpize
      /usr/bin/php-config
      Configuring for: ...
      PHP Api Version: 20190902
      Zend Module Api No: 20190902
      Zend Extension Api No: 320190902
      ```

   2. 安装编译工具和依赖库

      ```sh
      apt update
      
      apt install -y \
          autoconf \
          automake \
          build-essential \
          pkg-config \
          git \
          libxml2-dev \
          libcurl4-openssl-dev \
          libjpeg-dev \
          libpng-dev \
          libfreetype6-dev \
          libzip-dev \
          zlib1g-dev \
          libssl-dev \
          default-libmysqlclient-dev
      ```

   3. 下载源码

      ```sh
      wget https://www.php.net/distributions/php-7.4.33.tar.gz
      tar -xzf php-7.4.33.tar.gz
      ```

   4. 编译mysqli扩展

      ```sh
      cd /tmp/php-7.4.33/ext/mysqli
      phpize
      ./configure
      make && make install
      
      // 应输出
      Installing shared extensions:     /usr/lib/php/20190902/
      ```

   5. 编译pdo_mysql扩展

      ```sh
      cd /tmp/php-7.4.33/ext/pdo_mysql
      phpize
      ./configure
      make && make install
      
      // 应输出
      # 写入配置文件
      echo "extension=mysqli.so" > /etc/php/7.4/cli/conf.d/20-mysqli.ini
      echo "extension=pdo_mysql.so" > /etc/php/7.4/cli/conf.d/20-pdo_mysql.ini
      ```

   6. 验证

      ```sh
      php -m | grep -E "(mysqli|pdo_mysql)"
      // 应看到
      mysqli
      pdo_mysql
      ```

   7. 配置文件

      ```sh
      echo "extension=mysqli.so" > /usr/local/php/conf.d/20-mysqli.ini
      echo "extension=pdo_mysql.so" > /usr/local/etc/php/conf.d/20-pdo_mysql.ini
      ```

   8. 验证

      ```sh
      php -m | grep -E "(mysqli|pdo_mysql)"
      // 应输出
      mysqli
      pdo_mysql
      ```

### 2. 项目二：wordpress

1. 展示

   ![image-20250730103344712](https://img.lhjeong.cn/20250730103344835.png)

2. 编写dockerfile

   ```dockerfile
   # 使用官方的 WordPress 镜像
   FROM wordpress:latest
   
   # 配置 WordPress 需要的端口
   EXPOSE 80
   
   # 启动 WordPress 服务
   CMD ["apache2-foreground"]
   ```

3. 编写docker-compose.yml

   ```yaml
   version: '3.7'
   
   services:
     wordpress:
       build: .
       ports:
         - "9898:80" # 端口映射
       environment:
         WORDPRESS_DB_HOST: 192.168.231.3:3307  # 外部数据库的 IP 和端口
         WORDPRESS_DB_NAME: blogtest  # 数据库名
         WORDPRESS_DB_USER: su  # 数据库用户名
         WORDPRESS_DB_PASSWORD: 123456  # 数据库密码
       volumes:
         - wordpress_data:/var/www/html  # 数据持久化存储
   
   volumes:
     wordpress_data:
   ```

4. 启动

   ```sh
   docker-compose up -d --build
   // 访问
      192.168.231.4:9898
   ```

## 7. 镜像分享

1. 将容器保存为镜像

   ```sh
   docker commit halo jeong:halo
   ```

2. 将镜像保存为tar文件

   ```sh
   docker save -o halo.tar jeong:halo
   ```

3. 导入

   ```sh
   docker load -i halo.tar
   ```

4. 运行

   ```sh
   docker run -d --name haloblog -p 8090:8090 jeong:halo
   ```

   ![image-20250731092402411](https://img.lhjeong.cn/20250731092402513.png)

## 8. 搭建镜像仓库

1. 启动docker registry

   ```sh
   docker run -d -p 5000:5000 --name registry registry:2
   ```

2. 标记

   ```sh
   docker tag jeong:halo localhost:5000/jeong:halo
   ```

3. 推送到仓库

   ```sh
   docker push localhost:5000/jeong:halo
   ```

4. 从其他服务器拉取

   + 编辑`/etc/docker/daemon.json`（客户端，不是仓库的配置文件）

     ```json
     {
       "insecure-registries": ["localhost:5000"]
     }
     ```

   + 重启docker

     ```sh
     systemctl restart docker
     ```

   + 拉取

     ```
     docker pull localhost:5000/jeong:halo
     ```

     ![image-20250801154725814](https://img.lhjeong.cn/20250801154725877.png)

5. 把容器保存为镜像

   1. 查看容器ID或名称

      ```sh
      docker ps
      ```

   2. 使用docker commit创建镜像

      ```sh
      docker commit mynginx-container localhost:5000/mynginx
      ```

   3. 检查镜像

      ```sh
      docker images
      ```

6. 启动

   ```sh
   docker run -d --name halo -p 8090:8090 192.168.231.4:5000/jeong:halo
   ```

   ![image-20250801154915348](https://img.lhjeong.cn/20250801154915427.png)

