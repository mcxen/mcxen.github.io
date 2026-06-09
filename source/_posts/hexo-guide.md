---
title: hexo 搭建指南
date: 2024-04-07 10:00:00
tags:
  - Hexo
  - 博客
---

HEXO部署参考：https://zz2summer.github.io/github-hexo-%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/

环境准备
- node.js
- git

这两个应用windows用户直接搜索下载安装就可以。
如果习惯了使用linux命令的朋友，推荐windows神器`cmder`。
可以直接在windows环境下使用linux命令，样式可调，再也不要用黑乎乎的cmd了，而且自带git，完全可以不用下载windows git。

正式安装hexohexo官方中文文档

在node.js安装好的前提下，全局安装hexo
如何判断node.js是否安装成功？执行以下命令，如果能够看到版本号则说明安装成功了

```
node -v
```

安装`hexo`

```
npm install -g hexo-cli
```

npm安装卡住

执行

npm config set registry https://registry.npmmirror.com/

完成之后如果下载还是有问题，可以执行 npm set strict-ssl false， 然后就可以了npm i了

自选合适的目录，新建文件夹<folder>

```
cd <folder>
hexo init
npm install
```

不再赘述，直接看官方文档。

配置github新建仓库，仓库名必须为**[your_name.github.io]**

> 补充：本地配置github ssh连接，方便自动部署，以及clone你喜欢的主题(theme)
windows用户直接在`c:/用户/youername/.ssh/`下查看是否有`id_rsa.pub`文件。
没有的话命令行执行命令`ssh-keygen -t rsa -C "your eamil"`，会自动生成`id_rsa.pub`文件，打开后复制。

github->头像->Settings→SSH kyes→Add SSH key，粘贴复制的内容。

配置本地账户

```
git config --global user.name “your_username” #设置用户名
git config --global user.email “your_email” #设置邮箱地址,最好使用注册邮箱地址
```

测试是否配置成功

```
ssh -T git@github.com
```

hexo配置以及使用有两个配置文件：

- 一个是根目录下的`_config.yml`称为`站点配置`文件
- 一个是`themes/landscape/_config.yml`称为`主题配置`文件(默认主题：landscape)

站点配置如下：

```
url: https://yourname.github.io/
theme: landscape #选择你想用的主题，我用的是indigo
deploy:
    type: git # 不要使用github
    repo: git@github.com:pengwenwu/pengwenwu.github.io.git # 使用ssh连接
    branch: master # 默认master分支
    message: add new blog # 自动部署commit备注，可不填
```

Hexo切换主题```
npm install hexo-theme-icarus
hexo config theme icarus  #切换主题
```

Hexo快速启动文章hexo s

写文章、发布文章 首先在博客根目录下安装一个扩展npm i hexo-deployer-git。 

然后输入hexo new post “article title”，新建一篇文章。

 然后打开H:\blog\source_posts的目录，可以发现下面多了一个article-title.md文件，就是文章文件。 

编写完markdown文件后，根目录下输入hexo g生成静态网页，然后输入hexo s可以本地预览效果，

最后输入hexo d上传到github上。这时打开你的github.io主页就能看到发布的文章啦。

hexo常用命令hexo命令参考

`hexo n "我的博客"` &#x3D;&#x3D; `hexo new "我的博客"` #新建文章
`hexo p` &#x3D;&#x3D; `hexo publish`
`hexo g` &#x3D;&#x3D; `hexo generate` #生成
`hexo s` &#x3D;&#x3D; `hexo server` #启动服务本地预览
`hexo d` &#x3D;&#x3D; `hexo deploy` #部署
`hexo clean` #清除缓存 网页正常情况下可以忽略此条命令  

`hexo server` #Hexo 会监视文件变动并自动更新，您无须重启服务器。
`hexo server -s` #静态模式
`hexo server -p 5000` #更改端口
`hexo server -i 192.168.1.1` #自定义 IP

在执行之前，记得安装自动部署 (–save 加不加的区别在于是否写入到依赖文件package.json中)

```
npm install hexo-deployer-git --save
```

正常本地预览，直接执行`hexo s`,如果要发布话最好执行`clean`命令，会去删除生成的public文件，完整部署命令:`hexo clean && hexo g && hexo d`。或者直接`hexo d -g`

问题描述：
- 先是出现错误：
`error：spawn failed...`
- 然后经过一些博客的操作会出现以下问题：
`fatal: cannot lock ref &#39;HEAD&#39;: unable to resolve reference HEAD: Invalid argument error: src refspec`
- 或者：
`error: src refspec HEAD does not match any.`等等

总结一下：
问题大多是因为`git `进行`push`或者`hexo d`的时候改变了一些`.deploy_git`文件下的内容。

解决办法：
- 删除`.deploy_git`文件夹;
- 输入`git config --global core.autocrlf false`
- 然后，依次执行：
`hexo clean`
`hexo g`
`hexo d`

                        &larr; Previous Post

                    HEXO部署环境准备正式安装hexo配置githubhexo配置以及使用Hexo切换主题Hexo快速启动文章hexo常用命令问题描述：解决办法：