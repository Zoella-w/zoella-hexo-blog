---
title: 浏览器解析url过程
date: 2024-10-14 14:15:10
tags:
    - 面经
    - 计算机网络
    - url
categories:
    - 计算机网络
      - url
---

## 1. URL的格式

协议表示浏览器应当使用的访问方法，包括："http：""ftp：""file：""mailto："。
比如访问Web服务器使用HTTP协议，访问本地服务器使用FILE协议，访问FTP服务器使用FTP协议。

图1 列举了常见的几种URL，访问自标不同，URL的写法也不同。
例如在访问Web服务器和FTP服务器时，URL中包含服务器的域名和要访问的文件的路径名等，而发邮件的URL则包含收件人的邮件地址。此外，URL还可能包含用户名、密码、服务器端口号等信息。

图1.1：URL的各种格式
![图1.1](https://pica.zhimg.com/80/v2-e72ee2cb036bb9fa21e2817228a43e42_1440w.webp?source=2c26e567)

## 2. URL的组成元素

图1.2：Web浏览器解析url的过程
![图1.2](https://picx.zhimg.com/80/v2-42f11c5b60c90af055cc232d71d15fd0_1440w.webp?source=2c26e567)

图1.3：路径名为/dir/file1.html的文件
![图1.3](https://pic1.zhimg.com/80/v2-204e5983725b0bf4ea61bef9c66d8d99_1440w.webp?source=2c26e567)

## 3. 省略文件名的情况

#### (a) 以"/"结尾：`lab.glasscom.com/dir/`

以"/dir/"结尾代表/dir/后面本应该有的文件名被省略了。
我们会在服务器上事先设置好文件名省略时要访问的默认文件名，大多情况下是index.html或者default.htm。因此，像前面这样省略文件名时，服务器就会访问其中的默认文件。

#### (b) 域名+"/"：`lab.glasscom.com/`

以"域名+/"结尾表示访问一个名叫"/"的目录（根目录）；
由于省略了文件名，会访问/index.html之类的文件。

#### (c) 域名：`lab.glasscom.com`

以域名结尾时，就代表访问根目录下事先设置的默认文件，也就是/index.html之类的文件。

#### (d) 路径结尾无"/"：`lab.glasscom.com/whatisthis`

**先文件，后目录。**
如果Web服务器上存在名为whatisthis的文件，则将whatisthis作为文件名来处理；
如果存在名为whatisthis的目录，则将whatisthis作为目录名来处理。

### 备注

- URL：Uniform Resource Locator，统一资源定位符。
- FTP：File Transfer Protocol，文件传送协议。是在上传、下载文件时使用的协议（也指使用FTP协议传送文件的程序）。
- HTTP：Hypertext Transfer Protocol，超文本传送协议。
- 根目录："/"目录 表示最顶层的目录 "根目录"。
