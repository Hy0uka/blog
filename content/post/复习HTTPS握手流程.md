+++
title = '复习HTTPS握手流程及相关扫描器'

description = "Handshake"

tags = [ "网络基础设施" ]

date = 2024-11-15T10:29:53+08:00

+++

#### 0x01常见抓包工具Charles/Fiddler/Wireshark区别

为什么Charles等抓包工具或浏览器控制台看到的HTTPS是明文的？

HTTPS其实相当于HTTP+TLS,HTTP是明文操作，所有的加解密操作都在TLS这边。

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/11/7284e8bfe1801b74890299a2b1a6e594.png)

浏览器控制台能看到明文HTTPS是因为浏览器是应用层，所以浏览器的输出都是TLS解密过的。

Charles能看到是因为Charles抓包的时候有一个信任证书的操作，类似于中间人攻击。

Fiddler和Charles都是用于抓取应用层协议等上层的数据，都是建立连接成功以后的数据。而Wireshark可以抓取所有协议的数据包，因为直接读取网卡数据。需要抓取HTTPS建立连接成功前的过程，所以选择Wireshark。

Curl命令只发送一个请求，如果用浏览器打开百度，页面的各种资源也会发送请求，对分析造成很多不必要的数据包。

#### 0x01为什么指数数字签名证书需要hash一次

给证书明文签名和给明文的hash签名其实区别不大，但是对hash签名能让非对称加密的速度快很多。

#### 0x02传输流程

1. **三次握手**
2. **客户端发送client_hello**
3. **服务端发送server_hello**
4. **服务端发送证书**
5. **服务端发送Server Key Exchange**
6. **服务端发送Server Hello Done**
7. **客户端发送Client_key_exchange+Change_cipher_spec+Encrypted_handshake_message**
8. **服务端发送New Session Ticket**
9. **服务端发送Change_cipher_spec**
10. **服务端发送Encrypted_handshake_message**
11. **完成密钥协商，开始发送数据**
12. **完成数据发送，四次TCP挥手**：客户端FIN，服务端ACK，服务端FIN，客户端ACK。
