# Docsify自动脚本

+ 自动扫描 `docs/notes/` 目录中的 `.md` 文件，读取其首个 `# 一级标题` 作为名称，并生成 `_sidebar.md`

### 以下仅作参考

```js
const fs = require('fs'); // 引入文件系统模块，用于读写文件
const path = require('path'); // 引入路径模块，用于处理文件路径
// 设置 Docsify 根目录和 _sidebar.md 的路径
const rootDir = path.join(__dirname, 'docs');
const sidebarFile = path.join(rootDir, '_sidebar.md');
/**
 * 从 Markdown 文件中提取第一个一级标题作为标题
 * 如果没有一级标题，则返回文件名作为标题
 * @param {string} filePath - 文件完整路径
 * @returns {string} 提取到的标题或文件名
 */
function getTitleFromFile(filePath) {
  try {
    const content = fs.readFileSync(filePath, 'utf-8'); // 读取文件内容
    const lines = content.split('\n'); // 按行分割

    for (let line of lines) {
      const match = line.trim().match(/^# (.+)/); // 匹配一级标题 "# 标题"
      if (match) {
        return match[1].trim(); // 返回标题内容
      }
    }
    // 若无标题，则返回不带扩展名的文件名
    return path.basename(filePath, '.md');
  } catch (err) {
    console.error(`读取失败: ${filePath}`, err);
    return path.basename(filePath, '.md');
  }
}
/**
 * 递归遍历目录，生成 Docsify 所需的 sidebar 列表
 * @param {string} dir - 当前遍历的目录路径
 * @param {string} prefix - 缩进前缀，用于控制嵌套层级
 * @returns {string[]} sidebar 条目数组
 */
function walk(dir, prefix = '') {
  const items = fs.readdirSync(dir, { withFileTypes: true }); // 获取目录下所有条目
  let mdList = [];

  // 字典序排序，确保输出有序
  items.sort((a, b) => a.name.localeCompare(b.name));

  for (const item of items) {
    const fullPath = path.join(dir, item.name); // 获取完整路径
    const relativePath = path.relative(rootDir, fullPath).replace(/\\/g, '/'); // 转为相对路径（兼容 Windows）

    // 跳过自身（防止死循环）
    if (item.name === '_sidebar.md') continue;

    if (item.isDirectory()) {
      // 递归处理子目录
      const children = walk(fullPath, prefix + '  ');
      if (children.length > 0) {
        mdList.push(`${prefix}* ${item.name}/`); // 目录标题
        mdList = mdList.concat(children); // 添加子项目
      }
    } else if (item.isFile() && item.name.endsWith('.md')) {
      // 忽略 README.md（通常由 Docsify 默认显示）
      if (item.name.toLowerCase() === 'readme.md') continue;

      const title = getTitleFromFile(fullPath); // 提取标题
      console.log(`找到: ${relativePath} => 标题: ${title}`);
      mdList.push(`${prefix}* [${title}](${relativePath})`); // 添加 Markdown 链接
    }
  }
  return mdList;
}
// 生成 sidebar 内容并写入 _sidebar.md
const sidebarContent = walk(rootDir).join('\n');
fs.writeFileSync(sidebarFile, sidebarContent, 'utf-8');

console.log(`_sidebar.md 已生成并更新：${sidebarFile}`);
```

### 使用方法

```sh
node generateSidebar.js
```

-----------------------------

# 自动化脚本

+ 该脚本仅作参考

+ 参考脚本命名为generateSidebar.js

  ```bash
  #!/bin/bash
  # 将文件拉取并复制到目录文件夹中
  cd /home/gitwork/training-5-groups
  git pull
  cp *.md /home/mynotebook/docs/一些记录
  cd /home/mynotebook
  # 使用目录脚本生成新目录并重新加载
  node generateSidebar.js
  pm2 restart docsify
  # 设置周期任务
  20 23 * * *  /home/gitsh/trainingJeong.sh
  ```
