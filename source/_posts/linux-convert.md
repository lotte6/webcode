---
title: convert 转换图像格式和大小
categories: tech
tag: hide
date: 2018-12-19 00:29:08
tags:
---

convert 转换图像格式和大小，模糊，裁剪，驱除污点，抖动，临近，图片上画图片，加入新图片，生成缩略图等。

identify 描述一个或较多图像文件的格式和特性。

composite 根据一个图片或多个图片组合生成图片


举个例子

convert +profile '*' [src]{file}.{ext} -quality 80 -resize '280x140^>' -gravity Center -crop 280x140+0+0 +repage [out]{file}_280x140.{ext}

把一张图片按80的质量去压缩(jpg的压缩参数),同时按图片比例非强制缩放成不超过280x140的图片.居中裁剪280x140,去掉图片裁减后的空白和图片exif信息,通常这种指令是为了保证图片大小正好为280x140


下面对各个指令的含义简要说明


-quality 图片质量,jpg默认99,png默认75


-resize

100x100 高度和宽度比例保留最高值，高比不变

100x100^ 高度和宽度比例保留最低值，宽高比不变

100x100! 宽度和高度强制转换，忽视宽高比

100x100> 更改长宽，当图片长或宽超过规定的尺寸

100x100< 更改长宽 只有当图片长宽都超过规定的尺寸

100x100^> 更改长宽，当图片长或宽超过规定的尺寸。高度和宽度比例保留最低值

100x100^< 更改长宽，只有当图片长宽都超过规定的尺寸。高度和宽度比例保留最低值

100 按指定的宽度缩放，保持宽高比例

 x100 按指定高度缩放，保持宽高比


-gravity NorthWest, North, NorthEast, West, Center, East, SouthWest, South, SouthEast截取用的定位指令,定位截取区域在图片中的方位


-crop 200x200+0+0 截取用的截取指令 ,在用定位指令后,按后两位的偏移值偏移截取范围左上角的像素后,再按前两位的数值,从左上角开始截取相应大小的图片


+repage 去掉图片裁减后的空白

-dissolve 30 设定组合图片透明度dissolve示例

+/-profile * 去掉/添加图片exif信息

下面记录下一些比较复杂一点的指令:
```
convert +profile '*' [src]{file}.{ext} -quality 90 -resize '700>' /data/tony/watermark_1.png -gravity southeast -compose Dissolve -composite [out]{file}_1024x1024.{ext}

convert +profile '*' [src]{file}.{ext} /data/watermark_1.png -gravity southeast -geometry +10+10 -composite [src]{file}.{ext}

convert +profile '*' [src]{file}.{ext} -coalesce -resize '950x135>' [out]{file}_950x135.{ext}

convert +profile '*' [src]{file}.{ext} -resize '650x500>' miff:- | composite +profile '*' -dissolve 30 -gravity southeast /data/tony/watermark_1.png - [out]{file}_650x500.{ext}

convert [src]{file}.{ext} -quality 80 -resize '190>' -background white -gravity center -extent 190x +repage [out]{file}_width190.{ext}
```
【/bin/sh^M: bad interpreter: 没有那个文件或目录】
vim filename
然后用命令
:set ff? #可以看到dos或unix的字样. 如果的确是dos格式的。
然后用
:set ff=unix #把它强制为unix格式的, 然后存盘退出。

```
shell 除法计算
shell计算中使用除法，基本默认上都是整除。
比如：
num1=2
num2=3
num3=`expr $num1 / $num2`
这个时候num3=0 ,是因为是因为expr不支持浮点除法
解决的方法：
num3=`echo "scale=2; $num1/$num2" | bc`
使用bc工具，sclae控制小数点后保留几位
还有一种方法
awk 'BEGIN{printf "%.2f\n",’$num1‘/’$num2‘}'
如果用百分比表示
awk 'BEGIN{printf "%.2f%\n",(’$num1‘/’$num2‘)*100}'

```
convert +profile '*' -size 950x135 xc:#ffffff [src]{file}.{ext} -resize 950x135 -gravity center -compose Src-over -composite [out]{file}_950x135.{ext}
解读：该指令会把gif的动态图拆分成好几张图，

