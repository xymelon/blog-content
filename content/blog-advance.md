Title: 使用Pelican和GitHub Pages搭建个人博客 —— 进阶篇
Date: 2013-11-22 08:00
Modified: 2013-11-27 12:11
Category: Technology
Tags: pelican
Slug: blog-advance
Author: xymelon

###独立域名
1. 在[godaddy](http://www.godaddy.com/)上购买域名，推荐使用国内优惠码（搜索一下，你就知道~），价格会非常便宜，比如我花了111.95元就购买了两年期限的域名，你说爽不爽。
2. 购买成功后，需要将DNS服务解析转回到国内，推荐使用[DNSPod](https://www.dnspod.cn/)，至于为啥需要转回国内，你猜呢？DNSPod官网上有详细介绍，请点击[Godaddy注册商域名修改DNS地址](https://support.dnspod.cn/Kb/showarticle/?qtype=%E5%8A%9F%E8%83%BD%E4%BB%8B%E7%BB%8D%E5%8F%8A%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B&tsid=42)。
3. 完成第二步后，需注册[DNSPod](https://www.dnspod.cn/)，请点击[学会使用DNSPod，仅需三步](https://support.dnspod.cn/Kb/showarticle/tsid/177/)，按照教程添加上刚购买的域名后，为让其指向github分配给我们的二级域名，只需在DNSPod中添加A记录，指向204.232.175.78，请参见github pages教程[Setting up a custom domain with Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-pages)，如有不懂之处，也可看看DNSPod的帮助中心，里面的介绍非常详细。
4. 最后一步，登录github，进入我们所创建的cowfighting.github.io库中，在根目录创建一个名为CNAME的文件，内容为在godaddy购买的域名，例www.xycoding.com，到此为止，即大功告成。
<!-- PELICAN_END_SUMMARY -->

###URL配置
一个好的URL地址不光搜索引擎友好，也让自己看得舒心，pelican中也完全考虑到这一点，只需简单的配置就可打造出漂亮的URL。

打开`pelicanconf.py`配置文件，按照如下修改或添加，具体配置请参见[官方文档](http://docs.getpelican.com/en/3.3.0/settings.html)。

	ARTICLE_URL = 'pages/{date:%Y}/{date:%m}/{date:%d}/{slug}/'
	ARTICLE_SAVE_AS = 'pages/{date:%Y}/{date:%m}/{date:%d}/{slug}/index.html'

###pelican插件
插件有很多种，比如带来便利性、观赏性、易搜索性等，大家可根据需要进行选择。本博客中选用了`sitemap`,`summary`,`neighbors`等插件。首先，和主题一样，选择一个文件夹，去[pelican插件开源库](https://github.com/getpelican/pelican-plugins)clone下来插件，选择自己喜欢的~所有插件的功能都可在开源库中找到答案。

	//clone插件
	git clone https://github.com/getpelican/pelican-plugins

打开`pelicanconf.py`配置文件，按照如下修改或添加，插件的使用方法请进入相应的插件开源库进行了解。
	
	PLUGIN_PATH = 'pelican-plugins' 	//设置路径
	PLUGINS = ['summary','sitemap','neighbors'] //选用插件

	SITEMAP = {
	    'format': 'xml',
	    'priorities': {
	        'articles': 0.7,
	        'indexes': 0.5,
	        'pages': 0.5
	    },
	    'changefreqs': {
	        'articles': 'monthly',
	        'indexes': 'daily',
	        'pages': 'monthly'
	    }
	}

###Google Analytics和Google Webmasters
注册[Google Analytics](http://www.google.com/analytics/)和[Google Webmasters](http://www.google.com/webmasters/)可以更好的管理自己的站点，更好的让google收录等。具体教程非常简单，注册完基本就能知道~最后记得在`pelicanconf.py`中进行配置。

	GOOGLE_ANALYTICS = 'Tracking ID'


###最后
博客到此为此基本介绍完毕，以后想起什么再补充吧。其实我对现在这个主题还有不满意的地方，比如`key words，description`等没有地方设置，这对搜索引擎不友好；还有左边栏没有热门文章等~接下来再慢慢完善。

----------

博客源码地址：[https://github.com/xymelon/xymelon.github.io](https://github.com/xymelon/xymelon.github.io)

博客主题地址：[https://github.com/xymelon/xycoding-gum](https://github.com/xymelon/xycoding-gum)


