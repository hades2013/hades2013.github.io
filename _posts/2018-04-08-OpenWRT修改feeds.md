---
layout:     post
title:      OpenWRT 修改feeds.conf.default为GitHub源
subtitle:   OpenWRt feeds update
date:       2018-04-07
author:     hades
header-img: img/my-blog-picture.jpg
catalog: true
tags:
    - hades
---

# OpenWRT 修改feeds.conf.default为GitHub源

lede和openwrt合并之后
lede官网挂了。。
git.openwrt.org，也访问不了。。

只好去github上找最新源码： 

	git clone https://github.com/openwrt/openwrt.git

最新的lede

	git clone -b lede-17.01 https://github.com/openwrt/openwrt.git

但是问题又来了，，
`./script/feeds update -a ` 又挂了，，

查看文件`feeds.conf.default` ：
```
src-git packages https://git.openwrt.org/feed/packages.git
src-git luci https://git.openwrt.org/project/luci.git
src-git routing https://git.openwrt.org/feed/routing.git
src-git telephony https://git.openwrt.org/feed/telephony.git
#src-git video https://github.com/openwrt/video.git
#src-git targets https://github.com/openwrt/targets.git
#src-git management https://github.com/openwrt-management/packages.git
#src-git oldpackages http://git.openwrt.org/packages.git
#src-link custom /usr/src/openwrt/custom-feed
```
	
git.openwrt.org访问不了，无法更新，只能使用github源地址： 
更新如下： 
```
src-git packages https://github.com/openwrt/packages.git
src-git luci https://github.com/openwrt/luci.git
src-git routing https://github.com/openwrt-routing/packages.git
src-git telephony https://github.com/openwrt/telephony.git
src-git management https://github.com/openwrt-management/packages.git
```

OK，`./script/feeds update -a `  可以了



