---
layout: post
title: "使用Octopress搭建基于Github Pages个人博客的详细过程及原理分析"
date: 2014-10-20 21:55:36 +0800
comments: true
categories: [iOS]
---



前两天图省事，让一位梦想通过博客改变世界的[小伙伴](http://droidyue.com/)帮我搭建了一个基于[Github Pages](https://pages.github.com/)的个人博客，使用的是[Octopress](http://octopress.org/)静态博客引擎。之后博客虽然能跑起来，但面对繁杂的目录结构和分支，我却怎么也理不清其中的原理。之所以说目录结构繁杂，是因为根据`octopress`的[官方文档](http://octopress.org/docs/)描述，我简直在两者之间找不到多少交集(事后证明有些目录和分支确实冗余)。但它却能正常工作，这激起了我强烈的好奇心，索性就把它全删了，并根据官方文档重新搭建了一次。之后又根据`Rakefile`文件内容弄清楚了整个搭建过程和每个`rake`命令的作用。看着Github上清秀的远程仓库目录和知根知底的本地目录结构，顿时心旷神怡，感觉整个世界都敞亮了。所以说，要想搞懂某件事，就把脚伸出去，把鞋子弄湿。说了这么多废话，下面开始进入主题。

PS.以下内容新手可以选择性参考，大神请绕道，谢谢 :]

<!-- more -->


* list element with functor item
{:toc}

## 准备工作

