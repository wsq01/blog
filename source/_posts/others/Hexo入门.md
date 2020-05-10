---
title: Hexo入门
date: 2016-11-22 19:32:16
tags: 其他
---

# 什么是 Hexo？
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
# 安装
安装 Hexo 需要先安装下列应用程序：
- Node.js
- Git

## 安装 Hexo
所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。
```
$ npm install -g hexo-cli
```
安装 Hexo 完成后，执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。
```
$ hexo init <folder>
$ cd <folder>
$ npm install
```
新建完成后，指定文件夹的目录如下：
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
### _config.yml
网站的配置信息。
### scaffolds
模板文件夹。新建文章时，Hexo 会根据`scaffold`来建立文件。

Hexo的模板是指在新建的文章文件中默认填充的内容。例如，如果修改`scaffold/post.md`中的`Front-matter内`容，那么每次新建一篇文章时都会包含这个修改。
### source
资源文件夹是存放用户资源的地方。除`_posts`文件夹之外，开头命名为`_`(下划线)的文件/文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到`public`文件夹，而其他文件会被拷贝过去。
### themes
主题文件夹。Hexo 会根据主题来生成静态页面。
# 配置
可以在`_config.yml`中修改大部分的配置。
## 网站

| 参数 | 描述 |
| :--: | :--: |
| title | 网站标题 |
| subtitle | 网站副标题 |
| description |	网站描述，主要用于SEO |
| keywords | 网站的关键词。使用半角逗号, 分隔多个关键词。 |
| author | 您的名字 |
| language | 网站使用的语言 |
| timezone | 网站时区。默认使用电脑的时区。 |

## 网址
|参数	| 描述	| 默认值 |
| :--: | :--: | :--: |
| url |	网址	| |
| root | 网站根目录	| |
| permalink | 文章的永久链接格式 | :year/:month/:day/:title/|
| permalink_defaults | 永久链接中各部分的默认值 | |
| pretty_urls | 改写 permalink 的值来美化 URL	| |
| pretty_urls.trailing_index | 是否在永久链接中保留尾部的 index.html，设置为 false 时去除 | true |
| pretty_urls.trailing_html | 是否在永久链接中保留尾部的 .html, 设置为 false 时去除 (对尾部的 index.html无效)	| true |

如果您的网站存放在子目录中，例如`http://yoursite.com/blog`，则请将您的 url 设为`http://yoursite.com/blog`并把 root 设为 /blog/。

例如：
```
# 比如，一个页面的永久链接是 http://example.com/foo/bar/index.html
pretty_urls:
  trailing_index: false
# 此时页面的永久链接会变为 http://example.com/foo/bar/
```

## 目录

|参数	| 描述 |	默认值|
| :--: | :--: | :--: |
| source_dir| 资源文件夹，这个文件夹用来存放内容。|	source|
|public_dir| 公共文件夹，这个文件夹用于存放生成的站点文件。| public |
|tag_dir | 标签文件夹 | tags |
|archive_dir |	归档文件夹| archives |
|category_dir	| 分类文件夹 | categories |
|code_dir | Include code 文件夹，source_dir 下的子目录|	downloads/code |
|i18n_dir | 国际化（i18n）文件夹|	:lang |
|skip_render | 跳过指定文件的渲染。匹配到的文件将会被不做改动地复制到 public 目录中。您可使用 glob 表达式来匹配路径。| |

例如：
```
skip_render: "mypage/**/*"
# 将会直接将 `source/mypage/index.html` 和 `source/mypage/code.js` 不做改动地输出到 'public' 目录
# 你也可以用这种方法来跳过对指定文章文件的渲染
skip_render: "_posts/test-post.md"
# 这将会忽略对 'test-post.md' 的渲染
```
## 文章

| 参数 |	描述	|默认值 |
| :--: | :--: | :--: |
| new_post_name	| 新文章的文件名称 | :title.md |
| default_layout | 预设布局|	post |
| auto_spacing |	在中文和英文之间加入空格|	false |
| titlecase |	把标题转换为 title case |	false |
| external_link |	在新标签中打开链接 |	true |
| external_link.enable |	在新标签中打开链接|	true |
| external_link.field | 对整个网站（site）生效或仅对文章（post）生效 |	site |
| external_link.exclude |	需要排除的域名。主域名和子域名如 www 需分别配置 |	[] |
| filename_case | 把文件名称转换为 (1) 小写或 (2) 大写 |	0 |
| render_drafts | 显示草稿 | false |
| post_asset_folder |	启动 Asset 文件夹	| false |
| relative_link	| 把链接改为与根目录的相对位址 |	false |
| future|	显示未来的文章 |	true |
| highlight	| 代码块的设置 | |
| highlight.enable |	开启代码块高亮 | true |
| highlight.auto_detect | 如果未指定语言，则启用自动检测 |	false |
| highlight.line_number	| 显示行数 | true |
| highlight.tab_replace|	用 n 个空格替换 tabs；如果值为空，则不会替换 tabs |	'' |
| highlight.wrap | Wrap the code block in \< table \> |	true |
| highlight.hljs|	Use the hljs-* prefix for CSS classes |	false |

