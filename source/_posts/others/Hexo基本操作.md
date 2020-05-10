---
title: Hexo基本操作
date: 2016-11-23 17:42:51
tags: 其他
---

你可以执行下列命令来创建一篇新文章或者新的页面。
```
$ hexo new [layout] <title>
```
可以在命令中指定文章的布局（layout），默认为 post，可以通过修改 _config.yml 中的 default_layout 参数来指定默认布局。

# 布局（Layout）
Hexo 有三种默认布局：post、page 和 draft。在创建者三种不同类型的文件时，它们将会被保存到不同的路径；而您自定义的其他布局和 post 相同，都将储存到 source/_posts 文件夹。

| 布局 | 路径 |
| :--: | :--: |
| post | source/_posts |
| page | source |
| draft | source/_drafts |

如果你不想你的文章被处理，你可以将 Front-Matter 中的layout: 设为 false 。
# 文件名称
Hexo 默认以标题做为文件名称，但您可编辑`new_post_name`参数来改变默认的文件名称，举例来说，设为`:year-:month-:day-:title.md`可让您更方便的通过日期来管理文章。

| 变量 | 描述 |
| :--: | :--: |
| `:title` | 标题（小写，空格将会被替换为短杠） |
| `:year` | 建立的年份，比如， 2015 |
| `:month` | 建立的月份（有前导零），比如，04 |
| `:i_month` | 建立的月份（无前导零），比如，4 |
| `:day` | 建立的日期（有前导零），比如，07 |
| `:i_day` | 建立的日期（无前导零），比如，7 |

# 草稿
刚刚提到了 Hexo 的一种特殊布局：draft，这种布局在建立时会被保存到 source/_drafts 文件夹，您可通过 publish 命令将草稿移动到 source/_posts 文件夹，该命令的使用方式与 new 十分类似，您也可在命令中指定 layout 来指定布局。
```
$ hexo publish [layout] <title>
```
草稿默认不会显示在页面中，您可在执行时加上 --draft 参数，或是把 render_drafts 参数设为 true 来预览草稿。
# 模板（Scaffold）
在新建文章时，Hexo 会根据 scaffolds 文件夹内相对应的文件来建立文件，例如：
```
$ hexo new photo "My Gallery"
```
在执行这行指令时，Hexo 会尝试在 scaffolds 文件夹中寻找 photo.md，并根据其内容建立文章，以下是您可以在模版中使用的变量：

| 变量 | 描述 |
| :--: | :--: |
| layout | 布局 |
| title | 标题 |
| date | 文件建立日期 |

# Front-matter
Front-matter 是文件最上方以 --- 分隔的区域，用于指定个别文件的变量，举例来说：
```
---
title: Hello World
date: 2013/7/13 20:46:25
---
```
以下是预先定义的参数，您可在模板中使用这些参数值并加以利用。

| 参数 | 描述 | 默认值 |
| :--: | :--: | :--: |
| layout | 布局	|
| title | 标题 | 文章的文件名 |
| date | 建立日期 | 文件建立日期 |
| updated | 更新日期 | 文件更新日期 |
| comments | 开启文章的评论功能 | true |
| tags | 标签（不适用于分页）| |
| categories | 分类（不适用于分页） |	
| permalink | 覆盖文章网址 |
| keywords | 仅用于 meta 标签和 Open Graph 的关键词（不推荐使用）	||

分类和标签
只有文章支持分类和标签，您可以在 Front-matter 中设置。在其他系统中，分类和标签听起来很接近，但是在 Hexo 中两者有着明显的差别：分类具有顺序性和层次性，也就是说 Foo, Bar 不等于 Bar, Foo；而标签没有顺序和层次。
```
categories:
- Diary
tags:
- PS3
- Games
```
分类方法的分歧
如果您有过使用 WordPress 的经验，就很容易误解 Hexo 的分类方式。WordPress 支持对一篇文章设置多个分类，而且这些分类可以是同级的，也可以是父子分类。但是 Hexo 不支持指定多个同级分类。下面的指定方法：
```
categories:
  - Diary
  - Life
```
会使分类Life成为Diary的子分类，而不是并列分类。因此，有必要为您的文章选择尽可能准确的分类。

如果你需要为文章添加多个分类，可以尝试以下 list 中的方法。
```
categories:
- [Diary, PlayStation]
- [Diary, Games]
- [Life]
```
此时这篇文章同时包括三个分类： PlayStation 和 Games 分别都是父分类 Diary 的子分类，同时 Life 是一个没有子分类的分类。
# 生成文件
使用 Hexo 生成静态文件快速而且简单。
```
$ hexo generate // 监视文件变动
```
Hexo 能够监视文件变动并立即重新生成静态文件，在生成时会比对文件的 SHA1 checksum，只有变动的文件才会写入。
```
$ hexo generate --watch // 完成后部署
```
您可执行下列的其中一个命令，让 Hexo 在生成完毕后自动部署网站，两个命令的作用是相同的。
```
$ hexo generate --deploy
$ hexo deploy --generate

// 上面两个命令可以简写为
$ hexo g -d
$ hexo d -g
```