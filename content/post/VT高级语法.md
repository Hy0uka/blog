+++
title = 'VT高级语法'

description = "记录"

tags = [ "Cyber threat hunting" ]

date = 2024-11-21T10:24:27+08:00

+++

## VT高级语法：
**Network activity**：behaviour_network:"202.117.170.240"

**Network activity**：behaviour_network:"/vpnchecker.php"

**Filesystem operations**：behaviour_files:"C:\\\\Users\\\\<\USER>\\AppData\\Local\\Temp\\pnw2vuyz.e5s.ps1"

**Processes execution**：behaviour_processes:"powershell.exe -ep bypass -File C:\\Users\\Johnson\\AppData\\Local\\Temp\\4.ps1"

**不直接相关的恶意行为组合**：behaviour_network:tinyurl.com and behaviour_processes:powershell

**寻找可信签名但反病毒引擎检出率高的样本**：signature:"© Microsoft Corporation. All rights reserved." tag:signed not (tag:invalid-signature or tag:revoked-cert) fs:2022-01-01+ p:5+

**寻找带有特定cookie的url**：entity:url cookie:"njnmsdkfsdfbiuonsdkfnsdfl"

**对安卓包的搜索**：androguard_package:org.xmlpush.v3

**文件名**：type:document name:"Standard Chartered" p:1+

**URL**：entity:url title:"JP Morgan" p:5+

**email**：type:email have:email_attachment tag:exploit

**IOS野生恶意文件**：(type:apple OR type:mac) have:in_the_wild p:5+

itw:cdn.discordapp.com p:5+

**entity：分为file、url、ip、domain、collection**

**itw**：指“In The Wild”，主要描述一种病毒活动状态，即该病毒已在互联网上广泛传播，并对日常的计算机运行产生实际影响。因此，当我们说一个病毒是"In The Wild" （缩写为ITW）的，这意味着它不仅仅是在实验室环境中的假设或可能性，而是在现实世界中已经存在并对用户和系统构成威胁了。

**对特定组织进行查询**：entity:collection(name:apt28 or tag:apt28 or name:Sofacy or tag:Sofacy)

p:5+表示至少被5个反病毒引擎监测到
