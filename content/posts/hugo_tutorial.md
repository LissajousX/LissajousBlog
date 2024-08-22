---
date: 2024-08-22T17:50:00+08:00
description: "如何利用hugo和github部署个人博客"
featured_image: "/images/hugo.png"
images: ["/images/hugo.png"]
tags: ["tutorial", "hugo"]
categories: "Tutorials"
title: "Hugo + Github 部署个人博客"
---

# 安装前置

前置软件包括：

- Git
- Hugo

# 在本地部署Hugo

先验证hugo是否安装成功：

```bash
hugo version
```

然后正式开始，建议新建一个文件夹作为博客的工作区。在这个文件夹下新建项目：

```bash
hugo new site <blog>
```

这样就建立了一个文件夹。进入这个文件夹：

```bash
cd <blog>
```

然后初始化当前目录中的空 Git 存储库：

```bash
git init
```

下一步就是安装主题，在<https://themes.gohugo.io/> 里可以找一个你喜欢的，在主题的页面里一般都可以找到安装方法，一般格式是：

```bash
git submodule add <https> themes/<themename>
```

然后一般能在`<themename>`文件夹内找到`exampleSite`文件夹，把里面的文件复制到`<blog>`文件夹里。

注意！现在的`<blog>`文件夹里的默认配置文件是`hugo.toml`，而一般的`exampleSite`里的配置文件一般是`config.toml`和`config.yaml`，而一个网站只能有一个配置文件，且这三种文件名都是可以的。所以复制的时候记得把原来的`hugo.toml`配置文件删掉。

然后就可以开始本地部署了：

```bash
hugo server
```

从返回的信息里可以看到本地网址：`http://localhost:<port>/`，复制到浏览器里，就可以看到这个主题的示例了。

# 把Hugo部署到github上

这一部分是`本教程`与`其他所有中文教程`都不同地方，其他的教程都不知道是哪个时代的方法了，废话少说，正式开始。

先创建新存储库。只需要填上`Repository name`那一栏，格式为`<name>.github.io`。如果你想让你的博客网址就是`.github.io`，则`<name>`不能是任意名字，必须是你的github用户名。

其他都不需要动，点击`Create repository`就行了。

然后返回`<blog>`文件夹里，把这个本地库连接到远端库：

```bash
git remote add origin git@github.com:<name>.github.io.git
```

把这些修改后的文件添加到本地库，并标记上“提交信息”：

```bash
git add .
git commit -m "commit message"
```

"commit message"填啥都可以，这个是用来标记这次更新的内容。

把“主干”设置为“main”：

```bash
git branch -M main
```

最后把本地库的内容推送到远端库：

```bash
git push -u origin main
```

下一步是进入<https://github.com/<name>/<name>.github.io>，点击上方栏的`Settings`，然后点击左方栏的`Pages`，在`Build and deployment`里的`Source`中选择`Github Actions`，在下面找到`Hugo`，点击`Configure`，在新界面点击右侧的绿色按钮的`Commit changes...`。

找不到`Hugo`就去`browse all workflows`里找。

然后hugo就开始在github上部署了，部署好后就可以在`Pages`页面看到`Your site is live at ...`，点击`Visit site`就可以访问博客网站了。

# 后续写博客

后续更新只需要在`content`文件夹加入md文件，然后执行

```bash
git add .
git commit -m "commit message"
git push -u origin main
```

这三句指令就可以了。

# 参考链接 
[在github.io用hugo部署个人博客，2023新教程](https://zhuanlan.zhihu.com/p/649542248)