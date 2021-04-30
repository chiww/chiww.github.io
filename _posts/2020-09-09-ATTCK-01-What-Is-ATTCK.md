---
layout: post
title:  "ATT&CK系列-MITRE ATT&CK基本概念"
date:   2020-09-09 22:28:28 +0800
categories: MITRE ATT&CK 
tags: 网络安全 反入侵
---

* content
{:toc}



## 0. 引言

MITRE ATT&CK 框架是最近安全圈很火的概念，从这篇文章开始，将开一个新的系列（keng），记录点ATT&CK学习记录。



在介绍ATTCK之前，作为一个甲方安全从业者，请大家思考几个问题：

1. 怎么回答老板对你的灵魂拷问：
- 我们现在的安全做的怎样了，能防住哪些攻击，与业界差距是多少？
- 为什么今年的预算又要采购这些设备，去年不是刚刚买了XX吗，怎么又买？
- 为什么买了那么多安全设备，还会出这样那样的问题？
2. 如果要搭建一套SIEM或者SOC系统：
- 需要采集哪些数据？
- 做了哪些规则，可以检测哪些场景，这些规则场景到底可以在我们的环境中适用吗？
- 怎么验证规则场景有效？
3. 如果要自建威胁情报
- 怎么收集威胁情报，有哪些内容可以收集？

....



那么多问题，归根到底就是如何去评价组织内安全现状，到底哪里做的好，哪里做的不好，好的话好在哪里，不好的话又不好在哪里，以及如何逐步去推动安全防御建设。

有人可能就会说了，我们部署了主机防护，也有WAF，也有下一代防火墙，可以检测webshell、反连马，有流量分析检测DNS隧道，可以检测恶意域名，还有全部修复高危漏洞、没有高危端口... 等等这些列举现有的安全检测设备的功能和场景。

这就有一个问题，你怎么保证你所列的这些检测防护场景是覆盖所有已知的攻击手法呢？等等，已知的”攻击手法“又有多少？如果不是全覆盖，又是覆盖多少呢？通常有经验的安全从业者，会直接告诉你，列出的这些检测方法场景就是经验得出的最佳实践。（当然，如果基于”经验“的理由来向老板拿来年的安全经费，如果老板能接受，还是挺好的）

的确是这样，可靠的安全经验是很重要的，这些经验可以根据组织现状制定出最佳的安全建设方案。但经验是很个人的东西，每个人安全人员都明白自己的最终目标是保护组织内的信息资产，但出于安全经验不同、对安全的理解也不同，就会产生不同的安全实践，不同的实践也会产生不同的安全效果。

那有没有可能存在一种方法论可以很好的集合所有安全从业者的经验，形成一套通用的安全知识框架呢？

MITRE ATT&CK 就是一个很好的工具，可以解决上述的问题。




## 1. MITRE ATT&CK是什么

MITRE是一个面向美国政府提供系统工程、研究开发和信息技术支持的非盈利性组织。我们所熟知的CVE、CWE、CVSS等安全标准规范都是出自该组织之手。

ATT&CK是由MITRE创建并维护的一个对抗战术和技术的知识库，全称 Adversarial Tactics, Techniques, and Common Knowledge, 简称ATT&CK。这个知识库是由社区驱动的，并且是公开免费、全球可访问的知识库。


ATT＆CK是针对网络攻击行为的精选知识库和模型，反映了攻击者攻击生命周期以及各个攻击阶段的目标，由以下核心组件组成:
- 战术（Tactics）：表示攻击过程中的短期战术目标；
- 技术（Techniques）：描述对手实现战术目标的手段；
- 子技巧（Sub-Techniques）：描述对手在比技术更低的级别上实现战术目标的技术手段；

```
备注：在ATT&CK中，“对手”一般也可以理解为“攻击者”，此“攻击者”可能并非真正攻击者，也有可能是红蓝对抗演练中的“攻击方”；本文中，如无特殊说明，这两者含义无区别。
```

ATT＆CK有三个核心概念，用通俗一点话来讲就是：
- 攻击视角；
- 基于观察；
- 合适的抽象级别能够将攻击行为和防御策略结合；

### 1.1 攻击视角

ATT&CK与其他从防御角度的模型（例如，侧重于漏洞评分的CVSS，侧重于风险计算的DREAD）不同，ATT&CK在其术语、模型策略和技术都使用了攻击者视角进行描述。防御视角的检测告警缺乏告警的上下文，导致这些告警很难被深入分析，因此ATT＆CK使用攻击视角比从纯粹的防御角度更容易理解上下文中的行动和潜在对策。

