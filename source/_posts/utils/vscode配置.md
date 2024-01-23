---
title: vscode配置
date: 2020-03-25 19:06:04
tags: 工具
categories: 工具
---

# 设置项
## 缩进
文件->首选项->设置->搜索`tab`->`"editor.tabSize": 2`
## 首选引用样式
文件->首选项->设置->搜索`quote`->`"javascript.preferences.quoteStyle": "single"  "typescript.preferences.quoteStyle": "single"`

# 推荐插件
| 插件名 | 描述 |
| :- | :- |
| Auto Close Tag | 自动补全关闭标签 |
| Auto Rename Tag | 自动重命名标签 |
| Beautify | HTML、CSS、JS、JSON SASS语法高亮,格式化代码的工具 |
| Bracket Pair Colorizer | 对应括号显示同样的颜色，以防我们搞混括号配对 |
| Code Runner | 能够运行多种语言的代码片段或代码文件 |
| Document This | 可以快速地帮你生成注释 |
| Easy Sass | 编译输出css |
| JavaScript (ES6) code snippets | es6的代码自动补全 |
| jQuery Code Snippets | jquery代码自动补全，比如你要写一段ajax你需要输入jqAjax然后敲回车 |
| Path Autocomplete | 路径补全工具 |
| IntelliSense for CSS class names in HTML | CSS 类名智能提示 |
| Debugger for Chrome | 必备调试工具 |
| ESLint | 代码检查工具 |
| vscode-icons | 文件图标 |
| vue-beautify | vue文件格式化 |
| Vetur | Vue 语法高亮显示, 语法错误检查, 代码自动补全 |
| Live Server | 本地调试工具，浏览器实时刷新 |
| Polacode | 把代码保存成美观的图片 |
| Markdown All in One | markdown插件 |
| Chinese(Simplified) Language Pack for Visual Studio Code | 中文简体包 |
| vscode-plugin-swimming | 模拟写代码 |
| AZ AL Dev Tools/AL Code Outline | 用来梳理代码结构的插件，安装完后在文件图标里就会多出一个 AL OUTLINE 的选项。 |
| Postcode | 在 vscode 里面使用 postman。 |
| Project Manager | 项目管理器 |
| javascript console utils | 快速生成 console.log 的插件。选中变量，然后按 ctrl + shift + L 就可以生成了。需要删除的时候按 ctrl + shift + D 即可删除。 |
| 小霸王 | 小霸王 |
| Emoji | 在代码中添加 emoji 表情 |
| Vscode-element-helper | 使用element-ui库的可以安装这个插件，编写标签时自动提示element标签名称。 |
| Version Lens  | 在package.json中显示你下载安装的npm工具包的版本信息，同时会告诉你当前包的最新版本。 |

# 快捷键

![windows 快捷键](https://upload-images.jianshu.io/upload_images/3534846-f60825be9c73e777.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![mac 快捷键](https://upload-images.jianshu.io/upload_images/3534846-f188e4ae8c25f07f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 一次搜索所有文件的文本
```
Windows: Ctrl + Shift + F
Mac: Command + Shift + F
```
2. 集成终端
Windows: Ctrl + \`
Mac: command + \`
通过 Ctrl + `可以打开或关闭终端
3. 删除上一个单词
```
Windows: Ctrl + Backspace
Mac: option + delete
```
这在你打错字的时候非常有用。
4. 重复的行
```
Windows: Shift + Alt + 向下箭头
Mac: command + Shift + 向下箭头
```
复制行。
5. 批量替换当前文件中所有匹配的文本
```
Windows: Ctrl + F2
Mac: command + F2
```
可以选择任何一组文本，如果该选中文本出现多个，一次改所有出现的文本。
6. 向上/向下移动一行
```
Windows: Alt + 向上/下箭头
Mac: command+ 向上/下箭头
```
当前行向上/向下移动。
7. 删除一行
有两种方法可以立即删除一行。
```
Windows: Ctrl + X(Ctrl + Shift + K)
Mac: command + X( command + Shift + K)
```
使用Ctrl + X剪切命令来删除一行。或者使用 Ctrl + Shift + K 命令。

8. 文件操作
Ctrl+N：新建文件

Ctrl+W：关闭文件

Ctrl+Tab：文件切换
9. 格式调整
Ctrl+C/Ctrl+V：复制或剪切当前行/当前选中内容

Alt+Up/Down：向上/下移动一行

Shift+Alt+Up//Down：向上/下复制一行

Ctrl+Delete：删除当前行

Shift+Alt+Left/Right：从光标开始向左/右选择内容
10. 代码编辑
Ctrl+D：选中下一个相同内容

Ctrl+Shift+L：选中所有相同内容

Ctrl+F：查找内容

Ctrl+Shit+F：在整个文件夹中查找内容

# 平滑光标打字
VS Code有一个称为"平滑光标"的功能，它可以在光标移动时产生平滑的动画效果。这让打字感觉更加流畅和精致，并且在我们浏览代码行并将光标放置在不同位置时，给予我们更平滑、更自然的感觉。

{% asset_img 1.gif %}

要打开平滑光标功能，可以在命令面板中打开设置界面，并搜索`Editor: Cursor Smooth Caret Animation`，打开该设置即可。

{% asset_img 2.png %}

# 多光标编辑
多光标编辑功能允许我们在不同的位置同时编辑多个光标所在的文本，从而快速地进行批量操作。例如，我们可以一次性在多行代码中插入同样的内容，或者同时删除多处重复的文本，以加快编辑的效率。

在编辑时，至少会有一个光标，可以使用"Alt + 单击"的方式添加更多的光标。

{% asset_img 3.gif %}

还可以使用`Ctrl + Alt + Down`或`Ctrl + Alt + Up`快捷键轻松地在当前行上方或下方添加光标。

{% asset_img 4.gif %}