## 相对地址
默认情况下，Hexo 生成的超链接都是绝对地址。例如，如果您的网站域名为`example.com`，您有一篇文章名为`hello`，那么绝对链接可能像这样：`http://example.com/hello.html`，它是绝对于域名的。相对链接像这样：`/hello.html`，也就是说，无论用什么域名访问该站点，都没有关系，这在进行反向代理时可能用到。通常情况下，建议使用绝对地址。

## 分类 & 标签

| 参数 | 描述 |	默认值 |
| :--: | :--: | :--: |
| default_category |	默认分类 |	uncategorized |
| category_map |	分类别名	| |
| tag_map |	标签别名 | |

## 日期 / 时间格式
Hexo使用 Moment.js 来解析和显示时间

| 参数 | 描述 |	默认值 |
| :--: | :--: | :--: |
| date_format |	日期格式 | YYYY-MM-DD |
| time_format |	时间格式 | HH:mm:ss |
| use_date_for_updated |	启用以后，如果 Front Matter 中没有指定 updated， post.updated 将会使用 date 的值而不是文件的创建时间。在 Git 工作流中这个选项会很有用	| true |

## 分页

| 参数	| 描述 | 默认值 |
| :--: | :--: | :--: |
| per_page | 每页显示的文章量 (0 = 关闭分页功能)	| 10 |
| pagination_dir | 分页目录 |	page |

## 扩展

| 参数 | 描述 |
| :--: | :--: |
| theme | 当前主题名称。值为false时禁用主题 |
| theme_config | 主题的配置文件。在这里放置的配置会覆盖主题目录下的 _config.yml 中的配置 |
| deploy | 部署部分的设置 |
| meta_generator | Meta generator 标签。 值为 false 时 Hexo 不会在头部插入该标签 |

## 包括或不包括目录和文件
在 Hexo 配置文件中，通过设置`include/exclude`可以让 Hexo 进行处理或忽略某些目录和文件夹。你可以使用 glob 表达式 对目录和文件进行匹配。

| 参数 | 描述 |
| :--: | :--: |
| include	| Hexo 默认会忽略隐藏文件和文件夹（包括名称以下划线和 . 开头的文件和文件夹，Hexo 的 _posts 和 _data 等目录除外）。通过设置此字段将使 Hexo 处理他们并将它们复制到 source 目录下。 |
| exclude | Hexo 会忽略这些文件和目录 |
| ignore | Ignore files/folders |

举例：
```
# Include/Exclude Files/Folders
include:
  - ".nojekyll"
  # 包括 'source/css/_typing.css'
  - "css/_typing.css"
  # 包括 'source/_css/' 中的任何文件，但不包括子目录及其其中的文件。
  - "_css/*"
  # 包含 'source/_css/' 中的任何文件和子目录下的任何文件
  - "_css/**/*"

exclude:
  # 不包括 'source/js/test.js'
  - "js/test.js"
  # 不包括 'source/js/' 中的文件、但包括子目录下的所有目录和文件
  - "js/*"
  # 不包括 'source/js/' 中的文件和子目录下的任何文件
  - "js/**/*"
  # 不包括 'source/js/' 目录下的所有文件名以 'test' 开头的文件，但包括其它文件和子目录下的单文件
  - "js/test*"
  # 不包括 'source/js/' 及其子目录中任何以 'test' 开头的文件
  - "js/**/test*"
  # 不要用 exclude 来忽略 'source/_posts/' 中的文件。你应该使用 'skip_render'，或者在要忽略的文件的文件名之前加一个下划线 '_'
  # 在这里配置一个 - "_posts/hello-world.md" 是没有用的。

ignore:
  # Ignore any folder named 'foo'.
  - "**/foo"
  # Ignore 'foo' folder in 'themes/' only.
  - "**/themes/*/foo"
  # Same as above, but applies to every subfolders of 'themes/'.
  - "**/themes/**/foo"
  ```
列表中的每一项都必须用单引号或双引号包裹起来。

