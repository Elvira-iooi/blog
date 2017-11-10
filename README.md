# Blog

## 描述

1. Backup for Xiao's Blog.

## 博客迁移主机流程

### 配置Git

```bash
git config --global core.editor vim
git config --global merge.tool vimdiff
git config --global user.name "YuXiaoCoder"
git config --global user.email "xiao.950901@gmail.com"
git config --global color.ui true
```

### 主机间免密通信

```bash
chmod 600 ~/.ssh/github
chmod 600 ~/.ssh/xiaocoder
```

```bash
vim ~/.ssh/config
```

```text
Host github.com
HostName github.com
User YuXiao
IdentityFile ~/.ssh/github

Host 119.29.80.189
HostName 119.29.80.189
User root
IdentityFile ~/.ssh/xiaocoder
```

```bash
ssh -T git@github.com
ssh root@119.29.80.189 echo 'Hello, Bloger!!!'
```

### 拉取博客

```bash
cd /
git clone git@github.com:YuXiaoCoder/blog.git
```

### 拉取主题

```bash
cd /blog/
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

### 个性化

#### 添加汉化字段

+ 文件：`themes/next/languages/zh-Hans.yml`

```bash
vim themes/next/languages/zh-Hans.yml
```

+ 内容：

```text
menu:
  messages: 留言
```

#### 添加站点特效

+ 文件：`themes/next/layout/_layout.swig`

```bash
vim themes/next/layout/_layout.swig
```

+ 内容：

```text
<body>
    <script type="text/javascript" src="/js/src/particle.js"></script>
    <script type="text/javascript" src="/js/src/love.js"></script>
</body>
```

+ 位置：`backup/js/ --> themes/next/source/js/src`

```bash
\cp -f backup/js/* themes/next/source/js/src/
```

#### 优化文章样式

+ 文件：`themes/next/source/css/_custom/custom.styl`

```bash
\cp -f backup/css/custom.styl themes/next/source/css/_custom/custom.styl
```

```bash
vim themes/next/source/css/_custom/custom.styl
```

+ 内容：

```text
// Custom styles.

//首页文章阴影样式
.post {
  margin-top: 40px;
  margin-bottom: 40px;
  padding: 25px;
  -webkit-box-shadow: 0 0 14px rgba(202, 203, 203, .5);
  -moz-box-shadow: 0 0 14px rgba(202, 203, 204, .5);
}

//侧栏按钮样式
.sidebar-toggle {
    background: #649ab6;
}
.back-to-top {
    background: #649ab6;
}

//首页阅读全文样式
.post-button {
    margin-top: 30px;
    text-align: center;
}
.post-button .btn {
    color: #fff;
    font-size: 15px;
    background: #686868;
    border-radius: 16px;
    line-height: 2;
    margin: 0 4px 8px 4px;
    padding: 0 20px;
}
.post-button a{
  border-bottom: 1px solid #666;
}
.post-button a:hover {
    color: #7784ba;
}

//文章目录样式
.post-toc .nav .active>a {
    color: #4f7e96;
}

.post-toc ol a:hover {
    color: #7784ba;
}

.sidebar-nav .sidebar-nav-active:hover {
    color: #37596c;
}

// 文章标题
a {
    border-bottom: none;
}

.post-nav {
    border-top: 1px solid #58B2DC;
}

.posts-expand .post-tags {
    text-align: center;
}

.posts-expand .post-tags a {
    box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
    font-family: 'Comic Sans MS', sans-serif;
    transition: 0.2s ease-out;
}

.posts-expand .post-tags a:hover {
    background-color: #58B2DC;
    color: #f5f5f5;
}

.posts-expand .post-title:hover {
    border-left: #7DB9DE 20px solid;
    padding-left: 15px;
    cursor: pointer;
    transition: border-width 0.3s linear 0.1s, color 0.2s linear 0.3s;
}

// 圆形头像
.site-author-image {
    border-radius: 50%;
    padding: 2px;
    border: 2px solid #333; 
}

// 旋转并放大头像
.site-author-image:hover {
    -webkit-box-shadow: 0 0 10px rgba(0,0,0,.5);
    -moz-box-shadow: 0 0 10px rgba(0,0,0,.5);
    -o-box-shadow: 0 0 10px rgba(0,0,0,.5);
    -ms-box-shadow: 0 0 10px rgba(0,0,0,.5);
    box-shadow: 0 0 10px rgba(0,0,0,.5);

    -webkit-transform: rotate(180deg) scale(1.1);
    -moz-transform: rotate(180deg) scale(1.1);
    -ms-transform: rotate(180deg) scale(1.1);
    -o-transform: rotate(180deg) scale(1.1);
    transform: rotate(180deg) scale(1.1)
}
```

#### 优化链接颜色

+ 文件：`themes/next/source/css/_variables/base.styl`

```bash
\cp -f backup/css/base.styl themes/next/source/css/_variables/base.styl
```

```bash
vim themes/next/source/css/_variables/base.styl
```

+ 内容：

```text
// Colors
$my-link-blue = #6495ED  //链接颜色
$my-link-hover-blue = #0477AB  //鼠标悬停后颜色
$my-code-foreground = #DD0055  // 用``围出的代码块字体颜色
$my-code-background = #EEE  // 用``围出的代码块背景颜色

// Global link color.
$link-color                   = $my-link-blue
$link-hover-color             = $my-link-hover-blue
$link-decoration-color        = $gray-lighter
$link-decoration-hover-color  = $my-link-hover-blue

// Code & Code Blocks
// --------------------------------------------------
$code-font-family               = $font-family-monospace
$code-font-size                 = unit(hexo-config('font.codes.size'), px) if hexo-config('font.codes.size') is a 'unit'
$code-border-radius             = 4px
//$code-foreground                = $black-light
//$code-background                = $gainsboro
$code-background                = $my-code-background
$code-foreground                = $my-code-foreground
```

#### 使文章图片居中

+ 文件：`themes/next/source/css/_schemes/Mist/_posts-expanded.styl`

```bash
\cp -f backup/css/_posts-expanded.styl themes/next/source/css/_schemes/Mist/_posts-expanded.styl
```

```bash
vim themes/next/source/css/_schemes/Mist/_posts-expanded.styl
```

+ 内容:

```text
.post-body img {margin: 0 auto;}
```

#### 添加站点页脚

+ 文件：`themes/next/layout/_partials/footer.swig`

```bash
\cp -f backup/layout/footer.swig themes/next/layout/_partials/footer.swig
```

```bash
vim themes/next/layout/_partials/footer.swig
```

+ 内容：

```text
<div class="copyright" >
  <span class="author" itemprop="copyrightHolder">{{ config.author }}</span>
  &nbsp;&nbsp;
  <a href="http://www.miitbeian.gov.cn/">蜀ICP备17004598号-1</a>
</div>
```

***
