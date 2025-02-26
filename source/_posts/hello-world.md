---
title: Hello World
author: Zane.
top:
date: 2024-05-20 13:53:12
description: 欢迎来到我的第一篇博客
tags:
categories:
img:
cover: https://img3.wallspic.com/crops/2/9/8/1/7/171892/171892-lu_xing-cheng_shi-li_cheng_bei-cheng_shi_jing_guan-3840x2160.jpg
coverImg: 
password: 
---
## 这是我发的第一篇博客,以此证明我在这个世上存在过……

## 如何发表文章

### 安装hexo渲染器
```bash
npm install hexo-renderer-pug hexo-renderer-stylus
```
由于公式原因,需要更换渲染器里的东西,开启KaTeX 需要把 use 设置为 katex
```yaml
katex:
  # Enable the copy KaTeX formula
  copy_tex: false
```
你不需要添加 katex.min.js 來渲染数学方程。相应的你需要卸載你之前的 hexo 的 markdown 渲染器，然后an zhuang其它插件。

hexo-renderer-markdown-it 【建議】
hexo-renderer-markdown-it-plus
卸載掉 marked 插件，安裝 hexo-renderer-markdown-it

npm un hexo-renderer-marked --save # 如果有安裝這個的話，卸載
npm un hexo-renderer-kramed --save # 如果有安裝這個的話，卸載

npm i hexo-renderer-markdown-it --save # 需要安裝這個渲染插件
npm install katex @renbaoshuo/markdown-it-katex #需要安裝這個katex插件
在 hexo 的根目錄的 _config.yml 中配置


作者: Jerry
連結: https://butterfly.js.org/posts/4aa8abbe/
來源: Butterfly
版權屬於作者所有。商業用途請聯絡作者獲得授權，非商業用途請註明出處。
### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
<!--stackedit_data:
eyJoaXN0b3J5IjpbOTQ5OTUzMDM2XX0=
-->