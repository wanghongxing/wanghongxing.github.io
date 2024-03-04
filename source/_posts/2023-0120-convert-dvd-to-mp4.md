---
title: macbook把dvd视频提取转换成mp4视频
date: 2023-01-20 20:09:11
tags:
  - ffmpeg
  -	dvd
  - mp4
  -	h264	
  - h265
---



有很多旧的dvd，都是早年给小孩的刻的DVD碟片。时间久了，碟片机都扔了，光驱也快淘汰了。当下最方便的还是用手机看，决定弄一份出来存到硬盘上，然后转换成方便手机观看的格式。

手机支持观看的格式，基本都是h264或者h265编码的mp4文件。找了很多工具，都是收费或者免费只能导出一半，这个钱还是不愿意花，自己用弄 ffmpeg。

本方案需要有dvd光驱或者dvd writer，如果老旧光盘读取有问题，就需要dvd player，那就应该走video capture方案，这个后面有时间买个dvd player后再弄（ps：吐槽一下，买的那些个绿色的dvd rewriter盘片，基本都读不出来，只有清华紫光的有保障）。

### 软件安装

```
brew install ffmpeg
```

### 复制

把碟片查到硬盘，其实就是把dvd碟片中的VIDEO_TS目录下内容复制到硬盘上，因为我有好多碟片，就一张一张复制，每张都改成碟片刻录的日期。

其中有一张碟片的VTS_01_4.VOB复制不出来了。

看网上说需要用ddrescue来挽救。

GNU ddrescue是一个用于磁盘、CD-ROM与其他数字存储媒体的资料恢复工具。其将原始存储区块从一个设备或文件复制到另一个，同时以智能方式处理读取错误，透过从部分读取的区块中截取尚称良好的扇区来最小化资料损失。 GNU ddrescue是用C++编程语言编写的，并以开源软件的形式提供，最初于2004年发布。

```
brew install ddrescue
```

Locate the drive using `diskutil list`. 

```
/dev/disk3 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                            PRJ_20090118           *4.4 GB     disk3
```

Unmount the disk 

```
diskutil unmount /dev/disk3   
```

Start a rescue operation of the disk into an image. Make sure the location of `Rescue.dmg` is replaced with your desired location.

```
sudo /usr/local/bin/ddrescue -v -n -c 4096 /dev/disk3 Rescue.dmg Rescue.log
```

*注：因为死了，就强制kill &把光驱电源线，重新插入后发现disk3 变成了disk2。不知道什么鬼*

上面个直接就死给我看了

```
sudo /usr/local/bin/ddrescue -c 4096 -d -r 3 -v /dev/disk2  Rescue.dmg Rescue.log
```

提示我 `ddrescue: Direct disc access not available.`。

查了半天，说macos不支持direct access，可以通过raw方式；

再查raw方式，发现macos的raw格式disk是通过/dev/rdisk*来的。

```
sudo /usr/local/bin/ddrescue  -r1 -b2048  /dev/rdisk2  Rescue.dmg Rescue.log
```

经过一晚上折腾，放弃了，太慢了，12个小时才恢复10多M的坏块。



### 试一试编码

先从网上找几个使用例子

#### h264

