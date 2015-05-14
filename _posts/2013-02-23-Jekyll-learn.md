---
layout: post
category : tutorial
title: jekyll 学习
tagline: "Supporting tagline"
tags : [jekyll, tutorial]
---

## Jekyll是什么？

Jekyll是一个简单的，适合博客的静态站点生成器。它包含一个模板目录，模板目录里可以包含多种格式的未经处理的文件，Jekyll将这些文件交给Markdown（或者Textile）和Liquid进行处理，并输出完整的，可立即发布的静态页面。

   [Markdown](http://daringfireball.net/projects/markdown/) is a text-to-HTML conversion tool for web writers. 
   
   [Liquid](http://docs.shopify.com/themes/liquid-basics) is the templating engine.


## 一个基本的Jekyll目录结构如下：

```
        .
        ├── _config.yml
        ├── _drafts
        |   ├── begin-with-the-crazy-ideas.textile
        |   └── on-simplicity-in-technology.markdown
        ├── _includes
        |   ├── footer.html
        |   └── header.html
        ├── _layouts
        |   ├── default.html
        |   └── post.html
        ├── _posts
        |   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
        |   └── 2009-04-26-barcamp-boston-4-roundup.textile
        ├── _data
        |   └── members.yml
        ├── _site
        └── index.html
```

目录结构说明：

	1. `_config.yml` Jekyll的配置文件
	2. `_includes` include 文件所在的文件夹。例如：liquid标签`{% include file.ext %}` 用来引用文件 ` _includes/file.ext`
	3. `_layouts` 模版文件夹
	4. `_posts` 自己要发布的内容，post文件的命名规则必须为`YEAR-MONTH-DATE-title.MARKUP`。文章的永久链接可以自定义，但是日期和格式是由文件名唯一决定。
	5. `_sites` 预览时产生的文件都放在该文件夹中
	6. `_drafts` 草稿目录。运行`jekyll serve --drafts`可以看到草稿
	7. `_data` 格式良好的站点会把数据放到这里。Jekyll引擎会自动加载该文件夹下所有的yaml文件（以.yml或.yaml结尾）。如果 _data 文件夹里有一个`members.yml`文件,那么你就可以通过`site.data.members`获取文件内容
	8. `index.html` 如果这个文件带有 YAML Front Matter， 那么它将由Jekyll来处理。同样的道理适用于根目录下或其他的任何目录（除了上面列举的）中任何`.html` `.markdown` `.md` 或 `.textile` 文件
	9. 其他文件和文件夹 其他的任何目录（除了上面列举的）中的文件都会被一字不差的拷贝到_sites目录

## [Front-matter](http://jekyllrb.com/docs/frontmatter/)


    任何带有YAML front matter 块的文件都会经由Jekyll处理。front matter必须放在文件最前面，并且符合YAML格式，还要由三虚线包裹，如下：

        ---
        layout: post
        title: Blogging Like a Hacker
        ---

    在三虚线内，你可以设置已经定义的变量或者定义新的变量。这些变量可以通过liquid标签使用。你可以在当前页面中使用，也可以在当前页面的模板或者引入的文件中使用。
    
    如果你想在当前页面中使用liquid标签和变量，但是又不需要定义任何新变量，你只需把front matter 留空即可。带有空的三虚线的文件仍会被Jekyll处理。（这对CSS和RSS feeds 很有用）