首先强调一下，`Octopress`是一个为hacker们准备的博客框架，你应该对`shell`命令有亲切感，并且对基础的[Git](http://git-scm.com/)知识有所了解，否则Octopress可能不适合你。不过没关系，只要你有足够的热情，上述都是次要的。

开始之前，请确保你已安装了`Git`和`Ruby 1.9.3`及以上版本。如果Ruby版本过低，可以使用[rbenv](https://github.com/sstephenson/rbenv)或[RVM](http://rvm.io/)进行升级。个人推荐使用`RVM`。

## 搭建过程概述

如果上述的Git和Ruby环境都准备好了，顺次执行下述各命令就能很快搭建成功。关于部署到`Github Pages`上时的每个`rake`命令都做了什么，后边会详述。
  
### 设置(Setup)Octopress

`clone`下Github上的[octopress仓库](https://github.com/imathis/octopress)

```
git clone git://github.com/imathis/octopress.git octopress
cd octopress
```

然后安装相关依赖

```
gem install bundler
rbenv rehash    # 如果你使用的是rbenv, rehash 一下以确保能运行 bundle 命令
bundle install
```

安装默认Octopress主题

```
rake install
```

### 部署(Deploy)到Github Pages

[新创建](https://github.com/repositories/new)一个名字格式形如 `username.github.io` 的Github仓库，这里的`username`即你的Github用户名(或组织名)。

配置`Github Pages`

```
rake setup_github_pages
```

该命令会要求你输入上边所新建的Github仓库的`URL`。复制粘贴下你新创建仓库的`SSH`或`HTTPS` URL 即可(比如SSH的URL `git@github.com:username/username.github.io.git`)。

然后生成并部署站点

```
rake generate
rake deploy
```

最后不要忘记`commit`你`octopress框架`的源码(source)到服务器(关于这个`source`的具体含义后边详述)

```
git add .
git commit -m 'your message'
git push origin source
```

OK，如果你按照上述命令一条一条来的话，到此博客已经搭建成功。可以在浏览器输入上述`username.github.io`验证一下。另外你也可以通过[简单的配置](http://octopress.org/docs/deploying/github/#custom_domains)使用自己的独立域名(如果有的话)，这里不再赘述。

### 下面来简单介绍一下怎么新建一篇文章

创建一篇名为`"Hello World"`的文章

```
rake new_post["Hello World"]
```

如果直接`rake new_post`回车的话，会命令行提示输入博客名。

生成文章

```
rake generate   # 在互联网的公开目录生成文章的网页
```

预览新生成的文章

```
rake preview    # 生成blog预览
```

此时在浏览器中输入`http://localhost:4000`可预览刚生成的博客，但`Github Pages`上还看不到。

部署blog

```
rake deploy     # 部署blog到Github Pages
```

OK，一篇名为`"Hello World"`的博客已经发布到了`Github Pages`，可以在浏览器输入上述`username.github.io`验证一下。

到目前为止，关于怎样搭建博客和发布博客已经介绍完毕。看一下你本地的`octopress目录`和Github上的`username.github.io`仓库目录，如果你对他们的结构有所疑问，或者对上述`rake`命令究竟做了什么感到好奇，那么接着往下看。


## 搭建过程详解

在你本地的`octopress目录`下，静躺着一个名为`Rakefile`的文件，他就是今天的主角。该文件是用`Ruby`写的，如果你没接触过Ruby但对`shell`或`Python`较熟悉的话，基本上可以无障碍阅读该文件。

我们来看下`octopress目录`的结构，重点注意一下`_deploy`, `public`和`source`这三个子目录。再看一下你刚建的`username.github.io`仓库，注意下该仓库有两个分支，分别为`master`分支和`source`分支。注意了，这里的`master`分支所对应的本地分支在上述的`_deploy目录`下，而这里的`source`分支所对应的是本地分支即在上述的整个`octopress目录`下。下面我们根据`Rakefile`文件来分析上述各个`rake`命令具体都做了什么，然后我们就知道了这个结果是怎么发生的以及为什么要这么做。

这里所有的路径都是相对`octopress目录`来说的。

### rake install

初始化并配置`octopress`的主题，如果后边没跟主题名参数，则安装默认主题。该命令主要做了如下操作：

1. 首先根据octopress是否已有`source`目录或`sass`目录来判断是否已安装了主题，如果已安装，则询问用户是否覆盖已有主题。如果否，则流程终止(`abort`)。
2. 该命令后边是可以再加一个可选的主题名的，否则会使用默认主题`Classic`。
3. 创建`source`目录，并将`"#{themes_dir}/#{theme}/source/."`目录下的文件拷贝到`source`目录。
4. 创建`sass`目录，并将`"#{themes_dir}/#{theme}/sass/."`目录下的文件拷贝到`sass`目录。
5. 创建`source/_posts`目录和`public`目录。

### rake setup_github_pages

`用户`或`组织`的`Github Pages`使用`master`分支作为`web服务`的公开目录，为你URL为`http://username.github.io`的Pages提供内容文件。因此，你会有这样一个需求，即在`source`分支上做一些与博客(或说`octopress框架`)源码相关的工作，而在`master`分支上`commit`已经生成的博客内容供web访问。该命令主要就是为我们完成上述任务，具体主要做了下述一系列操作。

**NOTE:** 该命令主要用来生成并配置本地`octopress/_deploy`目录。下文中所提到的`Github Pages`仓库即下边首先要新建的这个名字形如`username.github.io`的仓库。

1. 要求输入你`Github Pages`仓库的`URL`。
该`URL`即此步之前你在Github上新建的那个命名格式为`username.github.io`的仓库的URL。之所以以这样的格式命名该仓库，是因为后边要通过该URL提取出该仓库名(即`username.github.io`)，以用来配置成你的Github Pages的域名，即你博客的域名。该URL有`SSH`和`HTTPS`两种格式，都很容易通过字符串截取拿到子串`"username.github.io"`。(Github之前使用`http://username.github.com`作为Github Pages的域名，后来将`.com`改为了`.io`，该`rake`脚本中对此做了兼容处理)。

2. 将指向`imathis/octopress`的远程库的名字由`origin`改为`octopress`。

    **NOTE:** Git在clone一个仓库时会自动将我们从Git服务器上clone下来的仓库命名为origin，并下载其中所有的数据，建立一个指向该仓库的 master 分支的指针，在本地将其命名为 origin/master，但你无法在本地更改其数据。接着，Git 建立一个属于你自己的本地 master 分支，始于 origin 上 master 分支相同的位置，你可以就此开始工作。

	因此，当我们`clone`了octopress的仓库`git://github.com/imathis/octopress.git`后，远程仓库是这样的：

		matrixzk:octopress matrixzk$ git remote -v
		origin     git://github.com/imathis/octopress.git (fetch)
		origin     git://github.com/imathis/octopress.git (push)

	这里为我们将远程仓库名由`origin`修改为`octopress`

		$ git remote rename origin octopress

	之后再查看远程仓库，可以看到`远程库`的名字已经改了

		matrixzk:octopress matrixzk$ git remote -v
		octopress     git://github.com/imathis/octopress.git (fetch)
		octopress     git://github.com/imathis/octopress.git (push)

3. 添加你的`Github Pages`仓库作为当前本地仓库默认的远程仓库`origin`

		$ git remote add origin git@github.com:matrixzk/matrixzk.github.io.git

	然后将远程库中指向之前`origin`(即`imathis/octopress`)的`master`分支的`指针`指向现在的`origin`(即`matrixzk/matrixzk.github.io`)的`master`分支
	
		$ git config branch.master.remote origin
	

	之后再查看远程库，可以看到已经将远程库`origin`指向了我们的`Github Pages`。
	
		matrixzk:octopress matrixzk$ git remote -v
		octopress     git://github.com/imathis/octopress.git (fetch)
		octopress     git://github.com/imathis/octopress.git (push)
		origin     git@github.com:matrixzk/matrixzk.github.io.git (fetch)
		origin     git@github.com:matrixzk/matrixzk.github.io.git (push)
	

4. 将本地master分支的名字由`master`改为`source`
	
		$ git branch -m master source
	

	之后查看本地分支
	
		matrixzk:octopress matrixzk$ git branch
		* source
	

	**NOTE:** 理解这里很重要，关于这里为什么要这样做，要结合下边的6一起理解。前边已经提到，用户或组织的Github Pages使用master分支作为web服务的公开目录，为你URL为`http://username.github.io`的Pages提供网站内容(即内容文件)。而当前的本地`master`分支即整个`octopress框架`的源码所在的分支。其实本地的`master`分支就是一个名字为`master`的`指针`，它现在指向的是整个octopress框架的源码所在的本地分支。我们这里做的其实是把指向整个octopress框架的源码所在的本地分支的指针的名字由`master`改为`source`，把这个`master`指针名字让出来，让它指向后边6所初始化的用于`web访问`的`octopress/_deploy`目录下的本地仓库的主分支。这样，该目录(本地`octopress/_deploy`目录)下的本地`master`分支对应的就是`Github Pages`远程库的`master`分支。

5. 根据前边所提供的Github Pages仓库的URL来配置博客的URL
从前边所提供的SSH或HTTPS类型的URL中截取仓库名`username.github.io`，然后从本地的octopress目录下读取博客的配置文件`_config.yml`，将其`url`参数值改为`http://username.github.io`。

6. 在本地`octopress`目录下新建`_deploy`目录，并对其进行Git初始化，然后添加`Github Pages`仓库作为其远程仓库。

	NOTE:此时已经有了两个本地仓库(这个和前边3提到的整个`octopress`目录(即`octopress框架`)所对应的本地仓库)指向`Github Pages`仓库作为其远程仓库了，分别作为其`master`分支和`source`分支。
		
		$ mkdir _deploy
		$ git init
		$ git add .
		$ git commit -m "Octopress init"
		# 添加Github Pages仓库作为其远程仓库
		$ git remote add origin git@github.com:matrixzk/matrixzk.github.io.git
		

	然后修改`Rakefile`中`deploy_default`和`deploy_brach`两个变量的初始默认值:

		# 代表部署时执行的命令，该'push'为Rakefile中定义的一个rake task
		deploy_default = "push"   # 初始默认值"rsync"
		# This will be configured for you when you run config_deploy
		# 代表部署时执行上述rake task命令’push'时的操作分支
		deploy_branch  = "master" # 初始默认值"gh-pages"


### rake generate

生成`jekyll`站点

```
compass compile --css-dir source/stylesheets
jekyll build
```

### rake preview

对修改后的站点(如新写了一篇文章)生成预览，在浏览器中输入`http://localhost:4000`可看到预览效果。

### rake deploy

将站点部署到服务器，即发布站点到互联网。由于`_deploy`目录所代表的本地仓库的`master`分支对应`Github Pages远程仓库`的`master`分支，该分支目录的内容即`Github Pages`在互联网上供公开访问的站点内容。因此这里做的主要就是将新写的博客文章copy到`_deploy`目录下，然后将此修改`push`到`Github Pages`远程库的`master`分支。

1. 首先查看是否存在预览模式的博客(它们不该被发布)，如果有则删除，并在此重新执行`rake generate`。
2. 将`source`目录下的文件拷贝到`public`目录下。
3. 进入`_deploy`目录，执行`git pull`操作。
4. 将`public`目录的内容拷贝到`_deploy`目录下。
5. 将`_deploy`目录所对应的本地master分支的修改`push`到Github Pages远程库的master分支，即将Github Pages即你的博客部署到了互联网。这里主要做了如下操作。

```
$ cd _deploy
$ git add -A
$ git commit -m "Site updated at #Time.now(即当前时间)"
$ git push origin master  # Pushing generated _deploy website
```

### 生成Github Pages远程库的source分支


搭建过程的最后一步是将你本地`octopress框架`的源码(即本地的`source`分支)`push`到`Github Pages`远程库。注意，此步之前Github Pages远程库还不存在`source`分支。

```
cd octopress
git add .
git commit -m 'your message'
git push origin source # 注意，此时你本地的source分支push到远程库，之后远程库会生成一个source分支
```

**NOTE:** 这里所做的是将本地`octopress`目录下的本地`source`分支(前边已将该默认分支名由`master`改为`source`)`push`到Github Pages远程库，这样，`Github Pages`远程库就生成了`source`分支。至此，`Github Pages`远程库有了两个分支，即`master`分支和`source`分支。这里的`master`分支所对应的本地分支为本地的`octopress/_deploy`目录下本地仓库的`master`分支，主要存放部署完毕生成的供互联网访问的`Github Pages`站点(即你的博客站点)的内容。`source`分支所对应的本地分支为本地的整个`octopress`目录下本地仓库的`source`分支(即该本地库的主分支，之前将其名字有`master`改为了`source`，缘由前边已详述)，主要存放整个`octopress框架`源码的内容。


### rake new_post["Hello World"]

新建一篇博客。这里以新建一篇名为`"Hello World"`的blog为例。

1. 判断是否提供了文章title，如果没给的话提示输入title。

2. 在`source/_posts`目录下创建名字为`"2014-10-17-hello-world.markdown"`的文件，即你新建的博客文章。新建博客的命名规则为`YYYY-MM-DD-post-title.markdown`。
	
		filename = "#{source_dir}/#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
	

	**NOTE:** 如果相同文件名的文件已经存在，会询问你是否覆盖原文件，如果否，则流程终止(`abort`)。这里所说相同文件名指的是自动生成的时间也相同。这个时间的作用，一是用来区分相同名字(`post title`)的文章，二是用来为决定博客文章列表的顺序。

3. 在新建的`"2014-10-17-hello-world.markdown"`文件开头写入如下内容
		
		---
		layout: post
		title: "hello world"
		date: 2014-10-17 19:59
		comments: true
		external-url:
		categories:
		---
		

	这段[yaml front matter](http://jekyllrb.com/docs/frontmatter/)主要是告诉`jekyll`你新建博客的一些生成规则。这里可以让你关掉评论功能，或者给你新建的文章添加分类。如果你在与多个作者同时维护一个博客，添加`author: Your Name` 到上述脚本中可以在适当的位置显示你的名字。如果你在写一篇草稿，添加`published: false`可以防止在你生成(`generate`)博客时把这篇草稿发布出去。如果你要发布一篇链接博客(`linklog`)，即点击标题后会跳转到另一个站点的某篇文章，只需将`external-url`参数值设为指定站点文章的`url`即可，例如：

		external-url: http://opinionguy.com/post/uninformed-rant-vs-straw-man/
	

	按如下方式可以给你新建文章添加一或多个类别(`category`)。
	
		# 一个category
		categories: Sass
		# 多个categories的例子1
		categories: [CSS3, Sass, Media Queries]
		# 多个categories的例子2
		categories:
		- CSS3
		- Sass
		- Media Queries
		

---


好了，到此为止已经将使用`octopress`搭建基于`Github Pages`个人博客的整个过程和边边角角都介绍了。有些命令和术语可能理解得不够准确，欢迎指正。
