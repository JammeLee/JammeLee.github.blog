---
title: Hexo使用命令生成文章
date: 2020-05-20 02:01:12
categories:
	- Other
tags:
	- Hexo
---

### Hexo使用命令生成文章

- hexo new <文章类型>  [文章名称]

``` bash
$ hexo new post post1 #在source/_posts/路径下生成post1.md和post1/
$ hexo new post -p Path/post2 #在source/_post/Path/路径下生成post2.md和post2/
$ hexo new page page1 #在source路径下生成page1/index/和page1/index.md
$ hexo new draft draft1 #在source/_draft下生成test/和test.md
```

即，
``` bash
Usage: hexo new [layout] <title>

Description:
Create a new post.

Arguments:
 layout Post layout. Use post, page, draft or whatever you want.
 title Post title. Wrap it with quotations to escape.

Options:
 -p, --path Post path. Customize the path of the post.
 -r, --replace Replace the current post if existed.
 -s, --slug Post slug. Customize the URL of the post.
```