---
title: Sublime Text初体验
date: 2017-04-20 18:30:53
tags: [Install]
---

## 简介
+ 子曰：`工欲善其事 必先利其器`，一款优秀的IDE(集成开发环境)不仅提供很多强大的工具(代码提示、代码纠错、括号匹配...)，而且可以提供一个舒适的编程环境；
+ Sublime Text：一款收费闭源的跨平台、可扩展的文本编辑器；

## 相关网址
+ Sublime Text：[官网](https://www.sublimetext.com/)，[Download](https://www.sublimetext.com/3)；
+ Hack：[官网](http://sourcefoundry.org/hack/)；
+ 主题：[优秀主题](https://packagecontrol.io/browse/labels/theme)，[Boxy](https://packagecontrol.io/packages/Boxy%20Theme)；

<!-- more -->

## 安装指导
+ 在`Sublime Text`官网下载对应系统的版本，本文下载的为`Windows_X64`的便携版；
+ 将文件解压到安装目录，并重命名为`SublimeText`；
+ 推荐安装编程字体：`Hack`；
+ 目录中`sublime_text.exe`为可执行文件，推荐为其创建桌面快捷键；

## 个性化
### Settings

+ `Preferences >> Settings`

```text
// 使光标闪动更柔和
"caret_style": "phase",
// 字体(必须安装Hack字体,可自定义)
"font_face": "Hack",
// 字体大小
"font_size": 14,
// 高亮当前行
"highlight_line": true,
// 高亮修改的标签
"highlight_modified_tabs": true,
// 转换Tab为Space
"translate_tabs_to_spaces": true,
// 添加行宽标尺
"rulers": [80, 100],
// 显示空白字符
"draw_white_space": "all",
// 保存时自动去除行末空白
"trim_trailing_white_space_on_save": false,
// 保存时自动增加文件末尾换行
"ensure_newline_at_eof_on_save": true,
```

### 安装`Package Control`

+ `Sublime Text 3`安装`Package Control`；
+ 使用`Ctrl + ~`，打开控制台，若无法打开控制台，可能为输入法占用了快捷键；
+ 安装完成后，重启`Sublime Text 3`；

```text
import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

### 安装常用插件

+ 使用`Ctrl + Shift + P`，输入`install Package`，然后输入插件名即可；
+ AdvancedNewFile：新建文件(Ctrl + Alt + N)；
+ AutoFileName：文件名；
+ AutoPep8：Python代码格式化(Ctrl + Shift + 8)；
+ Boxy Theme：Boxy主题；
+ Boxy Theme Addon - Font Face：Boxy主题用于设置字体；
+ Emmet：前端神器；
+ ConvertToUTF8：自动将文本转换为UTF-8格式；
+ IMESupport：解决输入法不跟随问题；
+ SublimeLinter：语法高亮；
+ SideBarEnhancements：增强侧边栏功能；

### 主题设置
+ `Preferences >> Settings`
+ 冷色`Style`

```text
"color_scheme": "Packages/Boxy Theme/schemes/Boxy Ocean.tmTheme",
"theme": "Boxy Ocean.sublime-theme",
"theme_accent_cyan": true,
"theme_bar": true,
"theme_bar_logo_atomized": true,
"theme_bar_shadow_hidden": true,
"theme_button_rounded": true,
"theme_dirty_colored_always": true,
"theme_icons_materialized": true,
"theme_scrollbar_rounded": true,
"theme_sidebar_highlight_selected_text_only": true,
"theme_sidebar_highlight_text_only": true,
"theme_sidebar_indent_top_level_disabled": true,
"theme_size_md": true,
"theme_tab_highlight_text_only": true,
"theme_tab_selected_transparent": true,
"theme_tab_selected_underlined": true,
"theme_tab_size_xxl": true,
"theme_unified": true,
```

+ 暖色`Style`

```text
"color_scheme": "Packages/Boxy Theme/schemes/Boxy Monokai.tmTheme",
"theme": "Boxy Monokai.sublime-theme",
"theme_accent_tangerine": true,
"theme_autocomplete_item_selected_colored": true,
"theme_bar_margin_top_sm": true,
"theme_dropdown_atomized": true,
"theme_find_panel_close_hidden": true,
"theme_icon_button_highlighted": true,
"theme_panel_switcher_atomized": true,
"theme_quick_panel_item_selected_colored": true,
"theme_quick_panel_size_md": true,
"theme_scrollbar_colored": true,
"theme_scrollbar_line": true,
"theme_sidebar_close_always_visible": true,
"theme_sidebar_folder_atomized": true,
"theme_statusbar_size_md": true,
"theme_tab_close_always_visible": true,
"theme_tab_selected_overlined": true,
"theme_tab_size_md": true,
```

+ 护眼`Style`

```text
"color_scheme": "Packages/Boxy Theme/schemes/Boxy Solarized Light.tmTheme",
"theme": "Boxy Solarized Light.sublime-theme",
"theme_accent_green": true,
"theme_bar": true,
"theme_bar_colored": true,
"theme_bar_shadow_hidden": true,
"theme_button_rounded": true,
"theme_icons_atomized": true,
"theme_quick_panel_size_md": true,
"theme_scrollbar_rounded": true,
"theme_sidebar_close_always_visible": true,
"theme_sidebar_folder_materialized": true,
"theme_sidebar_highlight_selected_text_only": true,
"theme_sidebar_highlight_text_only": true,
"theme_sidebar_indent_top_level_disabled": true,
"theme_statusbar_size_md": true,
"theme_tab_highlight_text_only": true,
"theme_tab_line_size_lg": true,
"theme_tab_selected_transparent": true,
"theme_tab_selected_underlined": true,
"theme_tab_size_lg": true,
```

+ `AtomStyle`

```text
"color_scheme": "Packages/Boxy Theme/schemes/Boxy Solarized Light.tmTheme",
"theme": "Boxy Solarized Light.sublime-theme",
"theme_accent_green": true,
"theme_bar": true,
"theme_bar_colored": true,
"theme_bar_shadow_hidden": true,
"theme_button_rounded": true,
"theme_icons_atomized": true,
"theme_quick_panel_size_md": true,
"theme_scrollbar_rounded": true,
"theme_sidebar_close_always_visible": true,
"theme_sidebar_folder_materialized": true,
"theme_sidebar_highlight_selected_text_only": true,
"theme_sidebar_highlight_text_only": true,
"theme_sidebar_indent_top_level_disabled": true,
"theme_statusbar_size_md": true,
"theme_tab_highlight_text_only": true,
"theme_tab_line_size_lg": true,
"theme_tab_selected_transparent": true,
"theme_tab_selected_underlined": true,
"theme_tab_size_lg": true,
```

### 设置UI字体
+ `Preferences >> Browse Packages`，切换到`Boxy Theme Addon - Font Face`目录；
+ 打开与你选择主题名称相同的文件；
+ 替换`Sans-Serif`字体为您钟爱的字体；

### 注册码

```text
—– BEGIN LICENSE —–
Ryan Clark
Single User License
EA7E-812479
2158A7DE B690A7A3 8EC04710 006A5EEB
34E77CA3 9C82C81F 0DB6371B 79704E6F
93F36655 B031503A 03257CCC 01B20F60
D304FA8D B1B4F0AF 8A76C7BA 0FA94D55
56D46BCE 5237A341 CD837F30 4D60772D
349B1179 A996F826 90CDB73C 24D41245
FD032C30 AD5E7241 4EAA66ED 167D91FB
55896B16 EA125C81 F550AF6B A6820916
—— END LICENSE ——
```

***