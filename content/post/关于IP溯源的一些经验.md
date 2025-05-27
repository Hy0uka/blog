+++
title = '关于IP溯源的一些经验'

description = "威胁情报"

tags = [ "溯源" ]

date = 2025-05-27T11:31:56+08:00

+++

*https://github.com/curated-intel/Attribution-to-IP*

| 方法                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| PDNS                     | 显示该IP的历史解析，并允许您识别指向该IP的域。               |
| 域DNS记录                | 包括A、PTR、MX、TXT和CNAME记录。PTR记录可以显示主机名；TXT和MX记录可以提供有关域所有者或提供商的线索。 |
| IP WHOIS                 | 查询Register和网络块注册信息，可以直接识别注册IP范围的实体或ISP。 |
| 开放端口和运行服务       | 扫描IP以查找可访问的端口和服务，Banner抓取可以显示软件、版本，有时还会显示组织名称。 |
| X509证书                 | 检查在开放的HTTPS端口上提供的SSL证书，证书通常包含域、电子邮件地址或者组织名称。 |
| ASN                      | ASN描述IP地址块的所有权，帮助识别网络背后的ISP、托管提供商或大型组织。 |
| IP地理定位               | 将IP映射到大致的物理位置。                                   |
| 边界网关协议BGP          | BGP公告告知谁在路由IP块，并确认管理地址的ASN。               |
| 内容存档                 | 该IP上的网站的存档内容或IP上的服务的屏幕截图，可能会显示网站更改之前的早期品牌、联系信息或者域。 |
| 手动浏览                 | 直接在浏览器中访问IP。                                       |
| Google Dorking           | 高级搜索语法。                                               |
| 链接到Tor节点、VPN或代理 | 检查Tor、VPN或Proxy节点的列表，表明IP不属于特定的最终用户，而是隐私或匿名网络的一部分。 |
| 云存储                   | Cloud Storage存储桶可能会包含与IP所有者相关的配置文件、日志、构建工作或者命名约定。 |
| IP行为                   | 蜜罐网络和众包报告网站还可以根据观察到的与蜜罐和受害者 IP 的交互来帮助辨别 IP 在做什么以及谁拥有它。 |
| 安全 TXT 记录            | 直接查询 IP 可以显示主机 IP 上 .well_known/security.txt 或 /security.txt 路径处的安全 TXT 文件。安全 TXT 文件也存在 DNS 实现，这些文件可以作为 DNS TXT 记录找到。 |
| 加密货币网络节点         | 用于支持加密货币挖掘和账本记录活动的公开分布式网络的 IP 列表。 |
| 服务提供商 IP 范围       | 报告为属于 Cloud、CDN 和其他服务提供商的 IP 地址列表。       |

**域名系统 (DNS) 记录是存储在 DNS 服务器上的指令，用于将域名（例如 `www.google.com`）与各种类型的信息相关联，最常见的是 IP 地址。下面是常见的DNS记录类型。**

**A (Address Record)**

- **功能**：将域名或子域名指向一个 IPv4 地址。这是最基本和最常见的记录类型。
- **示例**：`example.com. A 192.0.2.1` (表示 `example.com` 指向 IPv4 地址 `192.0.2.1`)

**AAAA (IPv6 Address Record)**

- **功能**：将域名或子域名指向一个 IPv6 地址。随着 IPv6 的普及，这种记录越来越重要。
- **示例**：`example.com. AAAA 2001:0db8:85a3:0000:0000:8a2e:0370:7334`

**CNAME (Canonical Name Record)**

- **功能**：将一个域名（别名）指向另一个规范的域名（Canonical Name）。当你想让多个域名指向同一个服务器，并且希望主服务器 IP 更改时只修改一个 A 记录时非常有用。CNAME 记录的值必须是另一个域名，不能是 IP 地址。
- **示例**：`www.example.com. CNAME example.com.` (表示 `www.example.com` 是 `example.com` 的别名)

**MX (Mail Exchange Record)**

- **功能**：指定负责接收该域名电子邮件的邮件服务器及其优先级。优先级数字越小，表示邮件服务器的优先级越高。
- **示例**：`example.com. MX 10 mail.example.com.` (表示发送到 `example.com` 的邮件应由 `mail.example.com` 处理，优先级为 10)

