---
title: 1.18 终端使用XX-Net
layout: post
date: 2017-01-25
tags:
  - 其他内容
catalog: true
---
怎么在终端使用XX-Net翻墙这个折腾了半年的东西，终于攻克了。写篇文章纪念一下，也方便在下次有相同需求时查阅

## 安装XX-Net

这个不是什么难事，很方便就能搞定，文档也很齐全。给出链接：

    https://github.com/XX-net/XX-Net/

## 安装privoxy

这个东西是用来在终端进行代理的，安装很简单，各个发行版的仓库里都有。比如openSUSE上可以用下面的命令安装：

    sudo zypper install privoxy

安装完成后，输入以下命令编辑配置文件：

    sudo vim /etc/privoxy/config

在配置文件原有的东西不用管，全是被注释掉了的。在开头增加以下两行即可，简单明了：

    forward / 127.0.0.1:8087      # 所有流量转发到本机8087端口，8087为XX-net的代理端口。如果需要使用其他代理，则可以根据情况更改端口号
    listen-address  0.0.0.0:8118  # 监听所有连接到本机8118端口号上的连接。这句用来设置代理共享，其他设备(手机,平板,pc等)在WiFi代理里设置本机局域网ip，8118端口号即可使用代理

*PS：打开后默认的是教程，详细描述了有哪些设置项，2000多行，惊呆了。。。哪位小伙伴有闲情雅致可以读一读*

## 配置代理

Linux其实默认是支持终端代理的，只要简单的地设置一下环境变量就可以了：

    export https_proxy=127.0.0.1:8087  
    export http_proxy=127.0.0.1:8087

不过并不推荐在终端中运行这两句话进行设置，还是放在bashrc或zshrc中比较好，不然每次都要重新设置。

另外，设置完成后，可以用`export`命令查看一下是否添加成功，该命令会列出所有生效的环境变量

## 检验代理效果

到目前为之，终端其实已经在走XX-Net的流量了，但怎么检验呢？终端输入以下命令，若迅速重新显示提示符，则说明已经成功访问到Google，这意味着什么应该不用多说了：

    curl http://www.google.com

不信？使用以下命令，撤销环境变量的设置，再输入上述命令，提示符应该出不来了：

    unset http_proxy
    unset https_proxy

## 配置证书

虽然说生效了，但还有一个问题。在刚开始为浏览器配置XX-Net时，我们导入了CA证书，而在终端中我们并没有导入。这就导致了诸如Git等使用https的应用无法正常使用。比如在使用`git clone`时会返回（具体地址已用*代替）：

    fatal: unable to access 'https://github.com/*****/*******': SSL certificate problem: unable to get local issuer certificate

针对这个问题该如何解决呢？这就需要用到根证书（Root Certificates）。每个系统各不相同，Google上有很多关于Debian系、RedHat系系统如何添加根证书的内容，但这些都无法用在openSUSE上，因为它们都要求将证书放入`/etc/pki/tls/certs/ca-bundle.crt`，然而openSUSE中根本没有这个文件，甚至没有这个目录！几经辗转，终于找到如下方法。

    openssl x509 -in /path/to/CA.crt -out CA.pem -outform PEM          # 把crt格式的证书转换为pem格式
    sudo mv /path/to/CA.pem /etc/pki/trust/anchors/                    # 移动生成的证书
    sudo update-ca-certificates                                        # 更新，使证书生效

经过上述操作，XX-Net终于能在终端中完美运行。

## 已知问题

尽管可以实现终端的http的https代理，Git所使用的SSH仍无法通过代理，因此若要让Git有明显提速，仍需使用https协议，但这样的话就需要每次输入密码。虽然有[凭证储存](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%87%AD%E8%AF%81%E5%AD%98%E5%82%A8)技术，但却是明文保存的，存在安全问题。目前对此问题还没有较好的办法。

## 参考资料

感谢一下网页的原作者对我的帮助！

- [XX-Net](https://github.com/XX-net/XX-Net/)
- [kali linux 使用privoxy与XX-net代理终端](http://blog.csdn.net/u010430099/article/details/51686058)
- [How to import root CA into system wide trusted store?](https://forums.opensuse.org/showthread.php/445106-How-to-import-root-CA-into-system-wide-trusted-store)
- [How to convert .crt to .pem](https://stackoverflow.com/questions/4691699/how-to-convert-crt-to-pem)
- [linux系统添加根证书 linux证书信任列表](https://my.oschina.net/lemonzone2010/blog/467213)
