---
title: ShadowsocksR代理模式
date: 2019/6/22
toc: true
catagories:
- tools
tags:
- shadowsocksr
- 代理
- 科学上网
description: git clone 速度被限制在20KB,明明又开了ssr，结果发现是终端没有开启ssr代理
---

## 代理模式
浏览器一般会自动启用系统代理，而其它软件则需要自身支持HTTP代理或Socks5代理，并且一般不会自动启用系统代理，需要进行手动配置才可以进行代理。

### 直连模式
直接模式会在系统代理设置里关闭代理，使启用系统代理设置的软件（一般为浏览器）直接连接网络。
但是，它并没有关闭在本地构建的代理服务器，其它手动配置代理的软件仍然可以进行连接。

### PAC模式（Proxy auto-config）
PAC模式会在系统代理设置设置一个PAC脚本文件，让系统通过这个文件自动选择每一个连接是否启用代理服务器，也就是判断流量是否进入客户端。
选择PAC模式后可以看到系统设置里面的PAC脚本地址：(该链接的内容就是shadowsocksr文件夹下的`pac.txt`)
![设置-网络和internet-代理-自动设置代理](https://github.com/CynicKimi/images/raw/master/pac.png)

### 全局模式
全局模式会在系统代理设置手动设置一个代理服务器，所有跟随系统代理设置的软件（一般是浏览器）都会使用这个代理服务器。
![设置-网络和internet-代理-手动设置代理](https://github.com/CynicKimi/images/raw/master/overall.png)

### 代理规则
除了上面三种模式，ssr还有一个代理规则：根据IP判断，按设定的规则来判断进入 客户端的流量是直连还是走代理。
顺序：先走三种模式之一 -> 再走代理规则
即当你访问 XXX 网站，然后是全局或者满足PAC条件从而访问 XXX网站的请求数据流量进入了客户端，然后客户端会根据 XXX网站的IP来判断：
- 绕过局域网：当IP属于局域网内时，直连；反之走代理。
- 绕过局域网和大陆：当IP属于大陆内或局域网时，直连；反之走代理。
- 绕过局域网和非大陆：当IP属于大陆外(非大陆IP都算大陆外)或局域网时，直连；反之走代理。

## 设置终端走代理
开启SSR客户端允许其他软件连接：右键->选项设置
![开启shadowsocksr本地代理](https://github.com/CynicKimi/images/raw/master/open_ssr_proxy.png)

### GIT设置代理
```
//设置
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
//取消
git config --global --unset http.proxy
git config --global --unset https.proxy
```
### 终端设置代理
```
//设置
export http_proxy="192.168.1.6:1080"
export https_proxy="192.168.1.6:1080"
//取消
unset http_proxy
unset https_proxy
```

### 连接测试
一般用`curl www.google.com` 来测试终端代理配置是否成功
也可用配置，直接测试：

- 测试HTTP代理访问:`curl -x <proxy_ip>:<proxy_port> www.google.com`
- 测试socks5代理(本地解析域名)：`curl --socks5 <proxy_ip>:<proxy_port> www.google.com`
- 测试socks5代理(代理解析域名)：`curl --socks5-hostname <proxy_ip>:<proxy_port> www.google.com`

#### --socks5和--socks5-hostname的区别
在客户端访问域名的时候，涉及到一个问题，这个域名是应该是客户端解析完告诉代理服务器ip还是应该把域名交给代理服务器去解析？
实验：
我在测试的时候发现使用--socks5去请求google.com是失败的，原因是国内的DNS解析获取到的ip已经被污染了。于是我通过国外的服务器获取到google.com的真实ip，然后在本地设置hosts，结果用--socks5成功的请求到google.com。
结论：当然是交给代理服务器去解析咯。

官方解释（`man curl`）：

>--socks5-hostname <host[:port]>
>	Use the specified SOCKS5 proxy (and let the proxy resolve the host name). If the  port  number  is  not specified, it is assumed at port 1080. (Added in 7.18.0)
>	This option overrides any previous use of -x, --proxy, as they are mutually exclusive.
>	Since 7.21.7, this option is superfluous since you can specify a socks5 hostname proxy with -x, --proxy using a socks5h:// protocol prefix.
>	If this option is used several times, the last one will be used. (This option  was  previously  wrongly documented and used as --socks without the number appended.)

>--socks5 <host[:port]>
>	Use  the  specified  SOCKS5 proxy - but resolve the host name locally. If the port number is not specified, it is assumed at port 1080.
>	This option overrides any previous use of -x, --proxy, as they are mutually exclusive.
>	Since 7.21.7, this option is superfluous since you can specify a socks5 proxy with -x, --proxy using  a socks5:// protocol prefix.
>	If  this  option  is used several times, the last one will be used. (This option was previously wrongly documented and used as --socks without the number appended.)
>	This option (as well as --socks4) does not work with IPV6, FTPS or LDAP.

------
参考资料：

[ Shadowsocks(R)设置：系统代理模式、PAC、代理规则](https://vimcaw.github.io/blog/2018/03/12/Shadowsocks%28R%29%E8%AE%BE%E7%BD%AE%EF%BC%9A%E7%B3%BB%E7%BB%9F%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F%E3%80%81PAC%E3%80%81%E4%BB%A3%E7%90%86%E8%A7%84%E5%88%99)
[Shadowsocks(R)基本原理](https://vimcaw.github.io/blog/2018/03/11/Shadowsocks%28R%29%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86/)
[自己动手开发Socks5代理服务器](https://www.cnblogs.com/cc11001100/p/9949729.html)
[ShadowsocksR中的代理规则是什么?](https://my.sssr.tw/knowledgebase/5/ShadowsocksR.html)

