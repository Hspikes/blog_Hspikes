+++
date = '2025-08-22T10:33:00+08:00'
draft = false
title = '关于 git 的那些事'
katex = true
+++

# 关于 git 的那些事

## git config

在编写搭建个人博客网站时曾遇到这样一个问题：将图片文件存储在本地时没有任何问题，可以正常打开，网站也可以正常渲染，然而当通过 git 上传到 github 之后，部署到 github page 后图片却无法正常渲染。经过原因调查发现**在 github 仓库中对应的图片文件无法正常打开，下载后 Linux 环境也无法正常打开**。

进一步查阅发现是 git config 配置的问题 ~~(虽然问题解决后也没能成功渲染)~~ ，由于以前没有详细研究关于 git 的设置，一怒之下查询了相关的资料整理了一下。

### Git 配置综述

#### 层级结构

Git 的配置有层级之分，共有三层，类似局部变量与全局变量的关系，更高的层级有更低的优先级：

1. 系统级（--system）：对整个操作系统上的所有用户生效。配置文件通常在 /etc/gitconfig。
2. 全局级（--global）：对当前用户生效。配置文件在 ~/.gitconfig 或 Windows 的 C:\Users\<用户名>\\.gitconfig。
3. 仓库级（默认）：只对当前 Git 仓库生效。配置文件在仓库的 .git/config。

#### 语法规则

这些配置项的语法规则类似结构体遵循```<section>.<key>```。其中 section 时区块名，比如 core, user, branch...


#### 查看设置

这些配置文件默认状态下均是隐藏的，在 Linux 下以位于 . 开头的 .git 中，ls 带参数 -a 即可查看。

对于一个特定项目，可以运行```git config --list --show-origin```命令，一是显示当前项目的所有配置，二是显示这些配置的来源文件。

需要查看某一项具体配置，直接运行命令```git config ***```即可。

比如本博客项目的仓库 git 配置如下：

```php
[core]
	repositoryformatversion = 0
	filemode = true               // git 会跟踪文件的可执行权限
	bare = false                  // 是否为裸仓库，即没有工作区，常用于远程仓库
	logallrefupdates = true       // git 会为所有引用保留日志，在 .git/logs/ 中
[remote "origin"]
	url = git@github.com:Hspikes/blog_Hspikes.git
	fetch = +refs/heads/*:refs/remotes/origin/*  // 在 fetch 时将远程仓库所有分支都同步到本地
[branch "main"]
	remote = origin
	merge = refs/heads/main
```

全局配置只有在你第一次设置了```--global```配置后才会自动生成。至于全局的配置如下：

```php
[user]
	email = ***@***
	name = Hspikes
[core]
	editor = vim
	autocrlf = false
```

可以看到全局配置基本上只约束了用户名邮箱地址、信息默认编辑器之类的，至于那个 ```autocrlf``` 我们下文会讲到。

本项目 git 没有系统级别的配置，通常系统级均是关于服务器多人协作、企业模版等场景下运用，个人的电脑上往往都不会进行系统级设置。

### 常见配置

这里的常见配置自然不包括 username, email... 一类的易懂的配置，而是总结一些较难理解却容易出错的配置。

#### core.autocrlf

显然根据区块名称这属于核心配置，auto 自动处理某些问题，crlf 代表**换行相关的处理**。控制的是 **Git 在提交和检出文件时，是否自动转换换行符**。

一个人尽皆知的问题是**不同系统的换行符不一样**，Windows → CRLF (\r\n), Linux/macOS → LF (\n)。Git 为了避免“同一文件在不同系统上显示换行不一致”，提供了这个自动转换机制。

这个设置共有三种模式:

1. true: 提交时：Git 会把 CRLF → LF(保证仓库里都是 LF)；检出时：Git 会把 LF → CRLF。
2. input: 提交时：Git 会把 CRLF → LF；检出时：不做转换(保持仓库里的 LF)。
3. false: Git 不做任何转换，提交和检出都保持原样。