-coalesce
完全定义一个GIF动画序列的每一帧的外观，形成“电影胶片”动画

根据它的-处置元数据覆盖各图像中的图像序列，以再现一个动画中的动画序列中的每个点的外观。有的图片应该是相同的大小，并且为动画继续如预期般GIF动画的工作分配适当的GIF处理设置。这样的帧更容易查看和比高度优化的GIF图像叠加处理。

该动画可以重新优化使用层法“优化”处理后，虽然没有保证恢复的GIF动画优化是比原来更好。

```
gif拿到的只是最后一帧的数据,所以gif的图片在width和height值上有出入
convert +profile '*' tly2.gif -resize "600>" tly2_600.gif
C:\>identify tly2.gif
tly2.gif[0] GIF 341x186 341x186+0+0 8-bit sRGB 256c 191KB 0.000u 0:00.003
tly2.gif[1] GIF 194x173 341x186+128+13 8-bit sRGB 256c 191KB 0.000u 0:00.010
tly2.gif[2] GIF 192x175 341x186+130+11 8-bit sRGB 256c 191KB 0.000u 0:00.013
tly2.gif[3] GIF 80x177 341x186+131+9 8-bit sRGB 256c 191KB 0.000u 0:00.016
...
tly2.gif[30] GIF 164x175 341x186+122+11 8-bit sRGB 256c 191KB 0.000u 0:00.094
tly2.gif[31] GIF 145x172 341x186+122+14 8-bit sRGB 256c 191KB 0.000u 0:00.097
tly2.gif[32] GIF 195x167 341x186+122+19 8-bit sRGB 256c 191KB 0.000u 0:00.100
tly2.gif[33] GIF 196x174 341x186+121+12 8-bit sRGB 256c 191KB 0.000u 0:00.105
```

一、crop_center.sh

如果原图宽>高:按宽*size/高缩放; 如果原图宽<高:按高*size/宽缩放; 并居中裁剪size*size
```

#!/bin/bash

export PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

filename=$1
filename2=$2
size=$3
resize="70"

wh=($(/usr/local/bin/identify -format "%w %h " $1))

if [ ${wh[0]} -gt ${wh[1]} ]
then
resize=$(echo "${wh[0]}*$size/${wh[1]}+1" | bc)
else
resize=$(echo "${wh[1]}*$size/${wh[0]}+1" | bc)
fi

/usr/local/bin/convert +profile "*" -quality 75 -resize ${resize}x${resize} $filename $filename2

/usr/local/bin/convert +profile "*" -gravity Center -crop ${size}x${size}+0+0 $filename2 $filename2
```
使用范例：/usr/local/bin/crop_center.sh [src]{file}.{ext} [out]{file}_small.{ext} 100


二、tony-convert-1.sh

原图按1024x1024>等比缩放，并打水印于右下角；如果宽高都小于300，则直接使用原图
```

#!/bin/sh

# we have 2 arguments: source file name and dest file name
if [ $# -ne 2 ] ; then 
  echo "Usage: tony-convert-1.sh [source path] [dest path]"
  echo "Example: ./tony-convert-1.sh src/1234.jpg out/1234_100x200.jpg"
  exit 1 
fi

srcFile="$1"
dstFile="$2"

index1=`expr index "${srcFile}" /`
index2=`expr index "${srcFile}" .`
file=`expr substr "${srcFile}" \`expr ${index1} + 1\` \`expr ${index2} - ${index1} - 1\``
ext=`expr substr "${srcFile}" \`expr ${index2} + 1\` \`expr length "${srcFile}" - ${index2}\``

#identify image num
picCount=`/usr/local/sbin/bin/bin/identify ${srcFile}|grep -c "${file}.${ext}"`
echo animate pictures: ${picCount}

