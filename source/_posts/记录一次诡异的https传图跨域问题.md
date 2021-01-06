---
title: 记录一次诡异的https传图跨域问题
date: 2020-12-17 16:41:04
tags:
---

公司有个需求，将windows的传图接口改造到centos（都为阿里云EC）， 基本配置如下：

>CentOS Linux release 7.8.2003 (Core)
>
>Server version: Apache/2.4.46 (codeit)
>
>PHP 7.3.25 (cli) (built: Nov 24 2020 11:10:55) ( NTS )



有个奇怪的问题是，功能代码在windows下http,https 传图都没问题，在centos下http传图没有问题，https传图会偶尔出现跨域提醒![错误提醒](/images/error.jpg), 结合网络的答案各种折腾，php header重新设置、设置阿里云OSS跨域、apache升级等等，甚至还想改造代码，设置代理，更换apache为nginx等。不过这样的话动作太大，也没有准确定位到问题，没有动。



对于这种偶发性的问题，一开始觉得apache的配置应该是没有问题的，可能出在其他的地方，不过测试了几种方案后感觉还是应该修改配置，毕竟是和ssl证书相关，我是把VirtualHost 单独配置的，想着如果windows配置没有问题，那会不会某些默认配置没有启用导致，想到就把VirtualHost 配置合并，之后就没在出现传图失败了。



再查看配置找寻原因，注意到ssl.conf默认配置中有这样的内容

```
#   SSL Protocol Adjustments:
#   The safe and default but still SSL/TLS standard compliant shutdown
#   approach is that mod_ssl sends the close notify alert but doesn't wait for
#   the close notify alert from client. When you need a different shutdown
#   approach you can use one of the following variables:
#   o ssl-unclean-shutdown:
#     This forces an unclean shutdown when the connection is closed, i.e. no
#     SSL close notify alert is send or allowed to received.  This violates
#     the SSL/TLS standard but is needed for some brain-dead browsers. Use
#     this when you receive I/O errors because of the standard approach where
#     mod_ssl sends the close notify alert.
#   o ssl-accurate-shutdown:
#     This forces an accurate shutdown when the connection is closed, i.e. a
#     SSL close notify alert is send and mod_ssl waits for the close notify
#     alert of the client. This is 100% SSL/TLS standard compliant, but in
#     practice often causes hanging connections with brain-dead browsers. Use
#     this only for browsers where you know that their SSL implementation
#     works correctly. 
#   Notice: Most problems of broken clients are also related to the HTTP
#   keep-alive facility, so you usually additionally want to disable
#   keep-alive for those clients, too. Use variable "nokeepalive" for this.
#   Similarly, one has to force some clients to use HTTP/1.0 to workaround
#   their broken HTTP/1.1 implementation. Use variables "downgrade-1.0" and
#   "force-response-1.0" for this.
BrowserMatch "MSIE [2-5]" \
         nokeepalive ssl-unclean-shutdown \
         downgrade-1.0 force-response-1.0
```

大意就是mod_ssl会关闭与浏览器的连接，保持连接的一种设置（实在是没有Google到合适的中文解释   - -!）。之后测试了上百张图片也没有再复现问题，就暂时告一段落。