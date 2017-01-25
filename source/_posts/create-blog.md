---
title: 博客扯淡
date: 2016-01-09 22:01:55
categories: personal
tags: 扯淡
---

## 开始(Start)

早在很久以前（2015年），我就已经从[DigitalOcean](https://www.digitalocean.com)购买了虚拟主机（vps），准备搭建自己的个人网站兼博客系统。谁知即将毕业，这个事情便自从开发了一个框架之后就一直搁置了。<!-- more -->直到室友将个人博客搭建完毕，并给我介绍了一个框架[Hexo](https://hexo.io)，我才开始设计自己的网站，[Hexo](https://hexo.io)的一个非常好的地方便是框架完全搭建好了，并且带有许多高手开发的主题可供选择。如果对主题不甚满意，也可以自己修改主题的源代码(并不需要很牛的技术)。[Hexo](https://hexo.io)采用npm进行构建，模板采用[EJS](http://http://www.embeddedjs.com/)，css预处理采用[Stylus](http://stylus-lang.com)，技术很简单大众。于是我便着手构建网站，但苦于毕业出路问题，一直未曾静下心来完成这件事儿，知道上述事情不再困扰我，我便用了几天时间自己搭建出了博客系统。当然我肯定对源代码进行了小小地修改，以满足我的三观需求(虽然并没有什么×用^-^)！好吧，我采用的主题为[Minos](https://github.com/ppoffice/hexo-theme-minos)。以下为具体过程。
   
## 后期(Later)
   
### 配置(Configuration)
博客搭建其实到此已经非常明了，[Hexo](https://hexo.io)官方文档非常齐全，不用担心技术问题。其实主要问题还是主题的文档有些不太全面，不过看看模板文件应该也还可以明白(吧？)。其实还有一个问题，那边是文章里的评论系统。室友一开始使用[多说](http://duoshuo.com/)，可这货不是https，导致博客中具有评论功能的页面提示不安全(强迫症程序猿怎么能够容忍`=‘)，所以我便用了[Disqus](https://disqus.com)，配置只需要在根目录加上disqus_shortname字段为你的short name即可。
### SSL
SSL现在是主流，所以不配一把怎么可以(蛋疼ing)。由于之前的博客https证书有效期只有一年，而且只能单个domain一个证书(毕竟是免费的Orz)，所以寻找了另外一个更加人性化的CA颁发机构[Let's Encrypt](https://letsencrypt.org/)，而且有一个将申请证书的过程自动化：[certbot](https://certbot.eff.org/)。详细的信息网站已经介绍得十分清楚了。
   
## 最后(At last)
   
Have a nice day!