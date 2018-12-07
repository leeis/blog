---
title: 基于Hexo和Github Pages搭建个人博客
date: 2017-04-26 16:34:43
comments: true
tags:
- Hexo
categories:
- 工具
toc: true
---
#### 前言
如果要搭建一个自己的个人博客，按照往常的经验，首选肯定是WordPress这种博客平台，基于PHP+MYSQL的黄金组合，买个服务器、买个域名，安装发布。过程繁琐不说，更是要花费一定的金钱。由于本人平时就是Markdown和Github的重度使用者，Github Pages就自然是首选了，至于为什么是Hexo而不是Git自家的Jekyll，这个看个人喜好吧, 因为两者都是基于Markdown的静态博客工具。

Github Pages是Github的一个网站工具，通过在Github中建立名字为**username.github.io**的项目(username就是你在Github上的用户名)，Github就会自动解析你在该项目中的html内容。通过访问**http://username.github.io** 就能访问你的网站了。

Hexo是一个支持Markdown进行写作，并解析成静态html页面，支持一键部署到Github上，与Github Pages无缝衔接。

所以基于Hexo和Github Pages的建站步骤应该是这样子的：

> 1. 申请Github账号(如果没有)
> 2. 配置Github Pages
> 3. 安装Hexo
> 4. 建站
> 5. 配置
> 6. 发布文章

#### Github Pages

Github账号的申请这里就不描述了，不懂的可以百度下，然后进入Github的个人页面，新建仓库(Repository)，名称为**username.github.io**，注意：`username必须和用户名一致`。
![新增仓库][image-1]

是的，到这里就已经可以具备使用能力了，是不是很简单，详细描述可以参考：[Github Pages][1]

#### 安装Hexo

Hexo是基于Node.js，所以安装Hexo之前，需要安装[Node.js][2]，还有就是[Git客户端][3]也是必须的。如果2个都已经具备了，就开始安装Hexo吧，一个命令搞定！

`$ npm install -g hexo-cli`

#### 建站

OK，下面就开始用Hexo建立你的站点目录吧, 具体可参考官网文档：[官方文档][4]

第一步：建立一个自己的站点，比如我的叫ioman

	$ hexo init ioman
	$ cd ioman
	$ npm install

站点结构如下：

	.
	├── _config.yml   # 配置文件，大部分的配置都靠它
	├── package.json  # 应用程序的信息
	├── scaffolds     # 模板信息，创建文章时使用
	├── source              # 资源和Markdown文件存放地方
	|   ├── _drafts       # 草稿，当layout指定为draft时，默认不解析发布
	|   └── _posts        # 准备发布的文章，每次generate会解析该目录下的文章
	└── themes              # 主题， 静态文件会根据主题来生成

#### 配置

建站完成之后，当然是要配置了，这里最重要的就是`_config.yml`文件，可以参考[官方文档][5]，记得要配置Github账号信息, 需要填写`type, repo, branch`三个配置, `type`和`branch`一般是固定的，`repo`请从自己的Github项目上复制。

```
	# Deployment
	## Docs: https://hexo.io/docs/deployment.html
	deploy:
	  type: git
	  repo: https://github.com/leeis/leeis.github.io.git
	  branch: master
```

#### 写文章和部署

好了，现在万事俱备，可以开始写文章了， 一般的操作流程是：

##### 新建文章
命令:`hexo new [layout] <title>`，可缩写为`hexo n`  
这里的`[layout]`就是布局，默认布局配置在`scaffolds`目录下，分别是`draft`,`post`,`page`.

* `draft`是草稿，当执行`hexo n draft hello`时，会在`source/_draft`下新增一个hello.md,在草稿中的文章默认不会被发布到网站上，直到你执行`hexo publish`命令将草稿中的文章发布。
* `post`是默认布局，如果你想立刻开始写文章，可以直接`hexo n hello`，文章会存放在`source/_post`中
* `page`是独立的页面，当执行`hexo n page hello`时， 会在`source/`下生成对应的目录`hello`，该独立的页面需要单独引用，不会出现在文章中，比如可用于生成关于作者的`about`页面。

