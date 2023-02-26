---
layout: post
title:  "构建网络安全事件调查响应平台"
date:   2022-12-31 10:28:28 +0800
categories: 安全运营
tags: 应急响应 攻防实战
---


### 0. 前言

网络安全事件是指组织内发生的网络入侵事件，“事件调查响应”就是对这些入侵事件进行调查分析、抑制、止损等响应的过程。“网络安全事件调查响应”更专业叫法的应该叫"**数字调查与事件响应(Digital Forensics and Incident Response (DFIR))**"

完整的DFIR除了安全事件调查、响应取证以外，还会涉及其他领域更专业调查取证知识的，因本人能力有限，只能从终端主机侧，以纯技术视角来简述安全事件调查的基本要素、思路、方式方法，以及简要介绍基于**Velociraptor(迅猛龙)**构建调查响应平台的基本概念。



### 1. 攻防的事件响应

网络攻防对抗是事件调查响应主要的应用场景，这种高强度对抗产生的事件是需要组织内安全调查人员去调查分析，其调查响应工作量是那些通用的木马、病毒或者脚本小子的攻击事件所不能比的。

通用的木马、病毒事件，其特征比较固定明显，且这些特征都是已经被安全厂商、安全研究机构研究通透了，那么这些恶意行为已经是清晰明了的，也就不需要安全人员再去调查一遍，只要发现特定的IOC特征，事件处置人员直接按照已有的调查报告处置风险就可以了，甚至一些安全防护工具已经自动完成拦截而无需人为介入处理。

但在网络攻防对抗中，攻击者的攻击意图是不确定的，攻击者的所使用的技术手法一定也是针对组织做了免杀或者绕过，这些战术手法就需要防守方的安全调查人员有更专业能力和工具来完成调查响应工作。

为了说明攻防中的事件响应是如何运作的，我们可以先简要列出攻防对抗中有哪些阶段，以及防守方是如何应对的。

从攻防视角上看，攻击过程会有以下几个阶段（为简化概念，就不用KillChain或者ATT&CK模型）：

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/pic01.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">攻击入侵阶段</div></center>

在整个过程中，“打点”是找到可以利用的突破口，一旦利用成功就实现了“驻留”，“驻留”是入侵的关键第一步，“驻留”的成功意味着攻击方已经突破防守方的安全边界进入内网。后续的“横向”是为了扩大战果，直到最终实现入侵攻击的目标。

同时，在防守方在防御过程中也有几个关键的阶段：

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/pic02.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">防守方防守阶段</div></center>

“检测”是对边界的安全检测，确保在攻击发生时能够及时发现异常，一旦发现高度可疑的异常，就需要快速进入“调查”阶段，梳理出攻击路径，并进行“抑制”止损；在完成“抑制”才会进行更完整的、彻底的修复阶段。

防守方防守阶段中的“调查”和“抑制”就是安全事件调查响应过程。而攻击中的“驻留”和“横向”是发生在终端主机上的，那么顺其自然，防守方的事件调查响应也应该就聚焦在终端主机上。

“检测”和“调查”都需要有相应的检测发现规则，最大的不同在于，“检测”关注系统中某一瞬间的状态改变值来识别异常，而“调查”是基于这个异常状态点进一步还原状态变更的原因，更关注系统在发生异常前后一段时间的上下文状态，也就是把一个“异常点”还原成一个“事件线”，如果涉及多个终端主机就还原成一个“攻击面”。

安全事件调查响应的工作，就是顺着这样的思路，通过历史状态记录找出攻击方的攻击手法和攻击路径，确定攻击者的意图，评估损失并及时有效制止恢复。 



### 2. 事件调查响应生命周期

安全事件调查响应在安全体系中从开始到结束有一个区间范围，暂可以认为是安全事件响应的生命周期。

在一个组织内，日常需要处理大量的安全设备告警，按照告警响应人员以及告警类型分类，大致有以下三种：

- Level1：处理设备日常告警，排除误报，处理低风险告警，快速闭环；

- Level2:  处理Level1无法处理的告警，对安全事件更深入分析；

- Level3：对高级威胁事件分析(如APT攻击)，需要更专业的知识(如木马后门二进制分析、攻击者画像溯源等)

通常情况下，`Level1`响应人员需要面对大量的设备告警，需要对每个告警进行判断分析，这需要耗费相当的人力资源。所以，`Level1`这部分工作现在越来越多由自动化工具来完成，例如，一部分告警是已经被拦截的自动化攻击告警，这些告警可能是一些“无差别”攻击行为，可以通过SOAR编排脚本直接拦截掉；另外一些告警可以通过多设备关联分析、AI机器学习、行为分析判断等策略方法进一步判别、过滤、聚合等手段以降低数量。

设备告警经过一段事件运营、经过`Level1`的处理，只有最后一小部分告警，才会进入到事件调查响应人员(`Level2`)的处理流程中。类似有这样的一个处理逻辑：

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/pic03.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">“告警”与“事件”</div></center>

`Alert`是各类安全设备产生的原始告警，这些告警数量巨大；`Event`是经过`Level2`的“事件”，相比`Alert`数量会降低很多，比`Alert`更准确描述环境状态，具有更高的可信度，但此时的`Event`还不能算是一个真实的安全事件(Incident)，`Incident`是指确实发生有损失的事件，这需要安全运营人员进行“事件调查”后才能判断。

当然，理想的状态是`Level1`之后就已经是可以明确的事件，或者说不需要人工介入就能够完全将所有告警自动判断是否是事件，但在安全运营实际情况下，这是很高的难度。主要有以下几个原因：

1. 安全运营阶段可以使用各种手法(例如，关联分析、SOAR)去降噪、聚合，提高运营告警风险研判效率，但组织内的环境是动态变化的，即使再完美策略，也需要根据业务状态而变化；而为了跟踪业务变化的过程，就需要人工介入去调查分析，输出新的检测策略去适配新的业务状态；
2. 面对APT类、攻防类等强目标性的攻击，攻击行为是“Case By Case”的，自动化的规则策略都有被绕过的可能，期望自动化就能解决所有问题是不切实际的。我们只能尽可能减少人工重复的事件调查，但不能完全避免人工的事件调查。 

