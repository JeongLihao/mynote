# 错误收录

### 侧边栏无内容

+ 解决方案

  1. 确保index.html中启用了配置项

     ```
     window.$docsify = {
       loadSidebar: true,
       subMaxLevel: 2,
       sidebarDisplayLevel: 1
     }
     ```

  2. `_sidebar.md` 放在 `docs/` 根目录下

### 样式丢失

+ 解决方案（引入主题）
  ```
  // 添加<link>标签
     <link rel="stylesheet"href="https://cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
  ```

### 图片图床无法显示（使用gitee）

+  Gitee 图床外链不稳定，有时会拦截

+ 解决方案

  1. 使用别的图床
  2. 或将图片直接放在服务器中


### 插件失效导致侧边栏无法自动生成

+ 解决方案
  1. 手动编写_sidebar.md文档
  2. 使用脚本，参考--->[目录自动生成脚本](https://note.lhjeong.cn/#/一些记录/Docsify自动脚本)

### 无法通过域名访问

+ 解决方案

  1. 配置DNS解析

     ![屏幕截图 2025-07-03 102602](http://img.lhjeong.cn/20250703102806172.png)

  2. 重启nginx配置