视角的转变就将问题从”基于可用资源列表发生的事情“转变为”在攻防结合的框架中可能发生的事情“，某种程度上为如何评估防御范围提供了更准确的参考框架。

ATT&CK参考框架与防御工具或具体的数据收集方法无关，防御者只要遵循框架中的攻击者单个行动的动机，就可以了解这些行动与环境中特定防御类别之间的关系。

### 1.2 基于观察

ATT&CK所描述的攻击活动很大部分取材于公开的事件，大部分事件是涉及APT组织所为，而这些公开的事件成为了ATT&CK这个知识库的基础。在这个知识库上，可以更准确描述“在野”的技术或可能发生的攻击活动。

ATT&CK也会吸收来自企业内网的攻击报告以及攻防演练的技术发现的技术（例如，可以颠覆常用防御的技术）。

ATT&CK是基于真实事件观察而得出的模型框架，是组织可能会遇到的真正威胁的攻击技术，因此这些技术并不是“由于使用困难”或“使用率太低”而不太能看见的理论技术。

ATTCK观察源主要有以下几类：
- 公开消息
- 社区贡献
- 未公开事件

另外，ATT&CK观察目标是攻击者的攻击手法、工具以及行为模式，也叫”战术、技术、过程（Tactics, Techniques, and Procedures ，TTPs）”，简称TTPs。

提到TTPs，就一定绕不过“痛苦金字塔（Pyramid Of Pain）”，痛苦金字塔描述攻守过程的失陷指标（IOCs）,TTPs 处于痛苦金字塔塔尖， 如下图：

![TTPs](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/TTPs.png)

<center>痛苦金字塔</center>

对于攻击方，TTPs 反映了攻击者的行为，调整 TTPs 所需付出的时间和金钱成本也最为昂贵。
对于防守方，基于TTPs 的检测和响应可能给对手造成更多的痛苦。

因此 TTPs 也是痛苦金字塔中对防守最有价值的一类 IOCs。但另一方面，这类 IOCs 更加难以识别和应用，由于大多数安全工具并不太适合利用它们，也意味着收集和应用 TTPs 到网络防御的难度系数是最高的。