所以，在一个成熟安全运营环境中，大部分的设备告警应该被自动化策略处理掉，仍存在一部分告警是必须人工通过事件调查才能完全闭环的。事件调查主要作用在`Level2`阶段，就是为了更好完成事件闭环。

#### 2.1 发现

检测告警是事件“调查”起始，没有告警就不会有事件调查响应的动作，检测告警同时也是事件调查的基础。如果安全设备系统轻悄悄的，什么告警都没有，那就没有什么事件需要调查响应了。正是因为检测告警是事件调查响应的基础，而事件调查响应是检测告警的上层，所以当一个组织内还在疲于`Level1`阶段，是没有精力去考虑事件调查响应平台化建设。

检测环节是整个安全体系的检测，可以是WAF、IPS等网络设备的检测，也可以是HIDS、EDR终端安全检测的告警。这个检测目标是保证安全边界安全，检测不安全的边界行为，能够快速发现异常行为，产生告警。这种告警虽然“快”和“准”，但未必都是确实发生了造成损失的安全事件。 

所以，`Event`是`Alert`在经过`Level1`处理后的结果，也是事件调查的起点；同时，如果组织内没有有效度过`Level1`告警风暴阶段，而进行事件调查平台化建设没有实际意义。

#### 2.2 调查

如前面所说，`Alert`告警只能描述环境中某一个系统模块的在某一个时间的状态存在异常，不能描述这个异常状态发生的原因。因此`Level2`的事件调查就是从发生告警的这个“异常点”，去还原整个攻击事件的过程。

例如一个主机的HIDS告警，只是描述在某个时间点当前这个主机发现了一个异常的行为点，事件调查就需要回溯出这个“异常点”发生的原因。首先要做的是，把这个主机状态变化梳理出一个事件的“时间线”，如果涉及到多个主机的，就需要在多个主机上列出状态变化，这样与这个告警的主机都有一个事件的“时间线”，这些不同的时间线最后就可以整合成这个告警的“面”，这个“面”才能回答为什么会出现这个告警、是否是真实的风险、是否已经造成损失、是否需要处置等等问题。

因此，事件调查从本质上看，关键是能够掌握足够多的环境信息，对环境是否具备相当的“可见性”，才能完成事件调查工作。这有点类似警察在调查案件需要案发现场监控、取证、访问当事人一样，办案的过程就是通过这些”可见性“方式去获取足够证据来实现案件的侦察。 (当然，真实情况中事件调查还包括人员访谈、恶意程序分析，这些都算是另外一个领域，能力有限，就不过多探讨。)

系统环境的状态以及状态变化记录，就是”可见性“最直接的一种，这种状态或者记录就是“日志”。常见的调查过程就是获取这些日志的过程。 例如，在终端主机做事件调查，就需要关注以下内容：

1. 进程网络日志
2. 用户账号记录
3. 用户访问记录
4. 计划任务
5. 用户登录日志
6. 服务创建日志
7. 应用日志
8. 命令执行日志
9. ......

