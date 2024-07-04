+++
title = 'Hugo+CloudFlare搭建博客指南'

description = "崭新架构的安全博客"

tags = [ "技术" ]

date = 2024-06-27T18:02:59+08:00

+++

1.首先创建Github库

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705001618.png)

2.安装hugo：

```
https://github.com/gohugoio/hugo/releases
```

3.下载后记得将hugo.exe路径加入系统环境变量，测试是否安装成功：

```
hugo version
```

4.使用hugo创建博客目录

```
hugo new site blogname
```

5.在系统中创建博客目录，并`git init`，配置好git的config文件。

6.选择hugo主题，并使用git clone将主题下载到本地，本网站选择的主题是`https://github.com/heyeshuang/hugo-theme-tokiwa.git`，第一眼看到便想到莫奈的《日出·印象》。

7.新建文章

`hugo new post/first-post.md`

8.在hugo网站的根目录下执行`hugo`进行编译，启动本地预览,打开网址 http://localhost:1313/ 可以进行预览`hugo server -D`

9.部署到GitHub

```
git add .
git commit -m "xxx"
git push https://github.com/username/projectname master(main)
```

10.在Godaddy或其他网站购买域名

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705005703.png)

11.将域名托管到Cloudflare，左边导航栏进入Websites

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705005944.png)

add site

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705010027.png)

记得选free plan

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705010033.png)

随后在Godaddy设置域名DNS为Cloudflare的DNS服务器地址

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705010351.png)

随后等待DNS解析成功，邮箱将会收到邮件通知

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705010400.png)

在Cloudflare创建pages并连接到Git，选择最开始在Git中新建的blog目录

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705011021.png)

随后博客文件将部署到Cloudflare上

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705011256.png)

最后自定义域名，即将域名DNS记录指向Pages

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705011654.png)

踩的坑

1. Hugo server -D的意思是忽略draft=true，即draft属性为true的md文件也会被渲染编译。折磨了一天的问题是“为什么本地能够编译出文章，但是部署到Github上就消失了？”因为实际部署的时候draft=true的文件是不显示的。

2. 域名的CNAME要解析到项目的别名/Aliases，一键交给Cloudflare添加的CNAME可能不是实际的Aliases，需要手动修改DNS添加上master。

   ![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/微信图片_20240705012026.png)