if [ ${picCount} -gt 1 ] ; then
  echo "No handle for animate pictures"
  cp ${srcFile} ${dstFile} 
  exit 0 
fi

#identify image size
sizeMatrix=`/usr/local/sbin/bin/bin/identify -format "%wx%h %f\n" ${srcFile}`

index1=`expr index "${sizeMatrix}" x`
width=`expr substr "${sizeMatrix}" 1 \`expr ${index1} - 1\``
echo width: ${width}
index2=`expr \`expr index "${sizeMatrix}" ' '\` - ${index1} - 1`
height=`expr substr "${sizeMatrix}" \`expr ${index1} + 1\` ${index2}`
echo height: ${height}

if [ ${width} -lt 300 ] ; then
  if [ ${height} -lt 300 ] ; then
  echo Image size less than 300x300
  cp ${srcFile} ${dstFile}
  exit 0
  fi
fi

if [ `echo ${ext}|tr 'a-z' 'A-Z'` = "BMP" ] ; then
  index1=`expr index "${dstFile}" .`
  dstFile=`expr substr "${dstFile}" 1 \`expr ${index1} - 1\``.jpg 
fi

#convert image/usr/local/sbin/bin/bin/convert +profile '*' ${srcFile} -resize '1024x1024>' /data/tony/watermark_1.png -gravity southeast -compose Dissolve -composite ${dstFile}
```
使用范例:/usr/local/sbin/bin/bin/tony-convert-1.sh [src]{file}.{ext} [out]{file}_1024x1024.{ext}


三、resizeByWidthOrHeight.sh:

宽大于高时：定宽等比缩放 高自适应；宽小于高时：定高等比缩放,宽自适应
```

#!/bin/sh

