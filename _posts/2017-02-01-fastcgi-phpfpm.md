---
layout: post
title: "php的运行原理、cgi对比fastcgi以及php-cgi和php-fpm之间的联系区别"
description: "How to Download or Use This Theme"
keywords: "laravel"
---

### CGI：
CGI的英文是（COMMON GATEWAY INTERFACE）公共网关接口，它的作用就是帮助服务器与语言通信，这里就是nginx和php进行通信，因为nginx和php的语言不通，因此需要一个沟通转换的过程，而CGI就是这个沟通的协议。

nginx服务器在接受到浏览器传递过来的数据后，如果请求的是静态的页面或者图片等无需动态处理的则会直接根据请求的url找到其位置然后返回给浏览器，这里无需php参与，但是如果是一个动态的页面请求，这个时候nginx就必须与php通信，这个时候就会需要用到cgi协议，将请求数据转换成php能理解的信息，然后php根据这些信息返回的信息也要通过cgi协议转换成nginx可以理解的信息，最后nginx接到这些信息再返回给浏览器。

**简单来讲，CGI就是Web服务器与应用程序间通信的一个协议**

### fast-cgi：
传统的cgi协议在每次连接请求时，会开启一个进程进行处理，处理完毕会关闭该进程，因此下次连接，又要再次开启一个进程进行处理，因此有多少个连接就有多少个cgi进程，这也就是为什么传统的cgi会显得缓慢的原因，因此过多的进程会消耗资源和内存。

而fast-cgi则是一个进程可以处理多个请求，和上面的cgi协议完全不一样，cgi是一个进程只能处理一个请求，这样就会导致大量的cgi程序，因此会给服务器带来负担

**那为什么又会有fast-cgi呢, 因为每来一个请求CGI都要新开一个进程处理，这样不性能不高，所以才会有fast-cgi**

### php-cgi：
php-cgi是php提供给web serve也就是http前端服务器的cgi协议接口程序，当每次接到http前端服务器的请求都会开启一个php-cgi进程进行处理，而且开启的php-cgi的过程中会先要重载配置，数据结构以及初始化运行环境，如果更新了php配置，那么就需要重启php-cgi才能生效，例如phpstudy就是这种情况。

**php-cgi就是处理cgi与php通信间的php的一端***

### php-fpm:
php-fpm是php提供给web serve也就是http前端服务器的fastcgi协议接口程序，它不会像php-cgi一样每次连接都会重新开启一个进程，处理完请求又关闭这个进程，而是允许一个进程对多个连接进行处理，而不会立即关闭这个进程，而是会接着处理下一个连接。它可以说是php-cgi的一个管理程序，是对php-cgi的改进。

php-fpm会开启多个php-cgi程序，并且php-fpm常驻内存，每次web serve服务器发送连接过来的时候，php-fpm将连接信息分配给下面其中的一个子程序php-cgi进行处理，处理完毕这个php-cgi并不会关闭，而是继续等待下一个连接，这也是fast-cgi加速的原理，但是由于php-fpm是多进程的，而一个php-cgi基本消耗7-25M内存，因此如果连接过多就会导致内存消耗过大，引发一些问题，例如nginx里的502错误。

**同时php-fpm还附带一些其他的功能：**  
例如平滑过渡配置更改，普通的php-cgi在每次更改配置后，需要重新启动才能初始化新的配置，而php-fpm是不需要，php-fpm分将新的连接发送给新的子程序php-cgi，这个时候加载的是新的配置，而原先正在运行的php-cgi还是使用的原先的配置，等到这个连接后下一次连接的时候会使用新的配置初始化，这就是平滑过渡。


最后一言胜千图
![a](/assets/images/fastcgi-phpfpm.png)

https://blog.csdn.net/belen_xue/article/details/65950658
#### Cheers!