**TXT (Text Record)**

- **功能：**允许管理员在 DNS 中存储任意文本信息。常用于多种目的，例如：
- **SPF (Sender Policy Framework)**：验证邮件服务器是否有权为该域名发送邮件，防止邮件欺诈。
- **DKIM (DomainKeys Identified Mail)**：通过加密签名验证邮件的真实性和完整性。
- **DMARC (Domain-based Message Authentication, Reporting, and Conformance)**：结合 SPF 和 DKIM，提供更强大的邮件认证策略。
- **域名所有权验证**：许多服务（如 Google Search Console, Microsoft 365）会要求你添加特定的 TXT 记录来证明你对域名的所有权。
- **示例**：`example.com. TXT "v=spf1 mx -all"` (一个 SPF 记录示例)

**NS (Name Server Record)**

- **功能**：指定负责管理该域名或子域名解析的权威 DNS 服务器。一个域名通常有多个 NS 记录，指向不同的 DNS 服务器以实现冗余。
- **示例**：`example.com. NS ns1.nameserver.com.`

**SOA (Start of Authority Record)**

- **功能**：包含关于 DNS 区域的重要管理信息，如区域的主 DNS 服务器、区域管理员的电子邮件地址、区域文件的序列号以及区域刷新、重试、过期和否定缓存的计时器。每个 DNS 区域必须有一个 SOA 记录。
- **示例内容**：主服务器名、管理员邮箱、序列号、刷新间隔、重试间隔、过期时间、最小 TTL。

**SRV (Service Record)**

- **功能**：指定特定服务（如 SIP、XMPP、LDAP）的主机名和端口号。允许服务在非标准端口上运行，并提供负载均衡和故障转移机制。
- **示例**：`_sip._tcp.example.com. SRV 10 5 5060 sipserver.example.com.` (表示 SIP 服务可以通过 TCP 协议在 `sipserver.example.com` 的 5060 端口找到，优先级10，权重5)

**PTR (Pointer Record)**

- **功能**：用于反向 DNS 查询，即将 IP 地址解析回域名。主要用于邮件服务器验证和其他反向查找场景。通常在 `in-addr.arpa` (IPv4) 或 `ip6.arpa` (IPv6) 域中设置。
- **示例**：`1.2.0.192.in-addr.arpa. PTR example.com.` (表示 IP 地址 `192.0.2.1` 反向解析为 `example.com`)

**CAA (Certification Authority Authorization Record)**