我所遇到的问题便是 autocrlf 设置为 true 并且 git 提交时**错误的把本属于二进制文件的图片文件 png 识别为了文本文件**，导致图片文件内容被损坏。

最合适的做法是在 .gitattributes 中声明哪些是二进制文件，比如：

```php
*.png binary
*.jpg binary
*.pdf binary
```

我这里由于手上的开发环境全都是 linux，直接偷了懒，更改了全局设置，等以后出问题时再来清算吧😋

#### pull.rebase

pull 的默认方式是 fetch + merge，若 pull.rebase = true，则默认方式为 fetch + rebase。前者会保留分支的分流历史，后者会合并两者而不保留分流历史。

## 不完全仓库

这里不完全仓库有两种情况：

1. 一是提交的仓库不完全：你可能并不希望把你的数据、测试痕迹与日志之类的文件提交。
2. 二是本地的仓库不完全：一个非常大的仓库你可能只需要其中的一小部分。

### 提交内容不完全

这种情况比较简单，直接用 .gitignore 就可以了。但一个常见的情况是在建立仓库时并没用想好到底有哪些需要 ignore，添加了一些文件**并且 commit**之后发现有些文件其实不应该提交。

这时可以利用 ```git rm --cache``` 删除响应的模块内容，基本语法和 ```rm``` 是一样的，也就是说要 ```rm``` 文件夹需要加上 ```-r``` 的选项。比如移除对数据的跟踪 ```git rm --cache -r ./data```，配合 .gitignore 就可以完成停止对 /data 的追踪。

### 本地仓库不完全

工作中一个复杂的工程，你可能只希望保留你需要更改的一小部分。分为两种情况：一种是一个已有的远程仓库，你 clone 其中一部分；另一种是你的一个个人仓库，维护的过程中越来越大，你需要去除本地的一部分。

#### 拉取远程仓库

先说 clone 这种情况，git 提供了一个名为**稀疏检出**的工具，其通过一份允许路径列表来筛选检出时的保留路径，其他路径不写或从工作区移除。

要注意：

1. sparse-checkout 不会删除远程仓库内容；
2. sparse-checkout 不等价于从历史里删掉对象（它主要管工作区）；
3. 随时可以把某目录加回来，Git 再把它检出到工作区。

稀疏检出有两种模式 cone 模式与非 cone 模式。

非 cone 模式创建一个文件 ```.git/info/sparse-checkout``` 类似 .gitignore，匹配路径、文件，支持通配*，来确认筛选规则。比如：

```text
/*
!/src/
!/docs/
!/README.md
```

其中的 ! 是不包含某目录。

core 模式相对简单，只想按目录为单位保留/排除，简单并且性能更为优良。只需要在仓库中执行：

```bash
git sparse-checkout init --cone
git sparse-checkout set src docs
```

便可以让工作区只出现 ```src/, docs/``` 以及必要的父目录结构。当后续希望添加检出内容的时候可以 ```git sparse-checkout add test``` 增加检出内容。

对于 clone 时需要参数 ```--filter=blob:none --no-checkout``` 确保克隆时尽量不下载 blob（文件内容），只先拿 commit/tree 结构；当你真的要检出/查看某个文件时，再按需从远程取该 blob。类似于文件系统的思想，文件实体和索引是相互分离的。

一次标准的拉取流程：

```bash
git clone --filter=blob:none --no-checkout <url> repo
cd repo
git sparse-checkout init --cone
git sparse-checkout set lab1 docs
git checkout main
```

#### 本地仓库减负

这种情况如果想要达到实际意义上删除某些磁盘文件，释放磁盘内存其实基本不可能，因为 git 的提交历史中引用了某些文件，那么 git 为了追踪就会保存相应的文件在 .git/objects 中，想要真正的删除它们只能篡改历史。

如果只是减轻工作区负担，可以单纯的利用稀疏检出：

```bash
git sparse-checkout init --cone
git sparse-checkout set lab1 docs   
```

但不会真正意义上删除文件，想要删除文件最为简单的方法就是重新拉取远程仓库。