---
title: Hexo文件夹名称大小写问题
date: 2020-05-20 01:43:49
categories:
	- Other
tags: 
	- Hexo
---

### Hexo文件夹名称大小写问题

因为更换了主题，所以修改了About和Tags文件夹的名称，把首写字母改成了小写，但是因为Git不区分大小写，所以导致修改后，执行如下命令时报错。
``` bash
$ hexo d
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
INFO  Copying files from extend dirs...
fatal: will not add file alias 'About/index.html' ('about/index.html' already exists in index)
FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
Error: Spawn failed
    at ChildProcess.task.on.code (/Users/jamme/Documents/WorkSpace/GitHub/MyBlog/JammeLee.github.blog/node_modules/hexo-deployer-git/node_modules/hexo-util/lib/spawn.js:51:21)
    at ChildProcess.emit (events.js:182:13)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:240:12)
```

随后，做出了如下尝试，虽然不报错了，但是GitHub pages仓库里出现了两个首写字母分别为大写和小写的文件夹：

``` bash
$ cd .deploy_git/ #从blog的工作路径下切入
$ git config --local core.ignorecase false #取消忽略大小写
```

这种情况下可以做如下修改：

``` bash
$ rm -rf about/
$ rm -rf tags/
$ git add -u
$ git commit -m'Remove'
$ git push
```

然后重新切回blog工作路径，执行`hexo clean`和`hexo g`，这个时候再次执行`hexo d`时，就OK了。
注：也可从Finder中直接找到`.deploy_git/.git`文件夹，然后修改其中的`config`文件。