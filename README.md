# Blog
+ 描述：Backup for Xiao's Blog

# 流程
+ 拉取博客
```
git clone git@github.com:YuXiaoCoder/blog.git
```
+ 切换目录
```
cd blog/
```
+ 拉取主题
```
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

# 个性化
+ 文件：`theme/next/languages/zh-Hans.yml`
+ 内容：
```
menu:
  messages: 留言
```
+ 文件：`theme/next/layout/_layout.swig`
+ 内容：

```
<script type="text/javascript" src="/js/src/particle.js"></script>
<script type="text/javascript" src="/js/src/love.js"></script>
```
+ 位置：`backup/js/ --> themes/next/source/js/src`

+ 文件：`theme/next/layout/_partials/head/external-fonts.swig`
+ 内容：
```
{% if font_families !== '' %}
  {% set font_host = font_config.host | default('//fonts.lug.ustc.edu.cn') %}
{% endif %}
```

+ 文件：`theme/next/source/css/_custom/custom.styl`
+ 内容：
```
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
```
+ 文件：`theme/next/source/css/_variables/base.styl`
+ 内容：
```
// Global link color.
//$link-color                   = $black-light
//$link-hover-color             = $black-deep
//$link-decoration-color        = $grey-light
//$link-decoration-hover-color  = $black-deep
$link-color                   = $my-link-blue
$link-hover-color             = $my-link-hover-blue
$link-decoration-color        = $gray-lighter
$link-decoration-hover-color  = $my-link-hover-blue

// Code & Code Blocks
// --------------------------------------------------
$code-font-family               = $font-family-monospace
//$code-font-size                 = 13px
$code-font-size                 = unit(hexo-config('font.codes.size'), px) if hexo-config('font.codes.size') is a 'unit'
//$code-border-radius             = 3px
$code-border-radius             = 4px
//$code-foreground                = $black-light
//$code-background                = $gainsboro
$code-background                = $my-code-background
$code-foreground                = $my-code-foreground
```
+ 文件：`theme/next/source/css/_schemes/Mist/_posts-expanded.styl`
+ 内容:
```
.post-body img {margin: 0 auto;}
```
