---
layout: post
title:  "HITB2018AMS Badge Reverse Note"
date:   2018-04-29 22:40:34 +0800
---
### 0x0 前言
演讲前一天拿到Badge，花了小半个小时尝试逆向，因其他事情作罢。今晚无聊，再次尝试，基本搞定，过程其实比较简单。

### 0x1 正文
Badge上除了展示会议信息外，另外还有一个secret menu。上下左右ok + esc 6个按键分别对应6个字符的输入。
![]({{ site.imageurl }}/20180492_secret_menu.jpg)
考虑到这个板子的代码已经在github开源，尝试在源码中查找线索。

源码src目录有一个hitb_secret.h文件，展示了这个menu的逻辑。当然，默认的‘000000’并不是实际板子中的code。
![]({{ site.imageurl }}/20180429_secret_code.png)

除了src目录外，hex目录有一个bin文件。如果这个文件只是sample生成的bin，那么只能从硬件逆向入手。不过，我猜测会议方应该不希望把badge搞坏，故继续尝试bin文件逆向。

加载bin文件到ida，直接搜索关键字符串"ACCESS DENIED!!! :("。此时，并没有ref的code代码，应该是加载基地址不对。通常，裸机嵌入式设备会对物理地址有一个统一的划分。下载对应的芯片手册，得到如下划分:
![]({{ site.imageurl }}/20180492_address.png)

演讲前一天误把0x20000000作为加载基地址，发现字符串仍然没有ref，尝试人工及脚本搜索后，发现并不能解决问题，受到安卓保护的影响，以为会存在复杂的混淆及逻辑，限于时间当时遂作罢。不过，当时发现很多ref到0x08000000的基地址时并没有细想。虽然对M系列不熟悉，但这个bin应该有不同的加载区段。今晚直接把base改到0x08000000后，直接ref相关代码逻辑。
![]({{ site.imageurl }}/20180492_show_secret_menu.png)

找到code: ”LRDROC“。其实可以更暴力点，因为比较字符串都存在rodata，编译生成都在一起。直接搜索"Code: "和"ACCESS DENIED!!! :("附近的字符串数据，并根据按键编码即可直接找到这个code。输入code得到如下二维码。
![]({{ site.imageurl }}/20180492_qrcode.jpg)

扫描之后，通过短链接跳转到 [http://hardware.reverser.io/hitb2018ams/challange/](http://hardware.reverser.io/hitb2018ams/challange/)

![]({{ site.imageurl }}/20180492_binary_site.png)

遗憾的是，这个服务器已经登陆不上去了。下载对应binary，直接main函数f5找到逻辑。
![]({{ site.imageurl }}/20180492_check.png)
![]({{ site.imageurl }}/20180492_file.png)

大致看了下代码，输入逻辑很简单，依赖了字符库界面，应该是一个字符界面的输入。检验逻辑一般，无花无混淆，至此不再向下逆向。

### 0x2 结尾
不一定非要硬着上去逆逻辑，换个思路借助编译器的处理逻辑可能达到事半功倍的效果。