##### 发布草稿
命令:`hexo publish [layout] <title>`，可缩写为`hexo p`  
publish命令是将\_draft中的文章发布到\_post中

##### 生成静态文件
命令:`hexo generate`，可缩写为`hexo g`  
此命令将md文件解析成静态html文件。可选参数`--watch`，将观察文件的变化，自动解析，不需要再执行`hexo g`命令

##### 部署到Github
命令:`hexo deploy`，可缩写为`hexo d`  
当配置了Git服务器信息之后，执行该命令之后会将`public`目录中的文件推送到远程Git服务器上，这时候再访问`leeis.github.io`就自动更新了，一切就是如此简单。  
`hexo d -g` 可以在部署之前重新生成静态文件，此命令和`hexo g -d`效果一样。

##### 补充命令  
`hexo clean` 用于清除本地静态页面缓存，可以在generate之前clean一下。  
`hexo server` 自带的服务器，可用于本地测试，启动之后，默认端口是4000，可以访问`http://localhost:4000`进行本地预览，`-p`参数可更改端口。`--draft`命令可进行草稿的预览。

#### 更换主题

主题就是指博客的样式，主题内容存在目录`themes`目录下，官方默认的主题是`landscape`，如果需要更换主题，步骤如下：  

1.去[主题页面][6]中选择自己喜欢的主题，可以点击图片进入作者的主页浏览，如果喜欢的话，直接点击标题进入作者的Github。
![官网主题页面][image-2]
![next主题的Github][image-3]

2.从github上下载主题文件到本地，进入到本地站点根目录，执行如下命令，最终将theme文件下载到本地的`themes/next目录`

`git clone https://github.com/iissnan/hexo-theme-next.git themes/next`

3.更改站点配置文件`_config.yml`使主题生效

_config.yml  

	# Extensions
	## Plugins: https://hexo.io/plugins/
	## Themes: https://hexo.io/themes/
	theme: next

> 备注：一般需要根据主题的README.md描述进行相关的配置修改，主要存在主题文件夹的`_config.yml`中。
	