关于"痛苦金字塔"更详细介绍，可以查看这个链接：[The Pyramid of Pain](https://detect-respond.blogspot.com/2013/03/the-pyramid-of-pain.html)。 

ATT&CK观察的主要目标是攻击者的TTPs，并将观察结果记录进行总结分类，从而形成知识库。

简单点说，ATT&CK是基于现实的、已经发生的TTPs的观察、总结形成的知识库，这不是学术理论的结果，意味着ATT&CK是具有很强的实战性和可落地性。


### 1.3 抽象级别

ATT＆CK中“战术（Tactics）”和“技术（Techniques）”的抽象级别是它与其他安全模型之间的重要区别。

一般安全模型可以分为高中低三种抽象级别：

- 高级抽象：例如：Lockheed Martin Cyber KillChain®、Microsoft STRIDE
- 中级抽象：例如：MITRE ATT&CK
- 低级别抽象：例如：漏洞利用库、恶意软件库；

高级别抽象的安全模型，对于理解高层次的过程和攻击目标很有用，但是这些模型无法有效传达攻击者的动作以及动作之间的关系，也无法描述动作序列与环境中的防御策略（监控的数据源、防御、配置等）之间的关系。

对于低级别抽象模型，例如漏洞利用数据，它描述了可利用软件的特定实例（这些实例通常与代码示例一起使用），但这些实例无法很好地描述”是否或能够(应该)被使用以及使用它们的困难程度“。 同样的，恶意软件数据库也通常缺少有关”如何使用恶意软件以及由谁使用“的上下文，以及它们也都没有考虑“如何将合法软件用于恶意目的”的情况。

因此，在实际环境中，需要像ATT&CK这样的一个中级抽象模型，将上述的高级别抽象模型和低级别抽象模型中各不同的组件组合在一起。

ATT&CK中的“战术”和“技术”在一定程度上定义了生命周期中的对抗行为，使它们能够有效的映射到防御中。诸如”控制“、”执行“和”维持“之类的高级别抽象模型的概念在ATT&CK中被进一步细分为更具体描述的类别，再在这些类别中定义和分类更具体的行为动作。

对于例如漏洞利用库和恶意软件库之类的较低级别概念，他们是很有用的工具包，但由于其数量很大，除了常规的漏洞扫描、快速修复和IOCs用途之外，很难在行为分析中被解释。要完全理解他们的效用，就必须了解其实现目标的上下文。中级抽象模型ATT&CK可以将漏洞库（或恶意软件库）等低级别抽象的信息和事件数据关联起来，以显示谁在做什么，以及特定技术的普遍使用情况。

各抽象级别示意图：

![abstraction](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/abstraction.png)

<center>抽象级别</center>

总的来说，ATT&CK的技术抽象实现了以下两点：
- 进攻和防守双方都能理解的行动和目标的常见分类。
- 适当的分类级别，将对手的行动与防御它的具体方法联系起来。


### 1.4 通用语言

这一点是我个人的补充的，在官方文档并没有把这一点单独拿出来说，但我认为”通用语言“也是ATT&CK会那么火的原因，也是我认为ATT&CK最重要的价值之一。

如前面所说，ATT&CK是站在攻击视角基于现实世界对攻击者的行为进行观察研究，并对观察的结果进行分类和抽象成具体的战术和技术。在这个过程中，就无形中将所有的观察到的攻击行为进行了规范化的处理，形成描述攻击行为（TTPs）的语言和词库。

还记得前文说的安全专家的个人经验吗？经过分类和抽象后的ATT&CK就可以将大部分的安全专家的经验纳入其中。相当于ATT&CK提供了一套描述攻防的标准语言，只要按照这套语言，就可以描述出安全专家的经验。当安全经验可以被描述之后，就可以有效提高业界交流效率和质量。

试想下，在没有CVE（Common Vulnerabilities and Exposures，通用漏洞披露）之前，安全从业者是如何描述一个漏洞，脑洞大开想想以下场景：

```
老板：小王啊，最近刚刚出现的那个Nginx漏洞我们环境修复了没有啊？
板砖王：老板，你说的是的哪个漏洞啊，最近几个月Nginx暴了好多个漏洞啊，13.02版本有一个，12.93版本有两个。
老板：就是那个，可以影响ssl版本，什么什么的可以窃取密钥的；
搬砖王：是那个吗，Nginx影响了...
老板：不是，是那个 .... 
搬砖王： 哦哦，那个啊，官网发了新的版本，只是说修复了XX漏洞，但不知道是不是包含这个，要上线验证下..
老板：.....
（以上场景纯粹瞎B，切勿对号入座）
```
在没有CVE漏洞编号之前，其实很难描述程序某一个具体漏洞情况，描述一个漏洞通常会使用漏洞描述、软件版本号、利用原理、风险级别等信息来描述，不同组织或个人对漏洞的不同理解，就会描述出不同的东西，在交流起来就会显得特别困难。但当出现CVE编号之后，复杂漏洞描述被抽象精炼成一个编号，根据这个编号就可以快速定位到漏洞详情，这样就大大节省了沟通成本。

接着上面例子，那如果有了CVE编号之后，上面这个对话就可以精简很多了：
```
老板：小王啊，最近刚刚出的那个CVE-2020-0000漏洞我们环境修复了没有啊？
搬砖王：哦，老板，CVE-2020-0000漏洞是Nginx提权漏洞，影响了XXX版本，官网今天上午发布了新的版本就是可以修复这个漏洞的，同时新的版本还修复了CVE-2020-002、CVE-2020-003漏洞。我们环境已经在灰度上线了，预计今天就可以全部修复。
老板：好的，要及时跟进好。
```
可见，有了CVE编号之后，就可以大大提升了漏洞描述的效率。而且从实际情况来看，CVE也已然成为一个很重要的漏洞描述事实标准，业界在描述一个漏洞时，都会使用CVE来描述。

同理，ATT&CK也是一个规范化的”攻击行为“描述。各类被观察到的攻击行为被分类成通用描述，甚至为这些分类都编上唯一的编号，这样一来，就可以像CVE描述漏洞一样方便来描述攻击行为了。

例如，组（Groups）G0020就代表“Equation（方程式组织）”这个APT组织，这样在交流时使用G0020就可以很直接明白所说的就是那个“曾经开发出永恒之蓝的APT组织“；

再比如T1053.003技术是在战术“Excution”TA0002下，表示在Linux平台下的crontab定时任务，在描述这个技术的时候就直接说T1053.003这个具体的技术概念来代替含糊不清的”定时任务“概念。

再次脑洞一下，如果`一个APT组织在Linux平台下使用了crontab这个功能实现了一次恶意代码的执行`，就可以被简单描述成：

```
G0030这个APT组织使用T1053.003技术实现了TA0002战术目标；  
```

就这样简单一句话就可以把攻击者、攻击行为、攻击手法描述清楚，不存在歧义且易于理解。

更为重要的是，经过标准化的攻击行为描述，更利用自动化检测的执行，更便于安全工具、安全策略的开发。想想现在有多少安全工具是基于CVE上的，就不难想象往后会有多少安全分析工具会基于ATT&CK来开发。

随着ATTCK不断被人熟知，ATT&CK也会成为一个关于”攻击行为“的事实标准，安全从业者就可以基于这个标准去描述组织的安全状况、防护能力、攻防演练行动、安全产品能力、安全产品覆盖范围，也可以描述整个组织的安全运营体系成熟度、防御差距、现状评估等等安全能力的情况。


## 2. ATTCK技术领域
ATT&CK由一系列技术领域组成，截止当前，MITRE ATTCK 已经定义了3种技术领域，分别是
- 企业(Enterprise)：传统企业网络和云环境；
- 移动(Mobile)：移动通信设备；
- ICS：Idustrial Control System 工控系统 

在每个技术领域，ATT&CK定义多个平台(platforms)，一个平台可能是一个操作系统或者是是一个应用；“技术”和“子技术”可以应用在不同平台上。

|技术领域 |  平台 |
|------|--------|
|Enterprise|Linux, macOS, Windows, AWS, Azure, GCP, SaaS, Office 365, Azure AD|
|Mobile|Android,IOS|

另外，PRE-ATTCK是ATT&CK技术领域的扩展，覆盖了获得网络访问之前的需求收集、侦查、和武器化的对抗行为描述，它独立于“技术”，通过对“攻击者尝试获取组织机构访问权限的行为”进行行为建模。

```
第一个ATT＆CK模型于2013年9月创建，主要侧重于Windows企业环境。通过内部研究和进一步完善开发并随后于2015年5月公开发布，涵盖了9种战术和96种技术。从那时起，ATT&CK在网络安全社区的贡献基础上经历了巨大的增长。MITRE创建了几个额外的基于ATT&CK的模型，这些模型是基于创建第一个ATT&CK的方法创建的。最初的ATT＆CK在2017年扩展到Windows以外的版本，包括Mac和Linux，被称为Enterprise ATT＆CK。PRE-ATT＆CK于2017年发布，重点关注kill-chain的“左侧攻击”行为。ATT＆CK移动版也于2017年发布，重点关注移动专用领域的行为。ATT＆CK Cloud于2019年作为Enterprise的一部分发布，描述了针对云的行为环境和服务。ATT＆CK for ICS于2020年发布，收录了工业控制系统的攻击行为。
```

PS： 如没有特殊说明，本文所说的ATT&CK模型指Enterprise，ATT&CK for ICS、PRE-ATTCK和Mobile暂不在本文讨论范围；

## 3. ATT&CK组成


### 3.1 ATT&CK 矩阵

ATT&CK的基础是一组”技术“和”子技术“，这些”技术“和”子技术“代表了对手可以执行的行动来实现目标。这些目标由”技术“和”子技术“所隶属的”战术“类别所代表。

”战术“指的是攻击行为的目标，而”技术“是指为了实现目标而执行的技术，”子技术“是比”技术“更具体的技术说明。

现阶段ATT&CK将已知的攻击行为分为12个战术、156个技术和272个子技术；

”战术“、”技术“和”子技术“之间的关系通过ATT&CK矩阵实现可视化。

![matrix](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/matrix.png)
<center>战术和(子)技术矩阵</center>


### 3.2 战术（Tactics）
”战术“就是“目标”，从某种意义上讲“战术”就是“技术”执行的意义，也是对手执行具体攻击技术的原因；”战术“是个别”技术“的上下文类别，涵盖了对手在行动过程中所做事情的标准表示法，例如持久性（Persistence）、发现（Discovery）、横向移动（Lateral Movement）、执行（Execution）和泄露（Exfiltration）等。“战术”也被视为ATT&CK中的标签，不同的“技术”或“子技术”如果实现相同的目的就会被打上同一个“战术”的标签；

每个“战术”都有一个定义描述了该类别，并作为“技术”的分类依据。例如：“执行”被定义为一种表示“导致在本地或远程系统上执行敌对控制代码”的“(子)技术”的“战术”，这种“战术”通常与“初始访问”一起使用，作为一旦获得访问就执行代码的手段，以及横向移动以扩展对网络上远程系统访问。

另外，也可以根据需要定义其他“战术”类别，以更准确地描述对手的目标。其他领域的ATT&CK建模方法的应用也可能需要新的或不同的类别来关联技术，尽管可能与现有模型中的策略定义有一些重叠。

![Tactics-Example](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/Tactics-Example.png)
<center>例如：Execution战术</center>


### 3.3 技术及子技术（Techniques and Sub-Techniques）
“技术”代表攻击者通过执行动作来实现“战术”目标的“方式”。

“技术”代表着一个对手如何通过执行一个行动来达到一个“战术”目标。例如，对手可能从操作系统转储凭据，以获得对网络中有用凭据的访问权。

“技术”也可以代表一个对手通过执行一个动作所获得的“什么”，对于“发现（Discovery）”战术来说，这是与其他“战术”最大的区别，因为这些技术可以突出对手通过特定行动获得的信息类型。

实现“战术”目标的方法或技术可能很多，因此每个“战术”类别中都有多种"技术"。 同样，可能有多种方法来执行一项“技术”，因此一项“技术”下可能有多种不同的“子技术”。

“子技术”进一步将技术描述的行为分解为关于“如何使用行为实现目标”的更具体的描述。 例如，对于”OS Credential Dumping“技术，此技术下有几种更特定的行为可以描述为”子技术“，包括”访问LSASS内存“，“安全帐户管理器”或“访问"/etc/passwd"和"/etc/shadow”。

![Techniques-Example](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/Techniques-Example.png)

<center>例如：Command and Scripting Interpreter技术</center>



"技术"主要字段及说明：

|字段 |  类型 | 描述 |
|------|--------| ------|
|名称（Name）| Field | 技术（子技术）的名称 |
|ID（ID）|Tag|知识库内技术（子技术）的唯一编号。格式： （技术）T####；（子技术）T####.###|
|子技术（Sub-Techniques）|Field|属于某一技术的子技术id。*只适用于技术，不适用于子技术|
|战术（Tactic）|Tag|(子)技术可以用来完成的战术目标。(子)技术可用于执行一种或多种策略。|
|描述（Description）|Field|关于(子)技术的信息，它是什么，它通常用于什么，对手如何利用它，以及如何使用它的变化。包括对与该技术相关的技术信息的权威文章的参考，以及在野外使用的参考。|
|平台（Platform）|Tag|一个内部运行着对手的系统;可能是操作系统或应用程序(例如微软Windows)。(子)技术可以应用于多个平台。|
|所需权限（Permissions Required）|Tag|对手在系统中执行（子）技术所需要的最低权限，限于权限提升。*仅在“权限提升”中需要|
|有效权限（Effective Permissions）|Tag|对手通过执行(子)技术将获得的权限级别。只适用于(子)特权升级策略下的技术。如果在执行(子)技术时可以设置有效权限，则可以有多个条目。*仅在“权限升级”中需要|
|数据源（Data Source）|Tag|由传感器或日志系统收集的信息来源，可用于收集与识别对手正在执行的动作、动作的顺序或这些动作的结果相关的信息。数据源列表可以包含针对特定(子)技术执行操作的不同变体。此属性被限制为一个已定义的列表，以允许基于唯一数据源的技术覆盖率分析。(例如，“如果我有进程监视，我可以检测哪些技术?”)|
|防御绕过（Defense Bypassed）|Tag|用来绕过或逃避一个特定的防御工具，方法或过程。仅适用于防御闪避(子)技术。*仅子“逃避防御”中需要|
|版本（Version）|Field|（子）技术版本，格式：MAJOR.MINOR|
|影响类型（Impact Type）|Tag|表示(子)技术是否可用于完整性或可用性攻击。适用于影响(子)技术。|
|检测（Detection）|Field|包含可以识别一个已被对手使用的（子）技术的高级分析过程、传感器、数据或检测策略。此字段描述用于防御检测的方法，以便安全检测人员能够根据此采取具体的防御方法或行动。存在很多检测（子）技术的方法，但ATTCK和MITRE不认可特定的供应商解决方案，因此检测建议仅是方法或工具类型，与特定的供应商或工具无关。另外，即使（子）技术并不是总是可能被检测的，但也应该被记录。|
|减缓措施（Mitigation）|Relationship/Field|包含可以阻止（子）技术执行成功或达到预期效果的配置、工具或过程。此字段描述了如何将特定的缓解措施应用在（子）技术的细节，以供安全防御人员能够据此采取具体的缓解策略。同样，缓解建议仍然与特定的供应商无关，也并非每个（子）技术都能够被缓解。|
<center>“技术”主要字段</center>



#### 3.3.1 过程（Procedures）

要完成一次成功的进攻，仅仅有好的“战术”和“技术”是不够的，攻击者需要使用一种称为“过程”的特殊动作序列来执行其攻击周期中的每一步。在ATT＆CK中，“过程”是对手用于“技术”或“子技术”特定的实施方式。

还需要注意的”过程“可以跨越多种“技术”和“子技术”。例如，攻击者用于“转储凭据”的“过程”包括PowerShell、Process Injection和LSASS Memory，它们都是不同的行为，所以“过程”还包括如何使用特定工具。

在ATT＆CK中，会将观察到“过程”记录在“技术”和“子技术”页面的“Procedure Examples”部分。

![Procedure-Example](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/Procedure-Example.png)

<center>例如：Command and Scripting Interpreter技术对应的Procedure，第一列是APT组织名称，第二列是该APT组织实现该技术使用的过程</center>

#### 3.3.2 子战术（Sub-Techniques）
ATT＆CK在2020年增加了子技术，这标志着知识库中行为描述方式的发生重大转变。 这种变化是由于需要解决随着ATT＆CK多年来的发展而出现的一些技术抽象级别问题。 有些“技术”描述非常广泛，有些却很狭窄（仅描述了非常具体的行为）。 这种不平衡导致ATT&CK变得异常臃肿，不仅使ATT&CK难以可视化，而且也使理解某些技术背后的目的变得困难。

子技术主要改善ATT&CK以下几点：

- 使技术的抽象级别在整个知识库中相似
- 将技术数量减少到可管理的水平
- 提供一种允许轻松添加子技术的结构，这将减少随着时间的推移对技术进行更改的需求
- 某些技术可能并不简单，可以采用多种方法进行实施，而这些方法是需要考虑的
- 简化向ATT&CK添加新技术（重叠的技术）域的过程
- 启用更详细的数据源和描述，让行为可以在特定的平台可被观察

“子技术”与“技术”没有一对多的关系，每个“子技术”将只与一个父“技术”有一个关系，这样可以避免整个模型变得复杂且难以维护。

在某些情况下，具有多个父项的“子技术”可能会采用跨越多种“战术”的“技术”。

例如，计划任务/作业（Scheduled Task/Job）技术既包含在“持久性（Persistence）”战术之中，也包含在“特权升级（Privilege Escalation）”战术之中，自身的4个子技术中只有某些子技术是适用于“特权升级（Privilege Escalation）”的。在这种情况下，“子技术（Sub-Techniques）”并不需要属于所有“战术（Tactics）”，即使该“子技术（Sub-Techniques）”的父“技术（Techniques）”是属于该“战术（Tactics）”的。只要“子技术（Sub-Techniques）”概念上属于“技术（Techniques）”即可，不需要满足每个父“技术”的“战术”。

上面这段话比较拗口，简单点说，就是“子技术（Sub-Techniques）”只与其父“技术（Techniques）”有关，与其“技术（Techniques）”所属的“战术（Tactics）”无关。

ATT&CK中“子技术”属性与“技术”基本相同，但会在ID上会在父“技术”后加后缀以示区别。

![Sub-techniques-Example](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/Sub-techniques-Example.png)

<center>例如：Command and Scripting Interpreter的子技术PowerShell</center>


### 3.4 组(Groups)

在ATT&CK中，“组(Groups)”是用来跟踪已知攻击者的对象，“组”被定义为命名的入侵集、威胁组、参与者组或活动，它们通常代表有目标的、持续的威胁活动。ATT&CK主要关注APT群体。

![Groups-Example](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/Groups-Example.png)

<center>例如：APT32</center>



"组"主要字段及说明：

| 字段                                            | 类型               | 描述                                                         |
| ----------------------------------------------- | ------------------ | ------------------------------------------------------------ |
| 名称（name）                                    | Field              | 组的名称                                                     |
| ID                                              | Tag                | 在知识库中组的唯一编号。格式：G####                          |
| 版本（Version）                                 | Field              | 组的版本，格式： MAJOR.MINOR                                 |
| 描述（Description）                             | Field              | 基于公开威胁报告中组的描述，它可能包含活动时间、可疑的细节、目标行业，以及归因于该组的其他值得注意的事件。 |
| （子）技术（Techniques / Sub- Techniques Used） | Relationship/Field | 组使用的（子）技术列表，以及描述该组使用该技术过程的细节（TTPs的上下文中）。 |

<center>Groups主要字段及说明</center>

### 3.5 软件(Software)

攻击者通常在入侵期间使用不同类型的软件。 软件可以代表一种“技术”或“子技术”的实例，因此在ATT＆CK中进行分类也是必须的。 

软件分为两大类：工具和恶意软件。

- 工具： 指商业、开源、自研或公开可用的软件，可以由防御者，渗透测试者，红队成员或对手使用。 该类别既包括通常在企业系统上找不到的软件，也包括通常作为环境中已经存在的操作系统的一部分而获得的软件。 例如PsExec，Metasploit，Mimikatz，以及Windows实用程序，例如Net，netstat，Tasklist等。

- 恶意软件：指商业、开源或闭源、用于用于恶意目的软件。 例如：PlugX、CHOPSTICK等。

软件类别可以进一步细分，但当前分类背后的想法是展示对手如何使用工具和合法软件来执行行动，就像他们使用传统恶意软件一样。

![Software-Example](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/Software-Example.png)

<center>例如：Cobalt Strike</center>



"软件"主要字段及说明：

| 字段                                            | 类型               | 描述                                                         |
| ----------------------------------------------- | ------------------ | ------------------------------------------------------------ |
| 名称（Name）                                    | Field              | 软件名称                                                     |
| ID（ID）                                        | Tag                | 在知识库中组的唯一编号。格式：S####                          |
| 版本（Version）                                 | Field              | 软件的版本；格式：MAJOR.MINOR                                |
| 类型（Type）                                    | Tag                | 软件类型，“恶意程序（malware）”或者“工具（tool）”            |
| 平台（Platform）                                | Tag                | 软件使用的平台，例如Windows                                  |
| 描述（Description）                             | Field              | 基于技术参考或公开威胁报告的软件描述。它可能包含与已知使用该软件的组或其他与之相关的技术细节。 |
| （子）技术（Techniques / Sub- Techniques Used） | Relationship/Field | 由软件实施或使用的（子）技术列表；                           |

<center>Software主要字段及说明</center>




### 3.6 减缓措施（Mitigations）

ATT＆CK中的缓解措施（Mitigations）是用于描述”如何阻止某种技术或子技术成功执行“的一种安全概念和技术类别；

截至2020年3月，Enterprise ATT＆CK中有41种缓解措施，其中包括应用程序隔离和沙箱，数据备份，执行预防和网络分段等缓解措施。缓解措施与供应商产品无关，仅描述技术的类别或类别，而不是特定的解决方案。

![Mitigations-Example](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/Mitigations-Example.png)

<center>例如：Disable or Remove Feature or Program</center>

"减缓措施"主要字段及说明：

| 字段                                             | 类型               | 描述                                |
| ------------------------------------------------ | ------------------ | ----------------------------------- |
| 名称（Name）                                     | Field              | 检索措施名称                        |
| ID（ID）                                         | Tag                | 在知识库中组的唯一编号。格式：M#### |
| 描述（Description）                              | Field              | 减缓措施的描述                      |
| 版本（Version）                                  | Field              | 减缓措施的版本；格式：MAJOR.MINOR   |
| （子）技术（Techniques Addressed by Mitigation） | Relationship/Field | 此缓解措施可能涉及的(子)技术列表。  |

<center>Mitigations主要字段及说明</center>



### 3.7 ATT&CK各组件之间的关系

上述几个组件并非单独存在，在ATT＆CK中每个组件都以某种方式与其他组件关联。

它们之间的关系可以用以下图展示：
![ATT&CK_Model_Relationships](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/ATT&CK_Model_Relationships.jpg)

<center>ATT&CK Model Relationships</center>

例如，APT28组织使用Mimikatz对Windows LSASS进程内存进行凭据转储，可以用以下图表示

![ATT&CK_Model_Relationships_Example](https://github.com/chiww/chiww.github.io/assets/what-is-ATTCK/ATT&CK_Model_Relationships_Example.jpg)

<center>ATT&CK Model Relationships Example</center>

## 4. ATTCK使用场景

上面讲了ATT&CK那么多，最关键的还是使用场景。ATTCK使用场景主要有4个场景，每个场景官方都有最佳实践的指导，并且列出场景下一些可使用的工具。     

```
受限于篇幅长度原因，本文仅对场景做基本描述，接下来会再起另外几篇文章会尝试逐一去描述每个场景的使用方法。
```


### 4.1 行为检测分析

传统的检测通常使用入侵特征（IOCs）或恶意活动签名来检测。行为检测分析与传统的特征检测不同，行为检测分析不依赖于先验的知识（IOCs或签名），与对手的使用的工具也无关。

ATT&CK可以作为构建和测试行为分析的工具，用于检测环境中的敌对行为，识别系统或网络中潜在的恶意活动。

网络行为分析库（Cyber Analytics Repository，CAR）是MITER基于MITER ATT＆CK对手模型开发的分析知识库，可以作为组织作为ATTCK进行行为检测分析的起点。


### 4.2 威胁情报
网络威胁情报涵盖网络和影响网络安全的威胁行动者（集团）的知识，它包括有关恶意软件、工具、TTPs、谍报技术、行为和其他威胁相关的指标信息。

ATT&CK从行为分析角度理解攻击者，收录形成文档，这些文档记录与攻击者可能使用的工具无关。

通过这些文档，分析人员和防御者可以更好理解各攻击者的通用行为，并有效的映射到自身的防御体系上。另外，ATT&CK的结构化格式可以对标准之外的行为进行分类来增加威胁报告的价值。


### 4.3 对手仿真与红队
“对手仿真”和“红队攻击”都是一种验证防御能力的行为，但两者又有不同。

“对手仿真”首先假定其模拟的是哪一个攻击组织，并根据该组织的威胁情报信息，模拟该攻击组织的行为。通过模拟攻击者的行为，来评估某一技术领域的安全全过程。“对手仿真”关注的是组织在其生命周期的所有适用点上验证检测和（或）缓解对手活动影响的能力。


ATT&CK是可以用于作创建“对手模拟场景”的工具，以测试和验证对手常见技术的防御。特定的攻击者详情会以文档的形式记录在ATT&CK结构化文档中，防御者和风险狩猎可以使用这些文档来调整和改机防御措施。

“红队攻击”是指不使用已知的威胁情报，以对抗的心态进行攻击演示。红队重点是在不被发现的情况下完成行动的最终目标，以显示成功入侵的任务或行动的影响。

ATT&CK可以作为一个工具用来创建“红队计划”和组织行动，用以在网络中避免某些防御措施。

另外，ATT&CK还可以用作研究的路线图，用来开发普通防御系统可能无法检测的新攻击手段或方法。


### 4.4 差距分析与评估

差距分析可以让组织确定环境中哪些部分缺少防护或缺少可见性，这些缺口代表在环境中存在潜在的防御或监控盲点，可以让攻击者在未被发现的或未被有效拦截的情况下访问其网络。

ATT&CK可以作为一个以常见行为为中心的攻击模型，用于评估防御措施中的工具、监控和拦截是否有效。识别出差距后，就可以根据优先级安排防御体系建设的投资计划。类似的，在购买安全产品之前，也可以与一个常见的对手攻击模型进行比较，以评估在ATT&CK中的覆盖范围。

另外，组织的安全运营中心（SOC）是许多大中型企业网络的重要组成部分，可以持续监控网络中的威胁。理解SOC的成熟度对于确定SOC的有效性非常重要。ATT&CK可以作为一种测量方法用来确定SOC在检测、分析和响应入侵方面的有效性。与差距分析评估相似，SOC成熟度评估关注SOC用于检测、理解和响应网络中不断变化的威胁和过程。



## 5. 后记

在完成此篇文章之前，阅读了MITRE ATT&CK的官方文档《MITRE ATT&CKÒ: Design and Philosophy》，并作为这篇的文章的主要参考源。《MITRE ATT&CK: Design and Philosophy》这篇文章详细阐述了ATTCK的设计哲学和理念，是作为了解ATT&CK整个体系很详细的入门教程，可惜的是全文都是英文，看起来挺费劲的，所以本文在参考此文档基础上，做了部分翻译和口语化解释，以便更好的阅读和理解。

接触ATT&CK也是一个偶然的机会，参加了今年4月份青藤云CEO张福一个线上的ATT&CK课程，受益匪浅。对于ATT&CK我也是一个新手，还有很多细节值得慢慢去研究学习。

ATT&CK官方资料丰富，社区活跃，基于的ATT&CK的检测思路和工具也相当多，可以说是一个网络安全攻防的知识宝藏库。接下来一段时间，我将会顺着官方文档教程逐一去实践这个框架，并整理成文，如果感兴趣的小伙伴可以关注本公众号。

PS：感谢青藤云CEO张福举办的那次线上交流会，并给我寄了一本ATTCK实践指南小册子，让我真正开始接触这个先进的理念，也期待青藤云能够对ATTCK做出更多研究，HIDS产品大卖！ 




## 参考：

1. 《ATT&CK框架使用指南》  青藤云安全
2. 《MITRE ATT&CK: Design and Philosophy》 ©2020 The MITRE Corporation. 
3. 《TTPs & IOCs & 痛苦金字塔》 https://www.jianshu.com/p/b3654b179277 Viola_Security
4. 《TACTICS, TECHNIQUES, AND PROCEDURES》 https://azeria-labs.com/tactics-techniques-and-procedures-ttps/

