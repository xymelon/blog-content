Title: 使用Pelican和GitHub Pages搭建个人博客 —— 基础篇
Date: 2013-11-21 11:21
Modified: 2013-11-27 12:11
Category: Technology
Tags: pelican
Slug: blog-create
Author: xymelon

###前言
一直以来都希望拥有属于自己的个人博客，随性发点信息，写点技术感想，记录自己的生活，重要的是不受广告的影响、不被河蟹、不会担心有一天被莫名其妙地消失。作为一名技术渣硕，抛弃拿来主义，勇于摸索，踏出第一步，不再纸上谈兵，从现在开始。

本博客是在windows下搭建完成。不过这和linux下是类似的，因为搭建过程是在git bash中进行，git bash的命令风格是仿linux的。
<!-- PELICAN_END_SUMMARY -->

###知识储备
搭建博客的工具选用了基于python的pelican，相比wordpress等其它工具来说，它比较轻盈并有很多令人兴奋的[特性](http://docs.getpelican.com/en/3.3.0/#features)，再配合免费无限制的github pages，近乎完美。搭建过程中涉及如下技术知识，不过你不必害怕，只是使用它们的开源框架而已，并不需要自己编码，点击可以了解它们是如何的强大，当然你也可以略过它们，后面遇到时再进行了解。<br/>
假如你不能打开它们，原因你懂的，请爬墙解决~

> [github](https://github.com/)
> 
> [github pages](http://pages.github.com/)
> 
> [git](http://git-scm.com/)
> 
> [python](http://www.python.org/)
> 
> [pip](https://pypi.python.org/pypi/pip)
> 
> [pelican](http://blog.getpelican.com/)
> 
> [markdown](http://daringfireball.net/projects/markdown/syntax)
> 
> [markdownpad](http://markdownpad.com/)

###下载安装
请点击下载以下内容，本文会一步一步介绍如何安装，不着急，慢慢来。

> [python3下载](http://www.python.org/getit/)
> 
> [git下载](http://msysgit.github.io/) 
> 
> [markdownpad下载](http://markdownpad.com/)
> 
> [window下make下载](http://www.equation.com/servlet/equation.cmd?fa=make) 

1. 安装python，这个就不用多说了吧~
2. 安装git，简单。git bash使用教程请参见[官方文档](http://www.git-scm.com/book/zh)~ 
3. 将python安装文件夹中scripts和make.exe加入环境变量。
4. 安装pip（PS：Python 2 >=2.7.9 或 Python 3 >=3.4安装python时已包含），具体可以看[官网介绍](http://www.pip-installer.org/en/latest/installing.html)，或者[Windows环境在python3装pip](http://tech.crandom.com/2011/10/21/python3_pip.html)。
5. 安装pelican和markdown，至于为什么使用markdown语法，这就看个人喜好呢，下载的markdownpad就是window下markdown的编辑器呢，非常简单，linux等其它操作系统编辑器请自行google。你也可以选用REST语法，具体请参见[markdown教程](http://daringfireball.net/projects/markdown/syntax)和[REST教程](https://beinggeekbook.readthedocs.org/en/latest/rst.html)。

		pip install pelican
		pip install markdown

###主体搭建
打开git bash，进入一个自己喜欢的文件夹（注意文件夹名最好不为空，因为后续make html命令会出错），执行以下命令

	mkdir blog //创建文件夹，名称可根据自己喜欢定
	cd blog
	pelican-quickstart
pelican-quickstart执行命令后，会提示输入博客的配置项，除了少数几个必填以外，其它都可以选择默认，而且都可以在`pelicanconf.py`文件中进行更改，所以你可以随意选择，如下图~

![pelican-quickstart](/images/pelican-quickstart.png)

命令成功执行后，会出现pelican的框架，如下所示

    blog/
    ├── content                # 存放输入的markdown或RST源文件
    │   └── (pages)            # 存放手工创建的静态页面，可选
    ├── output                 # 存放最终生成的静态博客
    ├── develop_server.sh      # 测试服务器
    ├── Makefile               # 管理博客的Makefile
    ├── pelicanconf.py         # 配置文件
    └── publishconf.py         # 发布文件，可删除
	
	
###书写博文
完成上述博客主体搭建后，使用markdownpad创建一个`.md`文件，保存于`content`文件夹中。博文格式如下所示

![markdown](/images/markdown.png)

图中左侧是我们书写的markdown格式的源文件，右边则是即使预览效果，很棒吧~
至于源文件顶部`Title,Date,Category等`内容则是必须的，具体可参见[文档](http://docs.getpelican.com/en/3.3.0/getting_started.html#writing-content-using-pelican)，它们各自意义如下

	Title: 文章标题
	Date: 创建日期
	Modified: 修改日期
	Category: 文章分类，标志本文处于该分类下
	Tags: 文章标签，标志本文处于该标签下
	Slug: URL中该文章的链接地址
	Author: 作者


写完后，回到`blog`目录下，执行`make html`命令进行博客的生成
	
	make html
	(pelican e:/blog/content/ -o e:/blog/output -s e:/blog/pelicanconfg.py)

`make html`命令将把刚才写的博文生成html，存放到output目录下，如果你没有make命令，也可执行第二行的pelican命令。接着执行`make serve`开启测试服务器

	make serve
	(cd e:/blog/output/ && python -m pelican.server)

`make serve`命令也可由第二行替代，在浏览器中输入`http://localhost:8000`即可看到博文效果。


###主题选择
回到`blog`目录下，按如下步骤下载pelican官方主题，从里面挑选出自己喜欢的主题吧，大多数主题预览界面你可以打开这个[网页](http://pelicanthemes.com/)进行查看。不过如今pelican又新出了很多主题，所以你需看看[pelican主题开源库](https://github.com/getpelican/pelican-themes)。

	git clone https://github.com/getpelican/pelican-themes.git

打开`pelicanconf.py`配置文件，更改或添加`THEME`为自己喜欢的主题，例如本博客所挑选的gum，更多的配置含义请关注[官方文档](http://docs.getpelican.com/en/3.3.0/settings.html)。
	
	THEME = 'pelican-themes/gum'


###添加评论系统
开启个人博客的原因在于分享知识，分享就需要交流，评论模块当然少不了。在[Disqus](https://disqus.com/)上申请帐号，按照流程Disqus会分配给你站点的Shortname，记牢Shortname，如果忘了请进入admin/settings中查看。然后同理，在`pelicanconf.py`添加

	DISQUS_SITENAME = Shortname


###博文发布
经过以上的折腾，离成功只差最后一步呢。写好的博文总要找个站点发布吧，本文基于github pages，当然你也可以选择你熟悉的站点服务器。具体步骤如[官方教程](https://pages.github.com/)，非常简单。按照官方教程，你就拥有一个二级域名和一个[版本库](https://github.com/xymelon/xymelon.github.io)，比如我的[http://xymelon.github.io/](http://xymelon.github.io/)，当然现在访问会出现跳转，这在后面独立域名章节会介绍。

进入上一章节所示的`output`目录下，依次执行以下命令

	git init
	git add .
	git remote add origin https://github.com/xymelon/xymelon.github.io
	git pull origin master
	git commit -m 'first blog'
	git push origin master

熟悉git的同学肯定知道上述命令表达的是什么意思吧，如果你还不甚了解，没关系，先照做或者看看git的[官方文档](http://www.git-scm.com/book/zh)。上述命令很长，如果你比较熟悉了，可以修改Makefile文件进行一键上传~


最后打开浏览器输入github pages的二级域名，效果如下

![article](/images/article.png)

目前为止，还算比较成功。从上图可以看出，书写源文件的`Title,Date,Category,Slug等`内容每个都有其的用武之处，具体你可以慢慢了解。

###TODO
目前来说，博客的基本使用已经可以了，不过我们依然可以对其进行完善。接下来的一篇[文章](http://www.xycoding.com/articles/2013/11/22/blog-advance/)我会介绍以下几个方面

1. 独立域名
2. 博文URL格式配置，例[http://www.xycoding.com/pages/2013/11/21/create-blog/](http://www.xycoding.com/pages/2013/11/21/create-blog/)
1. pelican插件的使用（Sitemap等）
2. Google Analytics和Google Webmasters


###感想
这是我第一次比较认真的写教程，参考了网上很多的例子，不过都觉得他们讲得不够详细，让我花了很多时间去看官方介绍，当然这可能是我太渣的缘故，看不懂大神们的教程，哈哈。不过从中我也学到了很多心得，比如不懂的地方第一选择是官方文档，本文基本给出了搭建博客过程的所有链接。

最后，不管怎样，属于自己个人的博客已经搭建起来，从现在开始，记录自己的生活吧~

----------

博客源码地址：[https://github.com/xymelon/xymelon.github.io](https://github.com/xymelon/xymelon.github.io)

博客主题地址：[https://github.com/xymelon/xycoding-gum](https://github.com/xymelon/xycoding-gum)