#### Front-matter
	
	Front-matter是指在文章开头部分的配置说明，可以指定文章的分类、标签、是否开启评论功能等等，具体参数可参考[官方Docs](https://hexo.io/docs/front-matter.html#Settings-amp-Their-Default-Values)。
	
> 有些主题对文章没有开启全局目录功能，可以通过`toc: true`内嵌文章的目录结构，这个参数官网没有提及，可以参考。
	
#### Github Pages绑定域名

1. 获取github pages主页的ip地址，比如我的地址是leeis.github.io
	
	![](http://op09okwcw.bkt.clouddn.com/ping_github_ip.jpg)
	
2. 修改域名解析方式将, 将记录值修改为获取到的IP地址
	
	![](http://op09okwcw.bkt.clouddn.com/blog_ali_hosts.jpg)
	
3. 进入`github`的leeis.github.io项目，进入`setting`中进行下图设置，配置自己的域名
	
	![](http://op09okwcw.bkt.clouddn.com/blog_github_setting_domain.jpg)
	
这样等待10分钟左右就能正常解析了。
	
#### Next主题的一些配置
	
##### 1. 打开赞赏功能
	需要在`主题配置文件`中填入**微信**和**支付宝**收款二维码图片地址 即可开启该功能。

```	
reward\_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
wechatpay: /path/to/wechat-reward-image
alipay: /path/to/alipay-reward-image
```
	
##### 2. 设置代码高亮
NexT 默认使用的是 白色的 normal 主题，可选的值有 normal，night， night blue， night bright， night eighties：

```	
更改 highlight_theme 字段:
# Code Highlight theme
# Available value: normal | night | night eighties | night blue | night bright
# https://github.com/chriskempson/tomorrow-theme
highlight\_theme: normal
```
	
##### 3. 订阅微信公众号
在每篇文章的末尾显示微信公众号二维码，扫一扫，轻松订阅博客。
在微信公众号平台下载您的二维码，并将它存放于博客source/uploads/目录下。
然后编辑`主题配置文件`，如下：
	
```	
# Wechat Subscriber
wechat\_subscriber:
  enabled: true
  qcode: /uploads/wechat-qcode.jpg
  description: 欢迎您扫一扫上面的微信公众号，订阅我的博客！
 ```
	
##### 4. 开启动画效果
编辑`主题配置文件`, 搜索 canvas_nest 或 three_waves，根据您的需求设置值为 true 或者 false 即可.

```	
canvas_nest 配置示例:
# canvas\_nest
canvas\_nest: true //开启动画
canvas\_nest: false //关闭动画
	
three_waves 配置示例:
# three\_waves
three\_waves: true //开启动画
three\_waves: false //关闭动画
```
	
#### 补充说明
	
##### Q: 为什么会出现`Cannot GET /tags/`和`Cannot GET /categories/`
	
A: 不管是任何的主题，凡是从菜单中进入`分类`和`标签`的，都是进入单独的page页面，需要我们手工新增。
	
1. 分别执行`hexo new page categories` 和 `hexo new page tags`，新增categories和tags页面
2. 修改/source/categories/index.md，添加`type: "categories"`
3. 修改/source/tags/index.md，添加`type: "tags"`
	
##### Q: 语言如何切换
	
A: 在每个theme中，都会有对应的`language`目录，上面的文件名就是对应的语言，在站点配置文件中我们可以指定语言，即名称要对应，如`next`主题的中文默认名称叫`zh_Hans`，我第一次安装的时候就找不到，因为正常的中文语言一般使用`zh_CN`,所以如果语言切换有问题，记得核对下名字是否匹配。
	
##### Q: 如何自己开发主题
	
A: 一般的，要先了解主题的目录结构和构成，通过node.js的模板语言EJS或者Swig进行开发, 可以通过分析各个主题开发者的代码进行研究。可参考：[写一个自己的Hexo主题](https://segmentfault.com/a/1190000006057336),[从零开始制作 Hexo 主题](http://www.ahonn.me/2016/12/15/create-a-hexo-theme-from-scratch/)
	
##### Q:为什么Next主题中字数统计插件无效
	
A: 需要安装[插件](https://github.com/willin/hexo-wordcount)  
`npm install hexo-wordcount --save`
	
##### Q:如何启用Ftp部署方式
需要安装ftp部署插件，支持[ftpsync](https://hexo.io/docs/deployment.html#FTPSync)和[sftp](https://hexo.io/docs/deployment.html#SFTP)  
	
**1.安装**:  
	FTPSync: `npm install hexo-deployer-ftpsync --save`  
	sftp: `npm install hexo-deployer-sftp --save`
	
**2.修改`_config.yml`配置**:  

```
deploy:
  type: sftp
  host: xx.xx.xx.xx
  user: root
  pass: abcdefg
  remotePath: /
  port: 22
```

**3.重新部署发布**  
`hexo g -d`

#### 资料来源

[Hexo Docs][7]  
[Next主题文档][8]  
[ejs.co][9]
[Hexo小书][10]  

--- 

> 文章来自IOMan的[个人博客][11]

---- 

[1]:	https://pages.github.com/
[2]:	http://nodejs.org/
[3]:	http://git-scm.com/
[4]:	https://hexo.io/zh-cn/docs/setup.html
[5]:	https://hexo.io/zh-cn/docs/configuration.html
[6]:	https://hexo.io/themes/
[7]:	https://hexo.io/docs/
[8]:	http://theme-next.iissnan.com/
[9]:	http://ejs.co/
[10]:	https://www.gitbook.com/book/pengloo53/hexo/details
[11]:	http://ioman.cc

[image-1]:	http://op09okwcw.bkt.clouddn.com/article_show_git_new_repo.jpg
[image-2]:	http://op09okwcw.bkt.clouddn.com/hexo_theme.jpg
[image-3]:	http://op09okwcw.bkt.clouddn.com/theme_github.jpg