这些日志保存的越详细越具体，颗粒度越精细，越能够真实可靠还原整个事件过程，在实际调查响应中，可以参考ATT&CK框架中技术项的检测[数据源](https://attack.mitre.org/datasources/)来决定需要从哪些日志开始调查。

在实际安全运营中，如果组织内有建设SIEM作为日志收集管理平台，那么SIEM就自然承担了一部分事件调查能力，因为SIEM目标是尽可能收集各类日志，只要SIEM收录了日志，就对组织内的某一部分环境具备一定的“可见性”。

笔者曾参与过SIEM建设，在SIEM建设中就有一个灵魂拷问，就是收集的日志到底是以安全设备告警为主还是原始日志为主？例如，在主机检测方面，在部署了HIDS、EDR情况下是否还需要收集主机的原始数据(进程创建、网络等)？从检测角度看，HIDS、EDR产品在具备一定检测能力的情况下，是可以不再收集原始事件信息，但这些告警信息对调查响应是不够的。主要原因是这些告警信息缺少上下文信息，很难还原整个事件过程。同理，如果SIEM系统能够完全收录所有终端主机所有原始事件信息，SIEM就可以成为一个极佳的调查平台，在能力允许情况下甚至能够补充更多安全设备没有实现的检测项，但这样做的成本是庞大的。试想想在一个组织中有上万台服务器、上十万台PC终端，这些终端主机所有原始数据是这么庞大的数据需要多少的存储和计算成本，而在实际调查响应中，能有1%的日志能够在事件调查用的上就已经很不错了，这种建设方式“投入产出比”实在是太低，是SIEM很难完全胜任事件调查平台的最重要的非技术因素。

另外，事件调查不是固定的模式，是“Case by Case”的，SIEM上存储的日志未必能够满足所有事件调查响应所需要的信息，同时日志收集在SIEM是一项工程化事项，一项日志从收集到存储可能需要经历一定的时间，也不能满足紧急情况下的事件调查的需要。

因此，理论上，SIEM可以完成事件调查，但在实际上，SIEM受限于一些技术、成本因素，SIEM只能提供给调查分析提供一部分便利，但不能提供完整的事件调查分析能力。



#### 2.3 抑制

“抑制”是响应过程中最后一个阶段，“抑制”的效果是强依赖于事件调查结果的。例如，一个钓鱼邮件攻击，除了简单拦截已经发现的恶意邮件，还需要分析出恶意后门的C2地址，再针对这个C2进行封禁。如果事件调查中没有找到这个C2地址，缺少C2的拦截，那么这个“抑制”动作的效果是不彻底而且存在风险的。

“抑制”主要有以下几个方式： 

- 清理：清理是最有效的抑制手段，能够以最小损失还原系统的状态；

- 封禁：封禁是简单而且粗暴的做法，但也可能会影响业务；

- 停止：停止是停止服务，这个服务可以是线上的服务停止，也有可能是完全隔离一台终端或者主机让其用户不能正常使用。不管是什么情况的停止，都是最无奈的选择。导致这种情况的，有可能是0day尚未有好的手段来减缓攻击，也或者是因为各种原因导致调查响应的失败，无法实施清理或者封禁。 

抑制是需要执行一定动作的，这个动作可以在终端主机上，也可以将动作应用在网关设备上。

在事件调查清晰的情况下，“抑制”操作相比事件“调查”的来说，是简单很多的。

 

### 3. 事件调查响应方式

如上述所说，事件调查响应在`Level2`阶段，在这个阶段每个组织都有不同的落地实践方法。常见的有”人工手动调查响应“、”工具脚本响应“和”平台响应“。

#### 3.1 人工手动调查响应

人工手动调查响应，是指当发生安全事件需要调查时，由安全分析人员直接登录到目标终端主机上进行调查分析的动作。人工手动调查响应的特点就是有点”费人“，哪里有疑似失陷，分析人员就往哪里上：登录机器、上传工具，捞日志、过滤日志、匹配IOC，再一顿命令操作......

人工手动调查响应在小规模的环境中是非常适用的，不需要维护复杂的环境或者工具，但在大的组织里面，人工手动调查响应很快就会遇到瓶颈，主要原因可能有如下几点。 

首先，每个调查响应人员都有自己的调查分析方法、工具，方法和工具不能做到统一，就有可能出现同一个安全事件经不同人调查会有不同的调查结果，或者也存在不同的终端主机环境下受限于系统环境的限制出现调查异常的情况。 

例如，调查常用的日志解析过程，就有可能出现各种坑，可能是终端主机环境不能执行常用的日志解析工具，也可能是终端主机内存太小导致日志文件无法正常打开，更别说还有日志的快速检索过滤处理了。

其次，人工手动调查最大的问题是无法应对批量调查响应问题。试想着上百个中了钓鱼邮件的终端用户，调查人员是无法逐个去上机排查的。

```
从攻防实战中，这种钓鱼攻击的响应场景是经常发生的。在没有好的工具事件调查工具平台的情况下，调查人员通常通过询问用户的方式来判断钓鱼邮件的后门是否已经在终端上执行，但这种依靠用户来确认的攻击是否成功的方式是不可靠的。用户会因为一些不能明说的原因故意隐瞒真实情况。比如，钓鱼邮件有可能是为伪造招聘跳槽类的邮件，收到这些邮件的用户会隐瞒自己有”跳槽“的动机，谎称没有收到邮件或者没有运行邮件中的附件。这种对”人性的攻击“是对安全调查最大的难度。所以，用户的”证词“只能作为参考，而非判断事件的真实依据。 
```

总的来说，人工手动调查响应还是有很多的局限性，是非常依赖调查人员的能力水平和调查经验、实际调查环境的。

#### 3.2 工具脚本调查响应

为了解决”人工手动调查响应“各种局限和问题，应运而生了各种应急响应的工具或者脚本。

工具脚本调查响应是指将工具或者脚本拷贝到目标终端主机上，在本地批量收集日志、信息，以加快事件调查进展，就是将人工手动调查过程自动化的一个方式。 

适用调查响应的工具脚本通常可以分两种：

1. “Case by Case”检测脚本工具：例如”log4j漏洞利用“的脚本、”永恒之蓝“的调查工具脚本；
2. 通用的信息采集脚本工具：适配通用的大多数调查场景，目标是尽可能收集事件调查关注的日志。

"Case By Case"的工具脚本有最好的效果，但只是针对某个具体安全事件，这种往往只能做一种类型检测。这种比较适合在处理日常常见的病毒、木马攻击。

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/3-2.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">Case By Case专用应急事件调查工具</div></center>


通用的信息采集脚本工具能够适配常见的调查场景，但针对性不强，在某些情况是需要调查人员辅助于上级的人工手动调查动作，才能完全完成事件调查。

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/3-1.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">通用型事件调查工具</div></center>

工具脚本最大的好处就是能够按照既定的流程快速完成检测，其缺点也是”只能按照既定流程“完成检测或信息收集，在真实对抗或演练中，未必每个事件都是能够按照预置好的流程完成的，一旦工具脚本无法满足事件调查时，就会又进入“人工手动调查响应阶段”。

其次，工具和脚本需要不断维护更新，随着需要面对的场景不断增加，工具和脚本也会不断臃肿，维护这些工具脚本也会成为新的挑战。

#### 3.3 平台响应

工具脚本响应是人工手动响应的进阶，但随着调查场景不断增加，调查事件复杂不断提升，如果想要进一步提升调查响应的质量和效率，仅仅靠工具脚本就很难满足实际的事件调查需要了。在这种情况下，要进一步提升事件调查效率需要解决两个问题：工具的持续迭代开发维护和事件调查经验的有效沉淀，这就需要将事件调查能力提升到“平台级”，以系统化、工程化的思路来建设调查响应能力。

要实现平台化的调查响应，首先想到的是终端主机的集中管控平台来实现平台化。如前面提到的，调查响应的”战场“主要在终端主机上，在终端主机Agent上实现调查响应能力应该是最简单的事情，例如常见部署在主机的HIDS和部署在终端的EDR两种Agent，都是调查响应最好的载体，例如可以利用这些HIDS或者EDR系统推送一些执行脚本，来代替某些需要上机调查的动作。 

但目前所大规模使用的HIDS和EDR的核心能力在威胁检测，响应能力只是作为产品的附加值存在，并不能算是完整的”调查响应平台化“的东西；即使有”调查响应“的能力，其功能还是受限的，只能完成一些初级的调查响应动作。

因此，基于现有HIDS和EDR产品的作为调查响应平台，只能说”能用“，但离”好用“还有一段距离。

在大的组织内部，只有建立一套专用的调查响应平台，才能更好应对日益复杂的事件调查需要。 比如，如何应对大批量的终端主机调查取证，如何可以在某个特定范围内针对某个IOC狩猎，如何实现调查响应工作分工协助等。 笔者认为这样新的高阶调查方式应该是一个集合调查分析取证多种能力的综合“平台”。

这个平台应该具备以下特点：

- 适用范围广，具备非标环境、离线环境等其他各种"恶劣"环境下进行事件调查的能力；
- 灵活扩展能力，具备用户可定制扩展事件调查方式方法的能力；
- 可在线交互调查能力，满足不同事件调查场景不同调查方式不同的能力；

- 快速分析能力，能够确定失陷状态，是被植入后门还是被作为跳板；

- 调查取证能力，能够有效抓取解析日志，以更直观方式展示；

- 二次分析能力，能够结构化本地日志，离线导入分析，供二次深入调查分析；

- 团队协作能力，具备让多人同时调查，各司其职，加快响应处置进度；

- 持续监控能力，具备更细颗粒度监控，动态发现更隐蔽的恶意行为；


简而言之，系统化、工程化的事件调查响应平台，既要满足人工手动调查的灵活自主，也要具备工具脚本的自动化能力，这种“半自动化”交互能力，才能最好提高事件调查能力，实现1+1>2的效果。

#### 3.4 小结

每个组织防守方都有各自的应急响应的方式方法，如何去提高应急响应能力是每个防守方队员应该思考的问题，笔者也只是是简单列出一些例子以及个人看法。或许也有人会觉得工具脚本也挺好用的，为啥还要去折腾平台建设呢？在不组织环境下，就会有不同的需求，没有绝对说一种方式方法”一定合适“或者“一定不合适”。

作为经历了一些在对抗中各种坑的苦逼事件响应人员来说，建立平台化的响应能力，才是最优提升事件调查分析能力的方式。这也就是为什么有“Velociraptor”的原因。下面将简要介绍“Velocirapotor”作为一个事件响应平台的要素要点，以及简要介绍是如何完成事件调查响应的几个基本概念。



### 4. Velociraptor实战

为了解决上述事件调查响应的一些局限和问题，我们引入了Velociraptor这个事件调查响应框架平台，用于提高整体的事件调查响应能力。

Velociraptor中文翻译名称是“迅猛龙”，是一款开源的、运行在终端主机的、监控、取证、响应平台，能够有效进行广泛的数字调查取证和网络安全事件调查。主要有三个核心功能：

1. 收集-Collect：收集终端状态、历史记录、日志等各类信息
2. 监控-Monitor:  持续收集终端事件，并将所有事件集中存储在服务端；
3. 狩猎-Hunt: 针对在线的终端执行Hunt操作，主动搜寻可疑活动；

Velociraptor是一款CS框架设计的，需要通过部署在终端主机上的Agent来完成事件日志的收集、监控和Hunt。

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/deployment.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">Velociraptor Deployment</div></center>

Velociraptor运行方式有点类似HIDS或者EDR等终端主机防护工具，与这些工具不同的是，Velociraptor提供的是一个终端主机“可见性”能力，并将这个能力以框架方式完全开放给用户，用户可以基于这个框架能力，按需自定义终端主机监控和事件调查能力。

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/4-1.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">Velociraptor Collection</div></center>

`VQL`是Velociraptor开发的一种类似SQL的语言，是Velociraptor的核心。用户可以利用这个语言来完成收集、监控、狩猎等各种安全能力，除此之外，`VQL`还是`Artifact`(工件)功能实现的基础模块，支持编程开发。

Velociraptor高度可扩展，尤其是在安全事件响应功能上，用户既可以直接使用官方或者社区提供的工件做到开箱即用，也可以基于框架的能力快速定制开发贴合业务场景的响应模块，是防守方不可多得的利器。

本文不尝试详细讲解Velociraptor配置明细或者使用细节，因为官方网站有很详细的教程文档；下面会从实际使用的角度简要介绍Velociraptor实战中关心的基本概念。

#### 4.1 环境与部署

Velociraptor提供强大的数据采集、监控功能，但要发挥出其调查能力的独特优势，还需要组织内的终端主机环境做好预配置。

首先，调查响应的最基础元素还是日志，完整的日志是高质量完成调查响应动作的必备条件，所以首先应该开启终端主机的日志功能，尽可能在本地开启详细的日志记录模式。例如操作系统日志关键的进程网络日志、应用日志。在Linux主机环境可以开启Audit审计记录；在Windows终端可以开启“创建进程”、“网络连接审计”等记录功能，条件允许的环境可以直接部署Sysmon。

其次，就是需要考虑Agent部署问题，是全量部署还是按需部署，这需要组织内根据环境实际情况来实施的：

- **全量部署**：全量部署是指在整个终端主机覆盖安装Agent。全量覆盖需要很大的人力成本，尤其是终端主机数量超过一定数量，对后端服务器的要求将高（另外，目前Velociraptor官方提供的服务器分布式部署方式仍在实验状态）：
  - 优势：全网终端主机Agent全覆盖，一旦发生安全事件快速响应；
  - 劣势：全量覆盖耗费人力成本，其成本不亚于在生产环境重新部署一套类HIDS；

- **按需部署**：按需部署是指以安全事件为驱动的，给有需要进行事件调查的终端主机推送Agent，一般这种推送是借助类似于“终端主机集中管控平台”来实现，当然也可以手动单独安装。
  - 优势：以事件驱动，无需全量覆盖，降低人力成本；
  - 劣势：对组织内IT基础环境要求将高，需要组织环境内有“终端主机集中管控平台”；

从实际运营来看，按需部署是更科学的选择，毕竟不是所有终端主机都需要时刻准备着安全事件调查，非调查期间运行着Agent不但占用终端主机的资源，更是给后台服务器的维护带来巨大的挑战。

在按需部署情况下，如果要做到快速响应，首先要考虑的是如何自动化推送Agent；借助终端主机集中管控平台下发Agent是其中最优的选择；其次，也可以将Agent和配置文件打包成分发安装包，或者制作分发安装脚本，实现一行命令就实现上线的效果。

例如，在实际部署中，Agent分发模式可以有如下几种：

- 脚本：自研一键Agent上线脚本，包含PowerShell、Bash环境；
- MSI： 自行生成Windows MSI分发包，Velociraptor文档有指引;
- RPM：通过Velociraptor Linux执行文件和配置文件，可直接客户端的RPM安全包；

上述提到的分发包和脚本可以直接以HTTP方式开放在内网，在需要安装时可以这样操作：

- PowerShell安装：
    ```powershell
    PS C:\>Invoke-Expression(New-Object Net.WebClient).DownloadString("http://[URL]/[URI]/VelociraptorStart.ps1")
    ```
- CMD安装:
    ```bash
    C:\>Powershell -NoProfile -NonInteractive -ExecutionPolicy Bypass Invoke-Expression(New-Object Net.WebClient).DownloadString("http://[URL]/[URI]/VelociraptorStart.ps1")
    ```
    
- Bash安装
    ```bash
    [root@localhost ~]# curl -s -L 'http://[URL]/[URI]/VelociraptorStart.sh' | bash
    ```
- MSI安装
    ```
    C:\>msiexec /i http://[URL]/[URI]/Velociraptor.msi /quiet
    ```
- RPM安装
    ```
    [root@localhost ~]# rpm -ivh http://[URL]/[URI]/Velociraptor.rpm 
    ```

这些安装方式都是可以使用命令行方式一键执行，除了可以手动执行以外，也可以将这些命令预置在终端主机集中管控平台中，通过平台下发。

另外，Velociraptor官方也提供一种通过域控实现的Agentless方案，但在实际操作中，通过域分发对环境要求比较高，还不如上述几种方式来的简单可控。

当然，实际的调查环境或许会更恶劣，例如待调查的终端主机是完全离线环境，此时就无法实现在线就调查响应，这时候可以使用Velociraptor离线收集模式，生成一个离线调查可执行文件，将该执行文件放到待调查主机上执行，执行后会生成一个压缩包，再将该压缩包上传到服务端并导入，就可以按照在线方式一样完成事件调查了。

综上，要想使用Velociraptor做好事件调查响应，终端主机日志功能要打开；在非全量部署的情况下，如果快速分发Agent、如何快速取到调查的信息，也是应急响应质量的关键因素。从实际运行效果来看，基于上述几种Agent分发模式已经能够满足绝大部分场景的需要。 

#### 4.2 VQL

`VQL`是Velociraptor中最类似SQL的一种编程、检索语言，其设计灵感是来源于OSQuery这个工具，但比OSQuery实现更多的功能。 基本语法如下：

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/vql_structure.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">VQL格式</div></center>

`VQL`提供`Plugin`和`Function`分别用来数据查询和逻辑实现的功能组件。根据作用的环境不同可以将`Plugin`和`Function`划分成十多种类型，例如基础的`VQL`支持以下的`Plugin`和`Function`：

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/vql_plugin_function.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">VQL Plugin Function</div></center>

这些原生支持的插件、函数，可以应对不同的数据处理要求，从实际使用情况来看，已有的这些已经能够满足90%事件响应需要了。

下面举几个例子，可以更直观了解`VQL`是怎么一个运作过程。



**日志排查**：

安全事件调查查看日志是通过`VQL`来实现的。例如，通过`pslist()`这个`Plugin`来列出所有进程: 

```sql
SELECT * from pslist()
```

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/pslist.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">VQL查询进程列表</div></center>

如果数量太多，也可以直接过滤出感兴趣的进程名，还可以通过`timestamp()`这个`Function`把`Createtime`格式化成更可的时间格式:

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/pslist-02.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">VQL过滤进程列表</div></center>



**执行命令**

如果需要在目标机器执行命令，也可以使用`VQL`中的`execve()`这个`Function`来实现：

```sql
SELECT * FROM execve(argv=["/bin/bash", "-c", "ls -al"])
```

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/execve-01.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">VQL执行命令</div></center>

可以看到，这个`VQL`语句实现在目标终端执行了`ls -al`这个命令，这个结果输出结果不太友好，我们可以简单优化下，例如这样：

```sql
SELECT split(string=Stdout, sep="\n") as Result FROM execve(argv=["/bin/bash", "-c", "ls -al"])
```

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/execve-02.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">VQL执行结果处理</div></center>

可以看到，`VQL`的`Function`不仅可以对结果做过滤，还可以对结果的数据进一步处理，这样就保证`VQL`可以有更多上下文，实现更多的能力。



**管理客户端**

Velociraptor提供的是一个`VQL`框架，很多能力并没有出现在操作界面上，而是通过`VQL`或者`Artifact`来实现，例如，可以用`clients()`这个`Plugin`来实现对终端的状态的查看；`VQL`可以这样写：

```
SELECT 
    client_id AS ClientId,
    agent_information.version AS AgentVer, 
    os_info.system AS System,
    os_info.hostname AS Hostname, 
    os_info.release AS Release,
    os_info.fqdn AS FQDN,
    timestamp(epoch=int(int=first_seen_at)) AS FirstSeen, 
    timestamp(epoch=int(int=last_seen_at/1000)) AS LastSeen, 
    last_ip AS LastIP, 
    join(array=labels, sep=" | ") as Labels  
from clients()
ORDER BY FirstSeen Desc
```

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/notebook-clients.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">VQL管理客户端</div></center>

如果想基于这个结果进行更多的操作，例如给特定的终端打上`label`标记，可以使用`label`这个`Function`来给实现，那么可以在这个`VQL`上继续编辑操作：

```sql
LET CLIENTS <= SELECT client_id AS ClientId,
	  agent_information.version AS AgentVer,
	  os_info.system AS System,
	  os_info.hostname AS Hostname,
	  os_info.release AS Release,
	  os_info.fqdn AS FQDN,
	  timestamp(epoch=int(int=first_seen_at)) AS FirstSeen,
	  timestamp(epoch=int(int=last_seen_at/1000)) AS LastSeen,
	  last_ip AS LastIP,
	  join(array=labels, sep=" | ") as Labels
  FROM clients()

LET LabelHosts <= SELECT * FROM foreach(
  row=TargetHostname,
  query={
	  SELECT ClientId, Hostname, Release, FQDN, FirstSeen, LastSeen, LastIP, Labels from CLIENTS WHERE Hostname = "MyHostname"
  }
)

SELECT ClientId, label(labels="MyLabel", op="set", client_id=ClientId) FROM LabelHosts
```

这样就可以给主机名是`MyHostname`这个终端打上`MyLabel`这个标记。

可以看出，`VQL`除了支持单行的语句，还是支持多行的语句操作，可以定义变量，可以定义子检索语句，这样就能极大扩展的`VQL`的使用场景。 

上述这几个例子只是简单介绍`VQL`常用的几个用法，`VQL`还有更多的玩法，因篇幅关系这里就不详细展开了。



#### 4.3 Notebook

`Notebook`是Velociraptor交互式"记事本"，是交互式调查基础，也可以理解成事件调查的一个工作台，可以用于跟踪处理事件调查的多个检索，也可以用来做后期数据处理。

一个`Notebook`可以由任意个`Cell`组成，`Cell`支持`VQL`、`Markdown`两种的类型，并且还会根据`Cell`内容提供不同的前端展示内容，例如时间轴功能。 

在Velociraptor中，`Notebook`出现在以下几个地方：

1. 每个终端的`Collect`动作的结果都会自动生成一个`Notebook`，且`Notebook`的内容可以在`Artifact`中提前预定义；
2. 在`Hunt`管理页面中也存在一个`Notebook`，这个`Notebook`可以对Hunt的内容进行检索处理，也就是对同一批Hunt的结果批量处理；
3. 全局有一个单独`Notebook`功能模块，全局`Notebook`可以根据需要任意新建`Notebook`；

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/notebooks.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">Notebook的位置</div></center>

不同位置的`Notebook`能力是一样的，但在使用上会有不同的侧重点。例如， 全局的`Notebook`可以引用系统内工件执行结果，就可以实现跨终端的关联分析，甚至可以用来生成事件调查的Report报告；终端的`Collection`动作产生的`Notebook`一般适合对结果的二次处理，或者通过在`Artifact`中预定义`Notebook`内容实现`Collect`的自动化处理。 

总的来说，`Notebook`可以实现包含但不限于以下几个场景：

- 检索结果的二次解析处理
- 聚合多个结果
- 批量数据结果操作
- 客户/服务端管理

`Notebook`是一个Velociraptor很重要的功能，这种`Notebook`提供的“交互性”既能够有足够灵活性保证“Case By Case”事件得以顺利进行，也能够通过编写`Artifact`预定义方式对结果自动化处理。 

#### 4.4 Artifact

`Artifact`中文翻译是“人工制品”或者简称“工件”，在Velociraptor是用户用`VQL`语言编写的自定义的检索、处理等动作的内容脚本，是Velociraptor实现调查的能力的关键组件。 

`Artifact`是以YAML文件格式保存的，`VQL`语句写在YAML其中的一个Section中。Velociraptor前端提供编辑这个YAML的界面，可以直接添加，也可以批量导入，甚至可以在服务端启动参数中指定文件目录的路径实现启动即自动加载。

`Artifact`可以定义执行的输入参数，编写`VQL`，也可以编写`Notebook`。 例如，下面是一个Velociraptor官方自带的读取Windows Eventlog 的`Artifact`内容，

```yaml
name: Windows.EventLogs.Evtx
description: |
  Parses and returns events from Windows evtx logs.
  ......
  Inspired by others in `Windows.EventLogs.*`, many by Matt Green (@mgreen27).

author: Chris Hendricks (chris@counteractive.net)
precondition: SELECT OS FROM info() WHERE OS = 'windows'
parameters:
  - name: EvtxGlob
    default: '%SystemRoot%\System32\winevt\Logs\*.evtx'
  - name: SearchVSS
    description: "Search VSS for EvtxGlob as well."
    type: bool
  - name: StartDate
    type: timestamp
    description: "Parse events on or after this date (YYYY-MM-DDTmm:hh:ssZ)"
  - name: EndDate
    type: timestamp
    description: "Parse events on or before this date (YYYY-MM-DDTmm:hh:ssZ)"
  - name: PathRegex
    default: "."
    type: regex
  - name: ChannelRegex
    default: "."
    type: regex
  - name: IDRegex
    default: "."
    type: regex

sources:
  - query: |
      // expand provided glob into a list of paths on the file system (fs)
      LET fspaths <=
          SELECT FullPath FROM glob(globs=expand(path=EvtxGlob))
          WHERE FullPath =~ PathRegex

      // function returning list of VSS paths corresponding to path
      LET vsspaths(path) =
          SELECT FullPath FROM Artifact.Windows.Search.VSS(SearchFilesGlob=path)
          WHERE FullPath =~ PathRegex

      // function returning parsed evtx from list of paths
      LET evtxsearch(pathList) =
          SELECT * FROM foreach(
            row=pathList,
            query={
              SELECT *,
                timestamp(epoch=int(int=System.TimeCreated.SystemTime)) AS TimeCreated,
                System.Channel as Channel,
                System.EventRecordID as EventRecordID,
                System.EventID.Value as EventID,
                FullPath
              FROM parse_evtx(filename=FullPath)
              WHERE
                if(condition=StartDate,
                   then=TimeCreated >= timestamp(string=StartDate),
                   else=true)
                AND if(condition=EndDate,
                       then=TimeCreated <= timestamp(string=EndDate),
                       else=true)
                AND Channel =~ ChannelRegex
                AND str(str=EventID) =~ IDRegex
            }
          )

      // only de-duplicate using GROUP BY when searching VSS
      // de-duplicate file-by-file to reduce memory impact
      SELECT * FROM if(condition=SearchVSS, then={
          SELECT * FROM foreach(row=fspaths, query={
            SELECT * FROM evtxsearch(pathList={
                SELECT FullPath FROM vsspaths(path=FullPath)
            })
            GROUP BY EventRecordID,Channel
          })
        }, else={
          SELECT * FROM evtxsearch(pathList={SELECT FullPath FROM fspaths})
        })

```

工件执行的结果是这样的：

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/eventlog-01.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">Eventlog</div></center>

这个`Artifact`实现的功能是读取evtx文件，并将evtx文件的内容解析成更易读的json格式。仔细查看`Artifact`内容是可以发现，`Artifact`核心能力还是`VQL`的能力，上述这个例子就是使用了`VQL`中的`glob`遍历文件和`parse_evtx`两个`Plugin`提供能力来实现“读取Eventlog”这个功能。

其实这个`Artifact`还可以进一步优化，例如，通常在事件调查中，特别关心“进程创建记录”(EventID:4688)，因此我们可以直接在这个`Artifact`基础上添加EventID过滤，并将“进程创建”事件单独筛选出来，并做优化关键字段显示处理，这样就可以更直观将关键信息展示。要实现这个效果，可以添加`EventID=4688`的过滤，并添加一个预定义的`Notebook`提取需要关注的字段：

```yaml
name: Windows.EventLog.Process

description: |
  Parses and returns events from Windows evtx logs.  
  EventID: 4688 

precondition: SELECT OS FROM info() WHERE OS = 'windows'
parameters:
  - name: StartDate
    type: timestamp
    description: "Parse events on or after this date (YYYY-MM-DDTmm:hh:ssZ)"

  - name: EndDate
    type: timestamp
    description: "Parse events on or before this date (YYYY-MM-DDTmm:hh:ssZ)"

sources:
  - name: Process History - EventID 4688
    query: |
      // expand provided glob into a list of paths on the file system (fs)
      LET fspaths <=
          SELECT FullPath FROM glob(globs=expand(path="%SystemRoot%\\System32\\winevt\\Logs\\Security.evtx"))

      // function returning parsed evtx from list of paths
      LET evtxsearch(pathList) =
          SELECT * FROM foreach(
            row=pathList,
            query={
              SELECT 
                // System,
                // EventData,
                // Message,
                timestamp(epoch=int(int=System.TimeCreated.SystemTime)) AS TimeCreated,
                // System.Channel as Channel,
                // System.EventRecordID as EventRecordID,
                System.EventID.Value as EventID,
                // FullPath,
                System.Computer as Computer,
      
                // 4688 EventData Detail
                EventData.SubjectUserSid as SubjectUserSid,
                EventData.SubjectUserName as SubjectUserName,
                EventData.SubjectDomainName as SubjectDomainName,
                EventData.SubjectLogonId as SubjectLogonId,
                EventData.NewProcessId as NewProcessId,
                EventData.NewProcessName as NewProcessName,
                // EventData.TokenElevationType as TokenElevationType,
                EventData.ParentProcessName as ParentProcessName,
                EventData.ProcessId as ProcessId,
                EventData.CommandLine as CommandLine,
                EventData.TargetUserSid as TargetUserSid,
                EventData.TargetUserName as TargetUserName,
                EventData.TargetDomainName as TargetDomainName,
                EventData.TargetLogonId as TargetLogonId,
                EventData.MandatoryLabel as MandatoryLabel
      
              FROM parse_evtx(filename=FullPath)
              WHERE
                if(condition=StartDate,
                   then=TimeCreated >= timestamp(string=StartDate),
                   else=true)
                AND if(condition=EndDate,
                       then=TimeCreated <= timestamp(string=EndDate),
                       else=true)
                AND EventID = 4688
            }
          )

      SELECT * FROM evtxsearch(pathList={SELECT FullPath FROM fspaths})
    notebook:
      - type: VQL
        template: |
          /*
          --------------------------------------------------------------
          ## 进程启动历史: EventID:4688 
          */
          SELECT TimeCreated, SubjectUserName, ProcessId, ParentProcessName, CommandLine, NewProcessId, NewProcessName 
          FROM source() 
          WHERE NOT SubjectUserName =~ "\\$"
          ORDER BY TimeCreated Desc
          Limit 50
```

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/eventlog-process.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">Eventlog</div></center>

可以看出，修改后的`Artifact`更好看的数据展示，也更贴近我们的调查的实际场景。

另外，在实际事件调查中，Velociraptor提供的`Plugin`未必能够所有场景，这时候就需要用到`VQL`加载第三方工具的能力了。

例如，如下例子就是将Windows Sysinternals套件中的Autoruns封装到`Artifact`中，目标终端执行该`Artifact`就相当于在本地执行该工具，并将结果优化展示给调查人员：

```yaml
name: Windows.Sysinternals.Autoruns
description: |
  Uses Sysinternals autoruns to scan the host.

  Note this requires syncing the sysinternals binary from the host.

tools:
  - name: Autorun_x86
    url: https://live.sysinternals.com/tools/autorunsc.exe
    serve_locally: true

  - name: Autorun_amd64
    url: https://live.sysinternals.com/tools/autorunsc64.exe
    serve_locally: true

precondition: SELECT OS From info() where OS = 'windows'

parameters:
  - name: All
    type: bool
    default: Y
  - name: Boot execute
    type: bool
  - name: Codecs
    type: bool
  - name: Appinit DLLs
    type: bool
  - name: Explorer addons
    type: bool
  - name: Sidebar gadgets (Vista and higher)
    type: bool
  - name: Image hijacks
    type: bool
  - name: Internet Explorer addons
    type: bool
  - name: Known DLLs
    type: bool
  - name: Logon startups (this is the default)
    type: bool
  - name: WMI entries
    type: bool
  - name: Winsock protocol and network providers
    type: bool
  - name: Office addins
    type: bool
  - name: Printer monitor DLLs
    type: bool
  - name: LSA security providers
    type: bool
  - name: Autostart services and non-disabled drivers
    type: bool
  - name: Scheduled tasks
    type: bool
  - name: Winlogon entries
    type: bool
  - name: ToolInfo
    type: hidden
    description: Override Tool information.

sources:
  - query: |
      LET Flags = '''Option,Name
      *,All
      b,Boot execute
      c,Codecs
      d,Appinit DLLs
      e,Explorer addons
      g,Sidebar gadgets (Vista and higher)
      h,Image hijacks
      i,Internet Explorer addons
      k,Known DLLs
      l,Logon startups (this is the default)
      m,WMI entries
      n,Winsock protocol and network providers
      o,Office addins
      p,Printer monitor DLLs
      r,LSA security providers
      s,Autostart services and non-disabled drivers
      t,Scheduled tasks
      w,Winlogon entries
      '''

      -- The options actually selected
      LET options = SELECT Option FROM parse_csv(accessor="data", filename=Flags)
        WHERE get(field=Name)

      LET os_info <= SELECT Architecture FROM info()

      // Get the path to the binary.
      LET bin <= SELECT * FROM Artifact.Generic.Utils.FetchBinary(
              ToolName= "Autorun_" + os_info[0].Architecture,
              ToolInfo=ToolInfo)

      // Call the binary and return all its output in a single row.
      LET output = SELECT * FROM execve(argv=[bin[0].FullPath,
            '-nobanner', '-accepteula', '-t', '-a',
            join(array=options.Option, sep=""),
            '-c', -- CSV output
            '-h', -- Also calculate hashes
            '*'   -- All user profiles.
      ], length=10000000)

      // Parse the CSV output and return it as rows. We can filter this further.
      SELECT * FROM if(condition=bin,
      then={
        SELECT * FROM foreach(
          row=output,
          query={
             SELECT * FROM parse_csv(filename=utf16(string=Stdout),
                                     accessor="data")
          })
      })
```

在这个工件中添加了`tools`这个section，这个section定义了Autoruns的执行文件。这个`Artifact`在可以在使用前需要将可执行文件上传到Velociraptor服务端，完成这个操作之后，每次在目标终端上执行该`Artifact`都会自动化下发这个可执行文件并按照`Artifact`预定义的命令参数执行，执行效果如下：

<center><img style="border-radius:0.3125em;box-shadow:0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"     src="https://raw.githubusercontent.com/chiww/chiww.github.io/master/assets/Build-network-security-platform/autoruns.png"><br><div style="color:orange; border-bottom: 1px solid #d9d9d9;display:inline-block;color:#999;padding:2px;">Autoruns工件</div></center>

可以看到Autoruns工具已经在目标终端上执行，以更直观的方式呈现给调查人员。

`Artifact`还支持嵌套，意味着可以针对一个场景去设计`Artifact`集合，而又保留每个独立`Artifact`的功能的完整性，例如我们可以做一个邮件钓鱼的专属`Artifact`专门应对钓鱼攻击事件的调查，在这个`Artifact`内把钓鱼相关的`Artifact`都集合进去，一旦这个某个终端主机需要钓鱼邮件攻击的调查，执行这一个`Artifact`就可以了。

基于某个场景将不同`Artifact`集合到一个`Artifact`中，就如同前面提到“工具脚本”，但这种“工具脚本”具有更灵活的执行方式、更集中的管理维护的优点。 从另外一个角度看，场景化的`Artifact`也相当于把邮件钓鱼这个事件调查动作SOP化，调查流程经验沉淀到平台中。

正是由于`VQL`和`Artifact`高度自由可自定制性，用户可以非常方便开发多种多样的`Artifact`，也得益于`Artifact`是YAML结构化文本定义，更容易实现分享获取，所以官方也提供这样[Artifact Exchange](https://docs.velociraptor.app/exchange/)这个平台，收录了开源社区用户共享的`Artifact`。这也无形中扩大了`Artifact`流通性，提高Velociraptor使用场景，构建一个更完善的开源分享生态。 

#### 4.5 小结

Velociraptor除了上面的介绍的`Collect`功能，还有`Hunt`、`Monitor`功能，因篇幅限制，就不多赘述了，但这些功能的使用逻辑都离不开`VQL`、`Notebook`和`Artifact`几个基础模块。

在实际使用中，Velociraptor也给事件调查响应工作带来积极变化：

1. 事件调查和应急响应流程变得更加可靠有序，调查响应的SOP有了可落地的技术平台；
2. 事件调查响应这项工作有了可运营基础，团队工作经验能够得到有效沉淀；

实践证明，从2022年初开始全面建设基于Velociraptor事件调查响应平台以来，在这接近一年的时间里，Velociraptor在重保对抗中、以及日常安全响应中都发挥了重要作用。



### 6. 后记

之所以会有应急响应平台这个概念，是因为在最近3年不断红蓝对抗中，作为防守方，在可用工具武器上，始终感觉比攻击方落后很多。尤其是现在各种攻击框架、各种工具武器顺手拈来，一旦被攻击队找到一个突破口就如入无人之境，而防守方除了断网、停业务似乎找不到更好的方法，甚至连红队怎么进来的、进来以后做了什么都不知道。攻击溯源也只能靠着人肉一项一项排查，或者执行没有针对性的固定化检测脚本，这些与攻击方各种武器相比起来简直天壤之别。就好像在战场中，双方都进入到激烈阵地战肉搏中，攻击队拿着机关枪在突突突对我方一顿扫射，而我们防守方只能拿着“小步枪”躲在一个角落弱弱还击，连对手在哪里突突、突突哪里都不清楚，始终处于被动和挨打的状态。 

于是那时候就在想，难道蓝队就没有一个能打的工具、系统或者平台能够治治这些嚣张的红队？直到找到Velociraptor这个工具，经过一年多的研究、建设、运营，现整个平台已初具规模，能够在一定程度上有效提升整体的事件调查和响应能力。

未来，基于Velociraptor事件调查响应平台，还需要进一步扩展能力，包括提高检测的覆盖面、提高检测效率，也相信未来Velociraptor在开源社区支持下功能会更加完善。
























