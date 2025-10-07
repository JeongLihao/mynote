# Docsify 搭建与部署手册

> 本文档记录基于 Docsify 的 Markdown 文档站点搭建过程，部署于 Debian 服务器

一、安装依赖环境

```sh
// 安装 Node.js 和 npm
   sudo apt update
   sudo apt install nodejs npm -y
```

二、确认版本

```sh
node -v
npm -v
```

三、安装 Docsify CLI 和 pm2

```sh
npm install -g docsify-cli pm2
```


---

### 一、初始化项目结构

1. 创建文本文档

   ```sh
   mkdir mynote
   cd mynote
   ```

2. 初始化Docsify项目

   ```sh
   # 在mynote中初始化
   docsify init ./docs
   ```

   + 初始化后，docs目录下应有以下内容
     + index.html（入口文件）
     + README.md（首页内容）
     + .nojekyll
   + MD笔记应放在docs目录下，同时可以创建子目录进行分类管理

+ 项目结构如下：

  ```
  .
  ├── docs/
  │   ├── index.html          # Docsify 主入口
  │   ├── README.md           # 首页内容
  │   ├── _sidebar.md         # 由脚本生成的导航栏
  │   └── notes/              # 存放各类 Markdown 文件
  │       ├── 20240904.md
  │       ├── ...
  └── generateSidebar.js      # 自动生成 _sidebar.md 的脚本
  ```

### 二、创建 index.html

+ 路径：docs/index.html

  ```sh
  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="UTF-8">
    <title>我的文档</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
  </head>
  <body>
    <div id="app"></div>
    <script>
      window.$docsify = {
        name: '我的笔记',
        repo: '',
        loadSidebar: true,
        subMaxLevel: 2,
        sidebarDisplayLevel: 1
      };
    </script>
    <script src="https://cdn.jsdelivr.net/npm/docsify@4"></script>
  </body>
  </html>
  ```

---

### 三、运行 Docsify 并托管至 pm2

```sh
pm2 start "docsify serve docs -p 3000" --name docsify-app
// 验证是否运行：
   pm2 list
// 设置开机自启（可选）：
   pm2 save
   pm2 startup
```


---

### 四、配置 nginx 反向代理

+ 示例

  ```sh
  server {
      listen 80;
      server_name 你的域名;
  
      location / {
          proxy_pass http://localhost:3000;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }
  ```
  
+ 启用配置文件

  ```shell
  ln -s /etc/nginx/sites-available/docsify /etc/nginx/sites-enabled/
  ```
  
+ 重载配置

  ```sh
  nginx -t
  systemctl reload nginx
  ```

---------------------------------------

# Docsify侧边栏

### 一、将md文件复制到Docsify目录

+ 复制到docs/目录下，可创建子文件夹分类管理

+ 示例

  ```
  my-docs/
  └── docs/
      ├── README.md         # 网站首页
      ├── getting-started/  # 分类文件夹
          ├── setup.md      # 子页面
          └── config.md
  ```

### 二、配置侧边栏

+ 需创建docs/_sidebar.md文件

+ 示例

  ```
  - [首页](README.md)
  - [入门指南](getting-started/setup.md)
    - [安装](getting-started/setup.md)
    - [配置](getting-started/config.md)
  ```

### 三、自定义网站标题和配置

+ 修改index.html

+ 示例

  ```sh
  <script>
    window.$docsify = {
      name: '我的笔记',        // 网站标题
      repo: '仓库链接',        //可不要
      loadSidebar: true,      // 启用侧边栏
      loadNavbar: false,       // 启用顶部导航栏
      auto2top: true,         // 滚动时自动回到顶部
    }
  </script>
  ```

### 四、重启Docsify服务

+ 示例

  ```sh
  pm2 restart docsify  # 重启服务
  pm2 list             # 查看服务状态（确保状态为 online）
  ```


### 五、侧边栏自动生成脚本

+ 注意：该脚本只推荐在插件失效的时候使用

+ 参照自动生成脚本

  [[自动生成脚本](https://note.lhjeong.cn/#/一些记录/Docsify自动脚本)]

### 六、搜索功能（扩展功能）

+ 该功能使用插件实现

+ 在index.html中添加如下内容

  ```js
  <script src="//unpkg.com/docsify/lib/docsify.min.js"></script>
  ```

-----------------------------------------------------

# 结果展示

+ 页面如下

  ![image-20250703103803231](http://img.lhjeong.cn/20250703103803312.png)

+ 访问网址---->[H-Jeong的note](https://note.lhjeong.cn/#/)

