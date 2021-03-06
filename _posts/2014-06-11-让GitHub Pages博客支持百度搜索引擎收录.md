---
layout: post
title: "让GitHub Pages博客支持百度搜索引擎收录"
date: 2014-06-11
comments: false
categories: 技巧
---
GitHub Pages搭建的网站，在百度搜索引擎访问的时候，经常性的会返回 403 forbidden（如下图所示），这是我用百度抓取工具抓取的，一直会返回这个错误。我百度了一下，好像是因为百度爬虫爬得太猛烈，已经对很多 Github 用户造成了可用性的问题了，所以Github直接把百度爬虫给屏了。
![](https://dn-zhunjiee.qbox.me/Snip20151028_1.png)

那么如何解决这个问题呢？网友给出的解决方案：

- 总结一下：
	- 换供应商，这个方案不是很靠谱，github 还是很好用的
	- 让 github 改，这个也很难
	- 利用 CDN 加速     √ 这个方案可行！
	


要想百度能正常收录，只有在国外买个VPS自己撘一个博客了。但如果直接用国外的VPS，普通用户访问起来也会很慢。毕竟Github Pages的CDN还是很牛逼的（上面提到只需200ms左右）。那能否让百度爬虫去抓国外VPS上的内容，普通用户直接访问Github Pages呢？

用所谓的智能DNS就能搞定了，用DNSPOD（直接百度）的免费版就有这个功能：
![](https://dn-zhunjiee.qbox.me/Snip20151028_2.png)

以上步骤做完后，再用百度的抓取工具测试，就能正常抓取到内容了。

###更...
如果不嫌弃,将博客转移到国内的GitCafe很轻松的就能被百度爬虫爬到的,博主已经将博客转移到了GitCafe上,而且上传速度明显快了许多.