- **功能**：允许域名所有者指定哪些证书颁发机构 (CA) 有权为其域名颁发 SSL/TLS 证书，以增强安全性。
- **示例**：`example.com. CAA 0 issue "letsencrypt.org"` (表示只有 Let's Encrypt 有权为 `example.com` 颁发证书)

查询Whois的时候，如果返回Privacy Protect说明信息被隐藏，可尝试历史WHOIS存档（如微步，WhoisHistory API）。 

PDNS：使用SecurityTrails或Censys查询IP历史绑定域名常用于： 识别被黑域名迁移路径，发现攻击者丢弃的C2服务器旧域名。’

通过网络测绘扫描IP开放端口，结合Banner信息判断用途。端口22+OpenSSH可能为老旧的Linux服务器。

SSL证书关联分析：提取IP的SSL证书SHA256指纹→全网搜索相同证书的IP/域名→若存在证书复用则可穿透CDN。

ASN网络区块溯源：访问BGP.TOOLS查询IP的AS号，通过AS关联信息确定主机商。(这个一般FOFA就做了)。

BGP路由劫持检测：通过RIPE Stat或BGP Stream监控IP路由变更：正常情况IP应该再所属AS内广播，若发现在其它AS内广播则可能遭遇BGP劫持。

什么是BGP：BGP（**Border Gateway Protocol**）是互联网的核心路由协议，用于在**自治系统（AS，Autonomous System）**之间交换路由信息。每个 ISP、大型组织或云服务商都有自己的 AS。某 IP 段的归属、路由方向，都是通过 BGP 协议“宣传”出来的。

BGP 劫持指的是：某个自治系统（AS）恶意或错误地向互联网宣称某 IP 段归它所有，从而把原本属于其他 AS 的网络流量吸引到自己那里。Eg：IP 段 `8.8.8.0/24` 本来是 Google 的。某个攻击者 AS 宣称：“我也有 `8.8.8.0/24`”，或者更具体的子网：`8.8.8.0/25`(/25表示前25位为网络地址，具体细分到2的7次方大的子网)。因为 BGP 优先选择更精确的前缀（更长的子网掩码），全世界访问 `8.8.8.8` 的流量就可能会被“吸引”到攻击者那里。

BGP 劫持的目的：**中间人攻击**（MITM）：拦截流量进行监听或篡改；**拒绝服务（DoS）**：导致原始网络瘫痪或隔离；**垃圾邮件/网络诈骗**：临时使用被劫持 IP 地址发送恶意内容；**掩盖真实身份**：通过临时接管 IP 避免追踪。

代码仓库情报挖掘：Github高级搜索语法"192.168.1.1" language:python "password" AND "10.0.0.0/8" extension:env 将某些服务所属IP明文卸载Github配置文件可能遭遇批量扫描。

#### 高阶对抗

Netflow流量图谱搭建：Elastic Stack + Logstash，从高频通信IP（如每5分钟连接乌克兰某IP）和异常协议比例（如80%流量为ICMP可能存在隧道隐匿） 两个维度分析。

蜜罐网络行为捕获：部署Cowrie/Conpot蜜罐，记录攻击者：输入的命令序列（`wget恶意脚本`、`curl C2地址`） ，使用的工具特征（如Metasploit默认User-Agent）。

泄露数据关联匹配：整合DeHashed+SpyCloud数据，实现：IP与泄露账号关联（攻击IP曾登录`admin@victim.com`），密码重用追踪（攻击者邮箱密码与某社工库记录一致） 。

#### 收费方案

| Method 方法                                     | Description 描述                                             | Source(s) 来源                                               |
| ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| NetFlow NetFlow 网络                            | NetFlow captures metadata about traffic flows (source/destination IPs, ports, protocol, timestamps) and can be used to track communication patterns, peer IPs, and usage behavior. Repeated traffic between an IP and a corporate network could indicate ownership. NetFlow 捕获有关流量（源/目标 IP、端口、协议、时间戳）的元数据，并可用于跟踪通信模式、对等 IP 和使用行为。IP 和公司网络之间的重复流量可能表示所有权。 | [Team Cymru Pure Signal](https://www.team-cymru.com/) Cymru Pure Signal 团队 |
| Breach Data 泄露数据                            | Breach Data includes leaked credentials, config files, and service records from compromised systems and could be used to directly connect an IP to an email address, domain, or username and imply corporate ownership. 泄露数据包括来自受感染系统的泄露凭据、配置文件和服务记录，可用于将 IP 直接连接到电子邮件地址、域或用户名，并暗示公司所有权。 | [SpyCloud](https://spycloud.com/) / [AmIBreached](https://amibreached.com/) / [Intelx](https://intelx.io/) / [Dehashed](https://dehashed.com/) / [Socradar](https://socradar.io/) / [Hudsonrock](http://hudsonrock.com/) |
| Internet Scraping Databases Internet 抓取数据库 | Internet scraping databases collect, store, and index citations from the darkweb, cybercrime forum posts, messaging services, paste sites, legit news sites, and technical blogs, among other sources. They may have data on IP addresses such as where they were first and last mentioned and in what context. Internet 抓取数据库从暗网、网络犯罪论坛帖子、消息服务、粘贴站点、合法新闻站点和技术博客等来源收集、存储和索引引文。他们可能有 IP 地址的数据，例如他们第一次和最后一次被提及的位置以及在什么上下文中。 | [Recorded Future](https://www.recordedfuture.com/) / [Anomali ThreatStream](https://www.anomali.com/products/threatstream) / [Intel471](https://intel471.com/solutions/dark-web-monitoring-and-investigations) / [Flashpoint](https://flashpoint.io/ignite/cyber-threat-intelligence/) 记录的未来 / Anomali ThreatStream / Intel471 / Flashpoint |

