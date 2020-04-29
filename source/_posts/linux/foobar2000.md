---
title: foobar2000插件音效
categories: home
date: 2012-12-13 19:00:13
tags:
---

　　准备搬迁blog的时候发现网易博客要寿终就寝了，12年，挺感慨的，既然第一篇文章就写这吧。

![keyboard](/img/blogoff.png)

---
正传

一、Foobar设置和优化
　　除了硬件要过硬外，我们也需要使用软件的方式来提高音效，我们可以使用：
　　1. 象PowerDVD和WinDVD等专业软件，都有着很好很强大的功能，可惜他们是收费的。
　　2. 象千千静听等免费软件，虽然免费，但音质有待提高。
　　3. 象PotPlayer和KMPlayer等多媒体播放软件，虽然免费，可惜他们消耗系统资源较多。

　　笔者最终推荐Foobar，Foobar2000的定位是专业数字音频播放工具，它有着精良的播放品质和极少的资源占用。可惜官方下载的Foobar并不直接支持播放DTS，当然，Foobar也有众多的修改版，但他们也有消耗系统资源较多的缺点。为此，我们只能自己定制Foobar，为了能播放DTS，需要为Foobar安装必要的插件，如下：

　　1. foo_input_dts      // 其实，只要这一个插件就可以播放DTS了，下一个用来改善音质。
　　2. foo_convolve        // 用来模拟胆机(真空管功率放大器)，使声音听起来温暖细腻。及其采样文件。

　　对于播放DTS、APE、FLAC等无损音乐，个人觉得还是不用其它插件为好，纯净才是真的美！

　　首先把下载的插件解压复制到D:\Program Files\foobar2000\components下，然后运行Foobar2000。点击工具栏上的“File(文件) → Preferences（参数设置）”，在左列设定中选“Playback（回放）→ DSP Manager”。在右边的 Available DSPs（可用的DSP） 清单中选取 DTS Decoder和Convolver（回旋混响器），将它移到 Active DSPs（当前使用的DSP）内，如图1。

　　然后点击“Convolver（回旋混响器）→Configure selected”，在控制面板中找到“Impulse file（脉冲文件）”旁的...按钮。我们可以通过下载一些impulse文件来载入，如果载入后显示load ok就表示载入成功了。如图2。

 　　在foobar2000中，我常使用的impulse文件是Hotstudio.wav，再加上平时常用的Resampler（重采样）等DSP功能的作用下，Foorbar呈现出来的声音, 整体表现颇温暖细腻，很有胆机的味道 。总之，配合音乐类型采用不同的脉冲文件就可以享受到高级的音响混音效果了。
　　这样就可以用Foobar播放DTS了，如图3。

二、优化硬件驱动

　　大家都明白 Windows 出于保护底层硬件的目的，一款播放软件要使声卡发出声音，需要经过很多软件层的过滤，只有在系统认为安全的情况下才可以通过。这样做虽然保护了系统，但也散失了声卡的一些表现力，耳朵听起来可以用一个字来形容，就是不够 “透”！那有没有一种技术可以使播放软件直接和声卡本身打交道，答案是有的，这就是 ASIO （Audio Stream Input Output —— 音频流输入输出）技术。ASIO 让播放软件直接和声卡接触而绕过其它软件层。有的声卡的驱动程序支持 ASIO 和 KS ，但有的可能不支持或仅仅支持它们中的一种。如果你的声卡支持ASIO，最好直接使用ASIO输出。ASIO是Steinberg提出来的比较新的音频流输入输出接口，一般用在对实时性要求很高的专业场合，对声卡的要求更高。

　　安装ASIO只需要简单的进行下面几步：
　　1. 到http://www.asio4all.com/下载 ASIO4ALL驱动程序并安装好。
　　2. 下载Foobar 2000 ASIO support组件，并安装到Foobar 2000的components中。
　　3. 之后就是在Foobar 2000中修改设置，首先添加ASIO虚拟驱动。如图4。

　　4. 之后单击图4的“Configure”按钮，使用ASIO4ALL提供的控制面板进行相关参数的设置，调整采样频率，缓存等。一般，使用默认设置即可。如图5。
　5. 然后在Foobar 2000中修改输出设置，让其使用ASIO来输出。如图
　　现在，听听看是不是比采用默认的 DS: 主声音驱动程序 时表现力要 “透”了许多？ 
三、高品质音乐下载

　　这里也提供一些DTS 下载地址：
　　1. verycd
　　2. 风云音乐谷
　　3. 捌零音乐论坛 
　　4. 炫音音乐

四、Foobar歌词显示设置

　　顺便提一下，如果你还想让Foobar显示歌词，再装一个迷你歌词(MiniLyrics)就可以了
Impulse file的选择
在Esther Kinderspiele人声的表现上，Urei 1178最好，润泽有张力，高音更清晰，不过低音不够结实。SPL Goldmike感觉有些沙哑。SPL Charisma综合表现介于上述两者之间，不过在低频范围有些含糊，不够通透。JoeMeek SC2的动态范围略胜一筹，中音厚实。

　　在Dans avant de tomber 1分37秒左右的提琴表现上，JoeMeek略微发闷，DBX 160SL高音更亮，SPL GoldMike清晰但感觉没有上述二者厚实。

　　至于SPL TubeVitalizer系列，陷于我的听力所限 —— 天天被SENNHEISER MX500的低音轰击已经快不行了呜呜，以后有钱了买个舒尔或者因特美的用用，或者Bayer Dynimac的东东 ……