include 和 exclude 并不适用于 themes/ 目录下的文件。如果需要忽略 themes/ 目录下的部分文件或文件夹，可以使用 ignore 或在文件名之前添加下划线 _。

# 使用代替配置文件
可以在 hexo-cli 中使用 --config 参数来指定自定义配置文件的路径。你可以使用一个 YAML 或 JSON 文件的路径，也可以使用逗号分隔（无空格）的多个 YAML 或 JSON 文件的路径。例如：
```
# use 'custom.yml' in place of '_config.yml'
$ hexo server --config custom.yml

# use 'custom.yml' & 'custom2.json', prioritizing 'custom3.yml', then 'custom2.json'
$ hexo generate --config custom.yml,custom2.json,custom3.yml
```
当你指定了多个配置文件以后，Hexo 会按顺序将这部分配置文件合并成一个 _multiconfig.yml。如果遇到重复的配置，排在后面的文件的配置会覆盖排在前面的文件的配置。这个原则适用于任意数量、任意深度的 YAML 和 JSON 文件。

例如，使用 --options 指定了两个自定义配置文件：
```
$ hexo generate --config custom.yml,custom2.json
```
如果 custom.yml 中指定了 foo: bar，在 custom2.json 中指定了 "foo": "dinosaur"，那么在 _multiconfig.yml 中你会得到 foo: dinosaur。

# 覆盖主题配置
通常情况下，Hexo 主题是一个独立的项目，并拥有一个独立的 _config.yml 配置文件。
你可以在站点的 _config.yml 配置文件中配置你的主题，这样你就不需要 fork 一份主题并维护主题独立的配置文件。

以下是一个覆盖主题配置的例子：
```
# _config.yml
theme_config:
  bio: "My awesome bio"
# themes/my-theme/_config.yml
bio: "Some generic bio"
logo: "a-cool-image.png"
```
最终主题配置的输出是：
```
{
  bio: "My awesome bio",
  logo: "a-cool-image.png"
}
```
# 指令
```
// 新建一个网站。如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。
$ hexo init [folder]
// 新建一篇文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。
$ hexo new [layout] <title>
// 如果标题包含空格的话，请使用引号括起来。
$ hexo new "post title with whitespace"
```
默认情况下，Hexo 会使用文章的标题来决定文章文件的路径。对于独立页面来说，Hexo 会创建一个以标题为名字的目录，并在目录中放置一个 index.md 文件。你可以使用 --path 参数来覆盖上述行为、自行决定文件的目录：
```
hexo new page --path about/me "About me"
```
以上命令会创建一个`source/about/me.md`文件，同时 Front Matter 中的 title 为 "About me"

注意！title 是必须指定的！如果你这么做并不能达到你的目的：
```
hexo new page --path about/me
```
此时 Hexo 会创建`source/_posts/about/me.md`，同时 me.md 的 Front Matter 中的 title 为 "page"。这是因为在上述命令中，hexo-cli 将 page 视为指定文章的标题、并采用默认的 layout。

```
// 生成静态文件。
$ hexo generate
```

| 选项 | 描述 |
| :--: | :--: |
| -d, --deploy | 文件生成后立即部署网站 |
| -w, --watch | 监视文件变动 |
| -b, --bail | 生成过程中如果发生任何未处理的异常则抛出异常 |
| -f, --force | 强制重新生成文件。Hexo 引入了差分机制，如果 public 目录存在，那么 hexo g 只会重新生成改动的文件。
使用该参数的效果接近 hexo clean && hexo generate |
| -c, --concurrency | 最大同时生成文件的数量，默认无限制 |

该命令可以简写为
```
$ hexo g
```
```
// 发表草稿
$ hexo publish [layout] <filename>
```
```
$ hexo server // 启动服务器，默认情况下，访问网址为： http://localhost:4000/。
```

| 选项 | 描述 |
| :--: | :--: |
| -p, --port | 重设端口 |
| -s, --static | 只使用静态文件 |
| -l, --log | 启动日记记录，使用覆盖记录格式 |

```
$ hexo deploy // 部署网站。
// 该命令可以简写为：
$ hexo d
```

| 参数 | 描述 |
| :--: | :--: |
| -g, --generate | 部署之前预先生成静态文件 |

```
$ hexo render <file1> [file2] ... // 渲染文件。
```

| 参数 | 描述 |
| :--: | :--: |
| -o, --output | 设置输出路径 |

```
$ hexo clean // 清除缓存文件 (db.json) 和已生成的静态文件 (public)。
// 在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。

$ hexo list <type> // 列出网站资料。
$ hexo version // 显示 Hexo 版本。
```
