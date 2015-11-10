title: 使用 hexo 创建github个人站点的最佳实践
---
标签： hexo

## 目录
[TOC]
## 一些背景情况
&nbsp;&nbsp;&nbsp;&nbsp;使用github有一段时间了，一直想在github上创建自己的博客。来记录一下自己学习中的心得体会。
&nbsp;&nbsp;&nbsp;&nbsp;我之前的做法是自己在本地编写好html,js然后通过git提交到github。时间长了，感觉非常的繁琐。尤其是写html实在是太麻烦了。现在markdown非常流行，我也非常喜欢markdown简洁高效的编写体验。无意中发现了nodeJs中hexo这个专为个人博客打造的神器。
&nbsp;&nbsp;&nbsp;&nbsp;以下我来记录一下自己在使用hexo过程中的一些感受。

## 环境搭建
&nbsp;&nbsp;&nbsp;&nbsp;首先，需要在计算机上安装git，node，hexo。下面是官网的地址：
> git  : [git官网](http://www.git-scm.com/download/)
> node : [node官网](https://nodejs.org/)

&nbsp;&nbsp;&nbsp;&nbsp;详细的安装过程，请参照 **[hexo官方文档](https://hexo.io/docs/),**这份hexo的官方文档中有非常详细的说明，上面详细的介绍了git和node的安装方法，以及hexo的使用方法。虽然使用的是英文，但是我相信，上过初中的人都能看懂。
&nbsp;&nbsp;&nbsp;&nbsp;具体安装过程，我就不多说了，但是有一点非常的重要，**需要切记：**把git和node配置到环境变量之中。尤其是git,必须配，如果不配后面再使用hexo进行部署的时候会失败。
&nbsp;&nbsp;&nbsp;&nbsp;同时还需要说明一点的就是，最新版的node默认安装了一个node 的包管理工具:npm。这是一个非常强大的神奇，类似于python中的pip。使用npm几乎可以完成绝大多数node下依赖包的安装。如果你不懂，没关系，照着文档做就好了，渐渐的你就明白了。

## 我曾经遇到的困难，我是怎么克服的
&nbsp;&nbsp;&nbsp;&nbsp;如果你认真的完成了第一步，我相信，你基本已经可以自己开发博客了，这里我主要说两点，这两点都是困扰我很久的问题，希望遇到同样问题的朋友少走弯路。

### 怎样高效的书写markdown
&nbsp;&nbsp;&nbsp;&nbsp;书写markdown的工具，如果在windows下我推荐使用**[CmdMarkdown](https://www.zybuluo.com)**,这是一个优秀的markdown编辑器，有在线版本，也有客户端，还自带同步功能。支持即写即看，所以你懂的。

### 部署报错了怎么办
&nbsp;&nbsp;&nbsp;&nbsp;我相信，如果你按照文档中说的走，一般不会报错。应该可以正常进行，现在的你，可能已经熟练的在写博客了。但是如果你向我当初一样，那么粗心就糟了。
#### git环境变量
&nbsp;&nbsp;&nbsp;&nbsp;我之前一直在最后一步使用hexo d 命令部署的时候出错，困扰了我很久，我在网上也找了很久，才解决这个问题，就是因为git环境变量没有配置导致的，所以，请你一定要注意。
&nbsp;&nbsp;&nbsp;&nbsp;git环境变量配置了，部署的时候还是报错。这种情况我也遇到了，这个时候，请你删除目录下.deploy_git文件夹，然后再次运行hexo g命令和hexo d命令。
### 每次编写博客，文件怎么管理
&nbsp;&nbsp;&nbsp;&nbsp;在你的工作目录中有一个文件夹是~\source\_posts文件夹，这个文件夹下面存放的文件是.md的扩展名。这些就是你的markdown格式的博客类型。每次写博客都把markdown文件存放在这个目录下，使用hexo g和hexo d命令之后博客就自动部署完成了，是不是感觉非常的爽？



