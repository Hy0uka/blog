+++
title = '主流APT基础设施特征整理'

description = "绝密"

tags = [ "经验之谈" ]

date = 2025-03-27T15:00:26+08:00
draft = true

+++

# 朝鲜Kimsuky

- 使用网易XSS 0day![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2025/03/7e6a1de4bd124fd78450c8057084b5a6.png)
- 喜欢打下跳板网站后使用开源邮件项目PGPMailer(https://github.com/PHPMailer/PHPMailer)来发送钓鱼邮件，**这点与APT37一致**。邮件的X-Mailer一项的值会是PHPMailer 5.2.14。
- 基础设施由xampp搭建，这点**与之前的朝鲜APT组织一致**。![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2025/03/66ca3568f7e1666e86227ea816fc48c3.png)
- 使用过之前被APT37入侵过的韩国医院的证书![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2025/03/2ee741b2a238006ba522428aaf1e0d7e.png)
- 

# 台湾毒云藤

- 喜欢用AS-CHOOPA的VPS

# 越南海莲花

- 喜欢入侵边界设备作为跳板，再进行下一步攻击。之前打了新华三的路由器。

# 美国NSA

- 通过Windows中的Internet Connecting Sharing失陷的内网节点，内网中的其它Windows机器通过配置到指定的这个共享段中来实现外网的连接。这样配置流量都会从svchost.exe -k netsvcs  -p -s sharedaccess这个ics服务出去。

# 印度CNC

- 具有印度背景，最早与白象组织共用基础设施和 github账号，直到疫情爆发初期 CNC 渐渐拥有自己的特马和插件，随即将其独立出来，该团伙的特马样式多变，免杀效果较好，在南亚地区的APT 组织中较为少见。
- https://ti.qianxin.com/blog/articles/operation-sea-elephant-the-dying-
  walrus-wandering-the-indian-ocean-cn/