[How to convert DVD to mp4 with ffmpeg](https://dev.to/ko31/how-to-convert-dvd-to-mov-with-ffmpeg-fli)  [Ko Takagi](https://dev.to/ko31) *Posted on 2021年4月17日 Updated on 2022年8月8日*

```
ffmpeg -i VTS_01_1.VOB -b:v 1500k -r 30 -vcodec h264 \
-strict -2 -acodec aac -ar 44100 -f mp4 convert.mp4


ffmpeg -i "concat:VTS_01_1.VOB|VTS_01_2.VOB|VTS_01_3.VOB" \
-b:v 1500k -r 30 -vcodec h264 -strict -2 -acodec aac -ar 44100 -f mp4 convert.mp4


```

就是单个转换和多个拼接一起转换；这兄弟指定了视频、音频码率。

试一下：

```bash
ffmpeg -i VTS_01_1.VOB -b:v 1500k -r 30 -vcodec h264 \
-strict -2 -acodec aac -ar 44100 -f mp4 VTS_01_1-1500k.mp4


-rwxrwxrwx   1 whx  staff   977M  9 21  2008 VTS_01_1.VOB
-rw-r--r--   1 whx  staff   194M  1 20 20:26 VTS_01_1-1500k.mp4

```

期间有提示错误

```bash

[mpeg @ 0x7fd5c0816400] stream 1 : no PTS found at end of file, duration not set
[ac3 @ 0x7fd5c081ca00] incomplete frame8kB time=00:16:28.91 bitrate=1628.7kbits/s dup=4945 drop=0 speed=2.92x

```

我估计应该是文件应该一起来转换。

不过看文件大小，vob文件977M ,生成的mp4文件 194M。

其中`bitrate=1628.7kbits/s`应该指码率是1628k。

再试一下4个一起

```bash
ffmpeg -i "concat:VTS_01_1.VOB|VTS_01_2.VOB|VTS_01_3.VOB|VTS_01_4.VOB" \
-b:v 1500k -r 30 -vcodec h264 -strict -2 -acodec aac -ar 44100 -f mp4 all-h264-1500k.mp4

-rwxrwxrwx   1 whx  staff   977M  9 21  2008 VTS_01_1.VOB
-rwxrwxrwx   1 whx  staff   977M  9 22  2008 VTS_01_2.VOB
-rwxrwxrwx   1 whx  staff   977M  9 22  2008 VTS_01_3.VOB
-rwxrwxrwx   1 whx  staff   977M  9 22  2008 VTS_01_4.VOB
-rwxrwxrwx   1 whx  staff   170M  9 22  2008 VTS_01_5.VOB
-rw-r--r--   1 whx  staff   781M  1 20 21:00 all-h264-1500k.mp4
```



#### h265

我想试试不限制码率，只指定编码方式

```bash
ffmpeg -i "concat:VTS_01_1.VOB|VTS_01_2.VOB|VTS_01_3.VOB|VTS_01_4.VOB" \
-vcodec libx265 all-x265.mp4

Output #0, mp4, to 'all-x265.mp4':
  Metadata:
    encoder         : Lavf58.76.100
  Stream #0:0: Video: hevc (hev1 / 0x31766568), yuv420p(tv, bt470bg, top coded first (swapped)), 720x576 [SAR 16:15 DAR 4:3], q=2-31, 25 fps, 12800 tbn
    Metadata:
      encoder         : Lavc58.134.100 libx265
    Side data:
      cpb: bitrate max/min/avg: 0/0/0 buffer size: 0 vbv_delay: N/A
  Stream #0:1: Audio: aac (LC) (mp4a / 0x6134706D), 48000 Hz, stereo, fltp, 128 kb/s
    Metadata:
      encoder         : Lavc58.134.100 aac
      
      
-rw-r--r--   1 whx  staff   265M  1 20 21:38 all-x265.mp4

```

H265编码比较费 CPU，反正慢的要死。期间看到

```
frame=85892 fps= 48 q=34.4 size=  231936kB time=00:57:15.47 bitrate= 553.1kbits/s speed=1.93x

```

貌似码率是553k，最终文件大小265M还是比较喜人。

**but**：播放的时候quick time player不识别。

查询说Quicktime Player和iOS不再支持hev1 tag的mp4/mov。

回看输出`Stream #0:0: Video: hevc (hev1 / 0x31766568)`，这儿应该指输出hev1.



二者大致有如下不同：

'hvc1' stores all parameter sets inside the MP4 container below the sample description boxes.
'hev1' stores all parameter sets in band (inside the HEVC stream).
我决定试试，只转一个vob，免得太慢。

```bash
ffmpeg -i "concat:VTS_01_1.VOB" \
-vcodec libx265  -vtag hvc1 VTS_01_1-x265-hvc1.mp4

Output #0, mp4, to 'VTS_01_1-x265-hvc1.mp4':
  Metadata:
    encoder         : Lavf58.76.100
  Stream #0:0: Video: hevc (hvc1 / 0x31637668), yuv420p(tv, bt470bg, top coded first (swapped)), 720x576 [SAR 16:15 DAR 4:3], q=2-31, 25 fps, 12800 tbn
    Metadata:
      encoder         : Lavc58.134.100 libx265
    Side data:
      cpb: bitrate max/min/avg: 0/0/0 buffer size: 0 vbv_delay: N/A
  Stream #0:1: Audio: aac (LC) (mp4a / 0x6134706D), 48000 Hz, stereo, fltp, 128 kb/s
  
  
-rwxrwxrwx   1 whx  staff   977M  9 21  2008 VTS_01_1.VOB
-rw-r--r--   1 whx  staff   194M  1 20 20:26 VTS_01_1-1500k.mp4
-rw-r--r--   1 whx  staff    73M  1 20 22:25 VTS_01_1-x265-hvc1.mp4

```

压缩率完美。

```
ffmpeg -i "concat:VTS_01_1.VOB|VTS_01_2.VOB|VTS_01_3.VOB|VTS_01_4.VOB" \
-vcodec libx265  -vtag hvc1  all-x265.mp4


```



闲着没事就试试，弄完了再试试别的

```bash
ffmpeg -codecs |grep EV |grep H.26

 DEV.L. flv1                 FLV / Sorenson Spark / Sorenson H.263 (Flash Video) (decoders: flv ) (encoders: flv )
 DEV.L. h261                 H.261
 DEV.L. h263                 H.263 / H.263-1996, H.263+ / H.263-1998 / H.263 version 2
 DEV.L. h263p                H.263+ / H.263-1998 / H.263 version 2
 DEV.LS h264                 H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10 (encoders: libx264 libx264rgb h264_videotoolbox )
 DEV.L. hevc                 H.265 / HEVC (High Efficiency Video Coding) (encoders: libx265 hevc_videotoolbox )

```

EV 就是过滤视频编码。

####  不指定那么多繁琐的参数试试看

开始测试h264有那么多参数，试着少点参数转h264试试看

```
ffmpeg -i "concat:VTS_01_1.VOB|VTS_01_2.VOB|VTS_01_3.VOB|VTS_01_4.VOB" \
-vcodec h264 all-h264.mp4

frame=64350 fps= 83 q=28.0 size=  443648kB time=00:42:53.82 bitrate=1412.0kbits/s speed=3.34x

-rw-r--r--   1 whx  staff   636M  1 20 22:02 all-h264.mp4

```

h264码率大概在1412k，播放效果不错。





### 批量转换

因为碟片多，让我一个一个的复制显然不是程序猿的作风，弄脚本~



```bash
##这个是查找所有的VOB文件然后转换成h265编码的mp4文件
find ./ -name '*.VOB' -exec bash -c 'ffmpeg -i $0  -vcodec libx265  -vtag hvc1   ${0/VOB/mp4}' {} \;


```



### 试试GPU

电脑是2015年的macbook pro 15寸 ,CPU 2.5 GHz 四核Intel Core i7 ,显卡AMD Radeon R9 M370X 2 GB。貌似可以试试GPU性能。

#### 试试h264

```
ffmpeg -i VTS_02_1.VOB -c:v h264_videotoolbox  whx-h264-gpu.mp4
```

速度贼啦啦快，但是效果惨不忍睹，基本上可到的都是马赛克。



换成1M码率，试试看：

```
 ffmpeg -i VTS_02_1.VOB -c:v h264_videotoolbox  -b:v 1000k whx-h264-gpu-1m.mp4

```

速度贼快，效果还好



```
 ffmpeg -i VTS_02_1.VOB -c:v h264_videotoolbox  -b:v 500k whx-h264-gpu-500k.mp4

```

换成500k码率，速度更快，效果又不行了。



换成1500k码率：

```
 ffmpeg -i VTS_01_1.VOB -c:v h264_videotoolbox  -b:v 1500k whx-h264-gpu-1500k.mp4

```

速度挺快，效果很好。

#### 试试h265

```
ffmpeg -i VTS_01_1.VOB -c:v h265_videotoolbox  -vtag hvc1  whx-x265-gpu.mp4

[hevc_videotoolbox @ 0x7f87d8058a00] Error: cannot create compression session: -12908
[hevc_videotoolbox @ 0x7f87d8058a00] Try -allow_sw 1. The hardware encoder may be busy, or not supported.
Error initializing output stream 0:0 -- Error while opening encoder for output stream #0:0 - maybe incorrect parameters such as bit_rate, rate, width or height

```



```
ffmpeg -i VTS_02_1.VOB -c:v hevc_videotoolbox  -b:v 1000k  -vtag hvc1  whx-x265-gpu.mp4

[hevc_videotoolbox @ 0x7f78eb80f200] Error: cannot create compression session: -12908
[hevc_videotoolbox @ 0x7f78eb80f200] Try -allow_sw 1. The hardware encoder may be busy, or not supported.
Error initializing output stream 0:0 -- Error while opening encoder for output stream #0:0 - maybe incorrect parameters such as bit_rate, rate, width or height


```

这样h265的gpu编码就失败了。

应该是这个显卡太老了不支持h265的硬解码。



#### gpu的优缺点

优点：速度贼快

缺点：文件太大





#### 使用gps批量命令

虽然硬盘占用大，但是速度快，决定公用gpu

```
find ./ -name '*.VOB' -exec bash -c 'ffmpeg -i $0 -c:v h264_videotoolbox  -b:v 1500k ${0/VOB/mp4}' {} \;


```



### 摄像机里面的视频文件处理

摄像机里还有大量拍的视频，都是MPEG2编码的，为了用方便用手机，就复制到硬盘上，然后转换成h265。

```

##这个是查找所有的 MPG 文件然后转换成h265编码的mp4文件
find . -name '*.MPG' -exec bash -c 'ffmpeg -i $0  -vcodec libx265  -vtag hvc1   ${0/MPG/mp4}' {} \;


```

#### 试着把生成的文件加上日期后缀

摄像机里面复制出来的 MPG 文件都是数字名称没法看出来具体年月，，但是复制出来的在电脑上的创建日期是保留的，修改一下脚本，把年月日记录在转换后的文件名上。

```
#!/bin/sh
set +x
convertFile(){
	prefix=`date  -r ${0} "+%Y年%m月%d日%H点%M分%S"`
	#fname= ${0/.MPG/-${prefix}.mp4}
   echo "文件名： $1 ${prefix}"
  ffmpeg -i $0  -vcodec libx265  -vtag hvc1  ${0/.MPG/-${prefix}.mp4}
}
export -f convertFile
find . -name '*.MPG' -exec bash -c 'convertFile ${0}' {} \;


```





修改视频码率，设置缩放后的视频大小和码率

```
ffmpeg -i 浩之宝视频2024-1-9.mp4   -r 15 -b 350k  -vcodec libx265 -vtag hvc1 浩之宝视频2024-1-9-350.mp4
ffmpeg -i 浩之宝视频2024-1-9.mp4 -vf scale=iw*.8:ih*.8 -r 15 -b 350k  -vcodec libx265 -vtag hvc1 浩之宝视频2024-1-9-350k.mp4
ffmpeg -i 澎众店视频2024-1-9.mp4 -vf scale=iw*.8:ih*.8 -r 15 -b 350k  -vcodec libx265 -vtag hvc1 澎众店视频2024-1-9-3501k.mp4
ffmpeg -i 澎众店视频2024-1-9.mp4  -r 15 -b 350k  -vcodec libx265 -vtag hvc1 澎众店视频2024-1-9-350k.mp4




ffmpeg -i 1彭众.mp4  -r 15 -b 200k  -vcodec libx265 -vtag hvc1 1彭众-350k.mp4
ffmpeg -i 2深蓝.mp4  -r 15 -b 200k  -vcodec libx265 -vtag hvc1 2深蓝-350k.mp4



```