# we have 2 arguments: source file name and dest file name
if [ $# -ne 3 ] ; then 
  echo "Usage: resizeByWidthOrHeight.sh [source path] [widthxheigth] [dest path]"
  echo "Example: ./resizeByWidthOrHeight.sh src/1234.jpg 150x150 out/1234_150x2150.jpg"
  exit 1 
fi

srcFile="$1"
dstFile="$3"

destSize="$2"
destWidth=${destSize%x*}
destHeight=${destSize#*x}
echo "destSize: ${destWidth}x${destHeight}"

index1=`expr index "${srcFile}" /`
echo "index1: $index1"
index2=`expr index "${srcFile}" .`
echo "index2: $index2"
file=`expr substr "${srcFile}" \`expr ${index1} + 1\` \`expr ${index2} - ${index1} - 1\``
echo "file: $file"
ext=`expr substr "${srcFile}" \`expr ${index2} + 1\` \`expr length "${srcFile}" - ${index2}\``
echo "ext: $ext"
#identify image num
picCount=`/usr/local/sbin/bin/bin/identify ${srcFile}|grep -c "${file}.${ext}"`
echo animate pictures: ${picCount}

if [ ${picCount} -gt 1 ] ; then
  echo "No handle for animate pictures"
  cp ${srcFile} ${dstFile} 
  exit 0 
fi

#identify image size
sizeMatrix=`/usr/local/sbin/bin/bin/identify -format "%wx%h" ${srcFile}`

echo "index1: $index1"
width=${sizeMatrix%x*}
echo width: ${width}
height=${sizeMatrix#*x}
echo height: ${height}

newSize="${destWidth}x${destHeight}"
# width < height
if [ ${width} -lt ${height} ] ; then
  newSize="'x${destHeight}>'"
else
  newSize="'${destWidth}>'"
fi
echo "newSize: ${newSize}"

# resize
resizeCmd="/usr/local/sbin/bin/bin/convert ${srcFile} -quality 100 -resize ${newSize} ${dstFile}"
echo "resizeCmd: ${resizeCmd}"
eval "${resizeCmd}"
```
使用范例：/usr/local/sbin/bin/bin/resizeByWidthOrHeight.sh [src]{file}.{ext} 594x395 [out]{file}_594x395.{ext}


四、tony-convert-2.sh:

缩放规则：

目标图宽<高时:定高等比压缩，宽自适应;

原图宽高比>目标图宽高比时:定高等比压缩，宽自适应;

否则：定宽等比压缩，高自适应

裁剪规则：默认居中，若目标图宽>=高，则居左上角

```
#!/bin/sh

# we have 2 arguments: source file name and dest file name
if [ $# -ne 3 ] ; then 
  echo "Usage: tony-convert-2.sh [source path] [widthxheigth] [dest path]"
  echo "Example: ./tony-convert-2.sh src/1234.jpg 150x150 out/1234_150x150.jpg"
  exit 1 
fi

srcFile="$1"
dstFile="$3"

destSize="$2"
destWidth=${destSize%x*}
destHeight=${destSize#*x}
echo "destSize: ${destWidth}x${destHeight}"

index1=`expr index "${srcFile}" /`
echo "index1: $index1"
index2=`expr index "${srcFile}" .`
echo "index2: $index2"
file=`expr substr "${srcFile}" \`expr ${index1} + 1\` \`expr ${index2} - ${index1} - 1\``
echo "file: $file"
ext=`expr substr "${srcFile}" \`expr ${index2} + 1\` \`expr length "${srcFile}" - ${index2}\``
echo "ext: $ext"
#identify image num
picCount=`/usr/local/sbin/bin/bin/identify ${srcFile}|grep -c "${file}.${ext}"`
echo animate pictures: ${picCount}

if [ ${picCount} -gt 1 ] ; then
  echo "No handle for animate pictures"
  cp ${srcFile} ${dstFile} 
  exit 0 
fi

#identify image size
sizeMatrix=`/usr/local/sbin/bin/bin/identify -format "%wx%h" ${srcFile}`

echo "index1: $index1"
width=${sizeMatrix%x*}
echo width: ${width}
height=${sizeMatrix#*x}
echo height: ${height}

if [ `echo ${ext}|tr 'a-z' 'A-Z'` = "BMP" ] ; then
  index1=`expr index "${dstFile}" .`
  dstFile=`expr substr "${dstFile}" 1 \`expr ${index1} - 1\``.jpg 
fi

srcRate=$(echo "scale=9; $width*1.0 / $height*1.0" | bc)
destRate=$(echo "scale=9; $destWidth*1.0 / $destHeight*1.0" | bc)

echo "srcRate: ${srcRate}"
echo "destRate: ${destRate}"

resizeCmd="/usr/local/sbin/bin/bin/convert +profile '*' ${srcFile} -resize '${destWidth}x${destHeight}^>' -grAvity Center -crop ${destWidth}x${destHeight}+0+0 +repage ${dstFile}"
newSize="${destWidth}x${destHeight}"
# width < height
if [ ${destWidth} -lt ${destHeight} ] ; then
  newSize="x${destHeight}>"
elif [ `echo "${srcRate} > ${destRate}" | bc` -eq 1 ] ; then
  newSize="x${destHeight}>"
else
  newSize="${destWidth}>"
fi

grAvityWay="Center"
# width < height
if [ ${destWidth} -ge ${destHeight} ] ; then
  grAvityWay="NorthWest"
fi

# resize
resizeCmd="/usr/local/sbin/bin/bin/convert +profile '*' ${srcFile} -resize '${newSize}' -grAvity ${grAvityWay} -crop ${destWidth}x${destHeight}+0+0 +repage ${dstFile}"
echo "resizeCmd: ${resizeCmd}"
eval "${resizeCmd}"

使用范例:/usr/local/sbin/bin/bin/tony-convert-2.sh [src]{file}.{ext} 170x105 [out]{file}_170x105.{ext}

```

五、tony-convert-3.sh:
* 以destWidth表示目的图宽，destHeight表示目的图高，persentSize=destWidth/destHeight表示目的图宽高比;
* 当原图宽<destWidth且高<destHeight时，不截取，直接上传;
* 当原图宽<destWidth,高>destHeight时，以原图宽为宽居中裁剪 竖图;
* 当原图宽>destWidth,高<destHeight时，以原图高为高居中裁剪 横图;
* 当原图宽>destWidth,高>destHeight,
* {

* 当原图宽/高>persentSize时，将原图高压缩为destHeight居中裁剪
* 当原图宽/高<persentSize时，将原图宽压缩为destWidth居中裁剪

* }

```
!/bin/sh

# we have 2 arguments: source file name and dest file name
if [ $# -ne 3 ] ; then 
  echo "Usage: tony-convert-3.sh [source path] [widthxheigth] [dest path]"
  echo "Example: ./tony-convert-3.sh src/1234.jpg 750x375 out/1234_750x375.jpg"
  exit 1 
fi

srcFile="$1"
dstFile="$3"

destSize="$2"
destWidth=${destSize%x*}
destHeight=${destSize#*x}
echo "destSize: ${destWidth}x${destHeight}"

index1=`expr index "${srcFile}" /`
echo "index1: $index1"
index2=`expr index "${srcFile}" .`
echo "index2: $index2"
file=`expr substr "${srcFile}" \`expr ${index1} + 1\` \`expr ${index2} - ${index1} - 1\``
echo "file: $file"
ext=`expr substr "${srcFile}" \`expr ${index2} + 1\` \`expr length "${srcFile}" - ${index2}\``
echo "ext: $ext"

#identify image num
picCount=`/usr/local/sbin/bin/bin/identify ${srcFile}|grep -c "${file}.${ext}"`
echo animate pictures: ${picCount}

if [ ${picCount} -gt 1 ] ; then
  echo "No handle for animate pictures"
  cp ${srcFile} ${dstFile} 
  exit 0 
fi

#identify image size
sizeMatrix=`/usr/local/sbin/bin/bin/identify -format "%wx%h" ${srcFile}`

echo "index1: $index1"
width=${sizeMatrix%x*}
echo width: ${width}
height=${sizeMatrix#*x}
echo height: ${height}

#if [ `echo ${ext}|tr 'a-z' 'A-Z'` = "BMP" ] ; then
  #index1=`expr index "${dstFile}" .`
  #dstFile=`expr substr "${dstFile}" 1 \`expr ${index1} - 1\``.jpg 
#fi


#newSize="${destWidth}x${destHeight}"
heightPadding="0"
widthPadding="0"
persentSize=`echo "scale=4; ${destWidth}/${destHeight}" | bc`
echo "persentSize: ${persentSize}"

# quality
qualityCmd="/usr/local/sbin/bin/bin/convert ${srcFile} -quality 90 ${dstFile}"
echo "qualityCmd: ${qualityCmd}"
eval "${qualityCmd}"


# width > 750 && height <375 hengTu
if [ ${width} -gt ${destWidth} ] && [ ${height} -lt ${destHeight} ] ; then
  heightPadding=$[ ${destHeight}/2-${height}/2]
  echo heightPadding: ${heightPadding}
  flag=0
# width < 750 && height >375 shuTu
elif [ ${width} -lt ${destWidth} ] && [ ${height} -gt ${destHeight} ] ; then
  widthPadding=$[ ${destWidth}/2-${width}/2 ]
  echo widthPadding: ${widthPadding}
  flag=0
# width > 750 && height >375 resize and crop
elif [ ${width} -gt ${destWidth} ] && [ ${height} -gt ${destHeight} ] && [ $ [ ${width}/${height} ]>${persentSize} ] ; then
  echo "resize by src's height then crop"
  newSize="'x${destHeight}>'"
  flag=1
elif [ ${width} -gt ${destWidth} ] && [ ${height} -gt ${destHeight} ] && [ $[ ${width}/${height} ]<${persentSize} ] ; then
  echo "resize by src's width then crop"
  newSize="'${destWidth}>'"
  flag=1
fi

echo "newSize: ${newSize}"

# resize
if [ ${flag} -eq 1 ] ; then
  resizeCmd="/usr/local/sbin/bin/bin/convert ${dstFile} -resize ${newSize} ${dstFile}"
  echo "resizeCmd: ${resizeCmd}"
  eval "${resizeCmd}"

  cropCmd="/usr/local/sbin/bin/bin/convert ${dstFile} -gravity Center -crop ${destSize}+0+0 +repage ${dstFile}"
else
  echo "only crop pic"
  cropCmd="/usr/local/sbin/bin/bin/convert ${dstFile} -gravity Center -crop ${destSize}+${widthPadding}+${heightPadding} +repage ${dstFile}"
fi

# crop
echo "cropCmd: ${cropCmd}"
eval "${cropCmd}"

使用范例：/usr/local/sbin/bin/bin/tony-convert-3.sh [src]{file}.{ext} 750x375 [out]{file}_750x375.{ext}
```

六、tony-convert-3_bg.sh:
* 以destWidth表示目的图宽，destHeight表示目的图高，
* 当原图宽<destWidth且高<destHeight时，不截取，直接上传;
* 当原图宽<destWidth,高>destHeight时，以原图宽为宽居中裁剪 树图;
* 当原图宽>destWidth,高<destHeight时，以原图高为高居中裁剪 横图;
* 当原图宽>destWidth,高>destHeight,
* {
* 当原图宽>高时，将原图高压缩为destHeight居中裁剪
* 当原图宽<高时，将原图宽压缩为destWidth居中裁剪
* }
* 所有情况都要居中补白
```

!/bin/sh

# we have 2 arguments: source file name and dest file name
if [ $# -ne 3 ] ; then 
  echo "Usage: tony-convert-3_bg.sh [source path] [widthxheigth] [dest path]"
  echo "Example: ./tony-convert-3_bg.sh src/1234.jpg 750x375 out/1234_750x375.jpg"
  exit 1 
fi

srcFile="$1"
dstFile="$3"

destSize="$2"
destWidth=${destSize%x*}
destHeight=${destSize#*x}
echo "destSize: ${destWidth}x${destHeight}"

index1=`expr index "${srcFile}" /`
echo "index1: $index1"
index2=`expr index "${srcFile}" .`
echo "index2: $index2"
file=`expr substr "${srcFile}" \`expr ${index1} + 1\` \`expr ${index2} - ${index1} - 1\``
echo "file: $file"
ext=`expr substr "${srcFile}" \`expr ${index2} + 1\` \`expr length "${srcFile}" - ${index2}\``
echo "ext: $ext"

#identify image num
picCount=`/usr/local/sbin/bin/bin/identify ${srcFile}|grep -c "${file}.${ext}"`
echo animate pictures: ${picCount}

if [ ${picCount} -gt 1 ] ; then
  echo "No handle for animate pictures"
  cp ${srcFile} ${dstFile} 
  exit 0 
fi

#identify image size
sizeMatrix=`/usr/local/sbin/bin/bin/identify -format "%wx%h" ${srcFile}`

echo "index1: $index1"
width=${sizeMatrix%x*}
echo width: ${width}
height=${sizeMatrix#*x}
echo height: ${height}

# quality
qualityCmd="/usr/local/sbin/bin/bin/convert ${srcFile} -quality 90 ${dstFile}"
echo "qualityCmd: ${qualityCmd}"
eval "${qualityCmd}"


# width > 750 && height <375 hengTu
flag=0
if [ ${width} -gt ${destWidth} ] && [ ${height} -lt ${destHeight} ] ; then
  heightPadding=$[ ${destHeight}/2-${height}/2]
  echo heightPadding: ${heightPadding}
  flag=0
# width < 750 && height >375 shuTu
elif [ ${width} -lt ${destWidth} ] && [ ${height} -gt ${destHeight} ] ; then
  widthPadding=$[ ${destWidth}/2-${width}/2 ]
  echo widthPadding: ${widthPadding}
  flag=0
# width > 750 && height >375 resize and crop
elif [ ${width} -gt ${destWidth} ] && [ ${height} -gt ${destHeight} ] && [ ${width} -gt ${height} ] ; then
  echo "resize by src's height then crop"
  newSize="'x${destHeight}>'"
  flag=1
elif [ ${width} -gt ${destWidth} ] && [ ${height} -gt ${destHeight} ] && [ ${width} -lt ${height} ] ; then
  echo "resize by src's width then crop"
  newSize="'${destWidth}>'"
  flag=1
fi

echo "newSize: ${newSize}"

# resize
if [ ${flag} -eq 1 ] ; then
  resizeCmd="/usr/local/sbin/bin/bin/convert ${dstFile} -resize ${newSize} ${dstFile}"
  echo "resizeCmd: ${resizeCmd}"
  eval "${resizeCmd}"
  cropCmd="/usr/local/sbin/bin/bin/convert ${dstFile} -gravity Center -crop ${destSize}+0+0 +repage -background white -compose Copy -gravity center -extent ${destSize} ${dstFile}"
else
  echo "only crop pic"
  cropCmd="/usr/local/sbin/bin/bin/convert ${dstFile} -gravity Center -crop ${destSize}+0+0 +repage -background white -compose Copy -gravity center -extent ${destSize} ${dstFile}"
fi

# crop
echo "cropCmd: ${cropCmd}"
eval "${cropCmd}"

使用范例：【240x240大小，背景补白，1PX灰色边框】 /usr/local/sbin/bin/bin/tony-convert-3_bg.sh [src]{file}.{ext} 240x240 [out]{file}_240x240.{ext}
```

七、tony-convert-4.sh:

原图宽大于destWidth:
{
高>destHeight时：定宽等比缩放 以原图高为高居中裁剪；
否则：从左上角裁剪
}
原图宽小于destWidth:
{
高>destHeight时：以原图高为高居中裁剪；
否则：从左上角裁剪
}

```
#!/bin/sh

# we have 2 arguments: source file name and dest file name
if [ $# -ne 3 ] ; then 
  echo "Usage: tony-convert-4.sh [source path] [widthxheigth] [dest path]"
  echo "Example: ./tony-convert-4.sh src/1234.jpg 150x150 out/1234_150x150.jpg"
  exit 1 
fi

srcFile="$1"
dstFile="$3"

destSize="$2"
destWidth=${destSize%x*}
destHeight=${destSize#*x}
echo "destSize: ${destWidth}x${destHeight}"

index1=`expr index "${srcFile}" /`
echo "index1: $index1"
index2=`expr index "${srcFile}" .`
echo "index2: $index2"
file=`expr substr "${srcFile}" \`expr ${index1} + 1\` \`expr ${index2} - ${index1} - 1\``
echo "file: $file"
ext=`expr substr "${srcFile}" \`expr ${index2} + 1\` \`expr length "${srcFile}" - ${index2}\``
echo "ext: $ext"

#identify image num
picCount=`/usr/local/bin/identify ${srcFile}|grep -c "${file}.${ext}"`
echo animate pictures: ${picCount}

if [ ${picCount} -gt 1 ] ; then
  echo "No handle for animate pictures"
  cp ${srcFile} ${dstFile} 
  exit 0 
fi

#identify image size
sizeMatrix=`/usr/local/bin/identify -format "%wx%h" ${srcFile}`

echo "index1: $index1"
width=${sizeMatrix%x*}
echo width: ${width}
height=${sizeMatrix#*x}
echo height: ${height}

#if [ `echo ${ext}|tr 'a-z' 'A-Z'` = "BMP" ] ; then
  #index1=`expr index "${dstFile}" .`
  #dstFile=`expr substr "${dstFile}" 1 \`expr ${index1} - 1\``.jpg 
#fi


newSize="${destWidth}x${destHeight}"
# width < height
if [ ${width} -gt ${destWidth} ] ; then
  newSize="'${destWidth}>'"
  if [ ${height} -gt ${destHeight}] ; then
  heihgtPadding="'${destHeight}/2'"
  else
  heihgtPadding="'0'"
  fi
else
  newSize="'${width}>'"
  if [ ${height} -gt ${destHeight}] ; then
  heihgtPadding="'${destHeight}/2'"
  else
  heihgtPadding="'0'"
  fi
fi

echo "newSize: ${newSize}"

# resize
resizeCmd="/usr/local/sbin/bin/bin/convert ${srcFile} -quality 90 -resize ${newSize} ${dstFile}"
echo "resizeCmd: ${resizeCmd}"
eval "${resizeCmd}"
# add white ground
addWhiteCmd="/usr/local/sbin/bin/bin/convert ${dstFile} -crop ${destSize}+0-${heihgtPadding} +repage ${dstFile}"
echo "addWhiteCmd: ${addWhiteCmd}"
eval "${addWhiteCmd}"

使用范例：【160*120裁剪或居中加水印】/usr/local/sbin/bin/bin/tony-convert-4.sh [src]{file}.{ext} 160x120 [out]{file}_160.{ext}
```

八、tony-convert-5.sh:

宽大于高时：定宽等比缩放 高自适应；
宽小于高时：定高等比缩放,宽自适应;
并居中补白

```
#!/bin/sh

# we have 2 arguments: source file name and dest file name
if [ $# -ne 3 ] ; then 
  echo "Usage: tony-convert-5.sh [source path] [widthxheigth] [dest path]"
  echo "Example: ./tony-convert-5.sh src/1234.jpg 150x150 out/1234_150x2150.jpg"
  exit 1 
fi

srcFile="$1"
dstFile="$3"

destSize="$2"
destWidth=${destSize%x*}
destHeight=${destSize#*x}
echo "destSize: ${destWidth}x${destHeight}"

index1=`expr index "${srcFile}" /`
echo "index1: $index1"
index2=`expr index "${srcFile}" .`
echo "index2: $index2"
file=`expr substr "${srcFile}" \`expr ${index1} + 1\` \`expr ${index2} - ${index1} - 1\``
echo "file: $file"
ext=`expr substr "${srcFile}" \`expr ${index2} + 1\` \`expr length "${srcFile}" - ${index2}\``
echo "ext: $ext"
#identify image num
picCount=`/usr/local/bin/identify ${srcFile}|grep -c "${file}.${ext}"`
echo animate pictures: ${picCount}

if [ ${picCount} -gt 1 ] ; then
  echo "No handle for animate pictures"
  cp ${srcFile} ${dstFile} 
  exit 0 
fi

#identify image size
sizeMatrix=`/usr/local/bin/identify -format "%wx%h" ${srcFile}`

echo "index1: $index1"
width=${sizeMatrix%x*}
echo width: ${width}
height=${sizeMatrix#*x}
echo height: ${height}

#if [ ${width} -lt 300 ] ; then
# if [ ${height} -lt 300 ] ; then
# echo Image size less than 300x300
# cp ${srcFile} ${dstFile}
# exit 0
# fi
#fi

if [ `echo ${ext}|tr 'a-z' 'A-Z'` = "BMP" ] ; then
  index1=`expr index "${dstFile}" .`
  dstFile=`expr substr "${dstFile}" 1 \`expr ${index1} - 1\``.jpg 
fi
newSize="${destWidth}x${destHeight}"
# width < height
if [ ${width} -lt ${height} ] ; then
  newSize="'x${destHeight}>'"
else
  newSize="'${destWidth}>'"
fi
echo "newSize: ${newSize}"

# resize
resizeCmd="/usr/local/sbin/bin/bin/convert ${srcFile} -quality 80 -resize ${newSize} ${dstFile}"
echo "resizeCmd: ${resizeCmd}"
eval "${resizeCmd}"
# add white ground
addWhiteCmd="/usr/local/sbin/bin/bin/convert ${dstFile} -background white -gravity center -extent ${destSize} -gravity Center -crop ${destSize}+0+0 +repage ${dstFile}"
echo "addWhiteCmd: ${addWhiteCmd}"
eval "${addWhiteCmd}"
```