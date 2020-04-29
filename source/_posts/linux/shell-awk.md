---
title: awk相关
categories: linux
tag: hide
date: 2018-12-25 23:50:37
tags:
---
用awk命令计算文件中某一列的总和：

awk 'BEGIN{sum=0}{sum+=$1}END{print sum}' data.txt

比较完整的一个例子：

awk -F ',' 'BEGIN{sum=0 ;count=0}{if ($(NF-11) == 2 && $NF == 0 && $3 == "1.6.1_1_1") {sum +=$5; count++;} } END {print "sum="sum" count="count " avg="sum/count}'

说明：

BEGIN{sum=0 ;count=0} 初始化计数器；

END {print "sum="sum" count="count " avg="sum/count} 打印汇总，计数器 和均值；

if ($(NF-11) == 2 && $NF == 0 && $3 == "1.6.1_1_1") {sum +=$5; count++;} 判断倒数第11个字段，判断倒数第一个字段，判断第三个字段（字符串） 第五个字段汇总累加，计数器累加

很久以前看到gawk手册：从很多计算机术语的使用来看估计出处是台湾。

GAWK手册

第一章 前言

第二章 简介

第三章 读取输入档案

第四章 印出

第五章 Patterns

第六章 算式(Expression)作为Actions的叙述

第七章 Actions里面的控制叙述

第八章 内建函数(Built-in Functions)

第九章 使用者定义的函数

第十章 例子

第十一章 结论

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

＝

第一章 前言

awk 是一个程式语言，对於资料的处理具有很强的功能。对於文字档里的资料做修改、

比对、抽取等的处理，awk 能够以很短的程式轻易地完成。如果使用 C 或 Pascal 等

语言写程式完成上述的动作，会不方便且很花费时间，所写的程式也会很大。

awk 能够依照使用者的定义格式来分解输入资料，也可依照使用者定义的格式来印出资

料。

awk 名称的由来是由它的原始设计者的姓氏之第一个字母而命名：Alfred V. Aho,

Peter J. Weinberger, Brian W. Kernighan。 awk最初在1977年完成。一个新版本的

awk在1985年被发表，它的功能比旧版本增强不少。

gawk 是GNU所做的 awk，gawk 最初在1986年完成，之後不断地被改进、更新。gawk 包

含 awk 的所有功能。

往後的 gawk 将以下面的2个输入档案来做例子说明。

档案'BBS-list'： aardvark 555-5553 1200/300 B alpo-net 555-3412

2400/1200/300 A barfly 555-7685 1200/300 A bites 555-1675 2400/1200/300 A

camelot 555-0542 300 C core 555-2912 1200/300 C fooey 555-1234 2400/1200/300

B foot 555-6699 1200/300 B macfoo 555-6480 1200/300 A sdace 555-3430

2400/1200/300 A sabafoo 555-2127 1200/300 C

档案'shipped'： Jan 13 25 15 115 Feb 15 32 24 226 Mar 15 24 34 228 Apr 31 52

63 420 May 16 34 29 208 Jun 31 42 75 492 Jul 24 34 67 436 Aug 15 34 47 316

Sep 13 55 37 277 Oct 29 54 68 525 Nov 20 87 82 577 Dec 17 35 61 401

Jan 21 36 64 620 Feb 26 58 80 652 Mar 24 75 70 495 Apr 21 70 74 514

第二章 简介

gawk 的主要功能是针对档案的每一行(line)搜寻指定的 patterns 。当一行里有符合

指定的 patterns，gawk 就会在此一行执行被指定的 actions。 gawk 依此方式处理输

入档案的每一行直到输入档案结束。

gawk 程式是由很多的 pattern 与 action 所组成，action 写在大括号 { } 里面，一

个pattern後面就跟著一个action。整个 gawk 程式会像下面的样子：

pattern {action} pattern {action}

在 gawk 程式里面的规则，pattern 或 action 能够被省略，但是两个不能同时被省

略。如果 pattern 被省略，对於输入档里面的每一行，action 都会被执行。如果

action 被省略，内定的 action 则会印出所有符合 pattern 的输入行。

2.1 如何执行gawk程式

基本上，有2个方法可以执行gawk程式。

如果 gawk 程式很短，则 gawk 可以直接写在 command line，如下所示：

gawk 'program' input-file1 input-file2 ...

其中 program 包括一些 pattern 和 action。

如果 gawk 程式较长，较为方便的做法是将 gawk 程式存在一个档案， 即 patterns

与 actions 写在档名为 program-file 的档案里面，执行 gawk 的格式如下所示：

gawk -f program-file input-file1 input-file2 ...

gawk 程式的档案不止一个时，执行gawk 的格式如下所示：

gawk -f program-file1 -f program-file2 ... input-file1 input-file2 ...

2.2 一个简单的例子

现在我们举一个简单的例子，因为 gawk 程式很短，所以将 gawk 程式直接写在

command line。

gawk '/foo/ {print $0}' BBS-list

实际的 gawk 程式为 /foo/ {print $0}。/foo/ 为 pattern，意思为搜寻输入档里的

每一行是否含有子字串 'foo'，如果含有 'foo' 则执行 action。 action 为 print

$0，是将现在这一行的内容印出。BBS-list 是输入的档案。

执行完上述指令後，会印出下面的结果： fooey 555-1234 2400/1200/300 B foot

555-6699 1200/300 B macfoo 555-6480 1200/300 A sabafoo 555-2127 1200/300 C

2.3 一个较复杂的例子

gawk '$1 == "Feb" {sum=$2+$3} END {print sum}' shipped

现在这个例子会将输入档 'shipped' 的第一个栏位与 "Feb" 做比较，如果相等，则其

对应的第2栏位与第3栏位的值会被加到变数 sum。对於输入档的每一行重复上述的动

作，直到输入档的每一行都被处理过为止。最後将 sum 的值印出。END {print sum}

的意思为在所有的输入读完之後，执行一次 print sum 的动作，也就是把 sum 的值印

出。

下面是执行的结果： 84

第三章 读取输入档案

gawk的输入可以从标准输入或指定的档案里读取。输入的读取单位被称为”记录”

(records)，gawk 在做处理时，是一个记录一个记录地处理。每个记录的内定值是一行

(line)，一个记录又被分为多个栏位(fields)。

3.1 如何将输入分解成记录(records)

gawk 语言会把输入分解成记录(record)。记录与记录之间是以 record separator 隔

开，record separator 的内定值是表示新一行的字元(newline character)，因此内定

的 record separator 使得文字的每一行是一个记录。

record separator 随著内建变数 RS 的改变而改变。RS 是一个字串，它的内定值是

"\n"。仅有 RS 的第一个字元是有效的，它被当作 record separator，而 RS 的其它

字元会被忽略。

内建变数 FNR 会储存目前的输入档案已经被读取的记录之个数。内建变数 NR 会储存

目前为止所有的输入档案已经被读取的记录之个数。

3.2 栏位(field)

gawk 会自动将每个记录分解成多个栏位 (field)。类似於字在一行里面，gawk 的内定

动作会认为栏位之间是以 whitespace 分开。在 gawk 里，whitespace 的意思是一个

或多个空白或 tabs。

在 gawk 程式里面，以'$1'表示第一个栏位，'$2'表示第二个栏位，依此类推。举个例

子，假设输入的一行如下所示：

This seems like a pretty nice example.

第一个栏位或 $1 是'This'，第二个栏位或 $2 是 'seems'，依此类推。有个地方值得

特别注意，第七个栏位或 $7 是'example.'而非'example'。

不论有多少栏位，$NF 可用来表示一个记录的最後一个栏位。以上面的例子为例，$NF

与 $7 相同，也就是'example.'。

NF 是一个内建变数，它的值表示目前这个记录之栏位的个数。

$0，看起来好像是第零个栏位，它是一个特例，它表示整个记录。

下面是一个较复杂的例子：

gawk '$1~/foo/ {print $0}' BBS-list

结果如下： fooey 555-1234 2400/1200/300 B foot 555-6699 1200/300 B macfoo

555-6480 1200/300 A sabafoo 555-2127 1200/300 C

这个例子是把输入档'BBS-list'的每个记录的第一个栏位作检查，如果它含有子字串

'foo'，则这一个记录会被印出。

3.3 如何将记录分解成栏位

gawk 根据 field separator 将一个记录分解成栏位。field sepa- rator 以内建变数

FS 表示。

举个例子，假如 field separator 是'oo'，则下面的行：

moo goo gai pan

会被分成三个栏位：'m'、' g'、' gai pan'。

在 gawk 程式里，可以使用'='来改变 FS 的值。例如:

gawk 'BEGIN {FS=","}; {print $2}'

输入行如下：

John Q. Smith, 29 Oak St., Walamazoo, MI 42139

执行gawk的结果将印出字串 ' 29 Oak St.'。BEGIN 後面的 action 会在第一个记录被

读取之前执行一次。

第四章 印出

在gawk程式里，actions 最常做的事就是印出(printing)。简单的印出，使用 printe

叙述。复杂格式的印出，使用 printf 叙述。

4.1 print叙述

print 叙述用在简单、标准的输出格式。叙述的格式如下所示：

print item1, item2, ...

输出时，各个 item 之间会以一个空白分开，最後会换行(newline)。

如果 'print'叙述之後没有跟著任何东西，它与'print $0'的效果一样，它会印出现在

的记录(record)。要印出空白行可使用'print ""'。 印出一段固定的文字，可用双引

号将文字的两边括起来，例如 'print "Hello there"'。

这里是一个例子，它会把每个输入记录的前二个栏位印出：

gawk '{print $1,$2}' shipped

结果如下所示： Jan 13 Feb 15 Mar 15 Apr 31 May 16 Jun 31 Jul 24 Aug 15 Sep

13 Oct 29 Nov 20 Dec 17

Jan 21 Feb 26 Mar 24 Apr 21

4.2 Output Separators

前面我们已提过如果 print 叙述包含有多个 item，item 之间用逗点分开，则印出时

各个item会被一个空白隔开。你能够使用任何的字串作为 output field separator，

可以经由内建变数 OFS 的设定来更改 output field separator。OFS 的初始值为"

"，即一格的空白。

整个 print 叙述的输出被称为 output record。print 叙述输出 output record 之

後，会接著输出一个字串，此字串称为 output record separator。内建变数 ORS 用

来指明此字串。ORS 的初始值为 "\n"，也就是换行。

下面这个例子会印出每个记录的第一个栏位和第二个栏位，此二个栏位之间以分号';'

分开，每行输出之後会加入一个空白行。

gawk 'BEGIN {OFS=";"; ORS="\n\n"} {print $1, $2}' BBS-list

结果如下所示： aardvark;555-5553

alpo-net;555-3412

barfly;555-7685

bites;555-1675

camelot;555-0542

core;555-2912

fooey;555-1234

foot;555-6699

macfoo;555-6480

sdace;555-3430

sabafoo;555-2127

4.3 printf叙述

printf 叙述会使得输出格式较容易精确地控制。printf 叙述可以指定每个 item 印出

的宽度，也可以指定数字的各种型式。

printf 叙述的格式如下：

printf format, item1, item2, ...

print 与 printf 的差别是在於 format, printf 的引数比 print 多了字串 format。

format 的型式与 ANSI C 的 printf 之格式相同。

printf 并不会做自动换行的动作。内建变数 OFS 与 ORS 对 printf 叙述没有任何影

响。

格式的指定以字元'%'开始，後面接著格式控制字母。

格式控制字母如下所示：

'c' 将数字以 ASCII 字元印出。 例如'printf "%C",65'会印出字元'A'。

'd' 印出十进位的整数。

'i' 印出十进位的整数。

'e' 将数字以科学符号的形式印出。 例如

print "$4.3e",1950

结果会印出'1.950e+03'。

'f' 将数字以浮点的形式印出。

'g' 将数字以科学符号的形式或浮点的形式印出。数字的绝对值如果 大於等於0.0001

则以浮点的形式印出，否则以科学符号的形式印 出。

'o' 印出无号的八进位整数。

's' 印出一个字串。

'x' 印出无号的十六进位整数。10至15以'a'至'f'表示。

'X' 印出无号的十六进位整数。10至15以'A'至'F"表示。

'%' 它并不是真正的格式控制字母，'%%"将印出"%'。

在 % 与格式控制字母之间可加入 modifier，modifier 是用来进一步控制输出的格

式。可能的 modifier 如下所示：

'-' 使用在 width 之前，指明是向左靠齐。如果'-'没有出现，则会在 被指定的宽度

向右靠齐。例如：

printf "%-4S", "foo"

会印出'foo '。

'width' 这一个数字指示相对应的栏位印出时的宽度。例如：

printf "%4s","foo"

会印出' foo'。

width 的值是一个最小宽度而非最大宽度。如果一个 item 的 值需要的宽度比 width

大，则不受 width 的影响。例如

printf "%4s","foobar"

将印出'foobar'。

'.prec' 此数字指定印出时的精确度。它指定小数点右边的位数。如 果是要印出一个

字串，它指定此字串最多会被印出多少个字节。

第五章 patterns

在 gawk 程式里面，当 pattern 符合现在的输入记录(record)，其相对应的 action

才会被执行。

5.1 Pattern的种类

这里对 gawk 的各种 pattern 型式作一整理：

/regular expression/ 一个 regular expression 当作一个 pattern。每当输入记录

( record)含有 regular expression 就视为符合。

expression 一个单一的 expression。当一个值不为 0 或一个字串不是空的， 则可视

为符合。

pat1,pat2 一对的 patterns 以逗号分开，指定记录的范围。

BEGIN END 这是特别的 pattern, gawk 在开始执行或要结束时会分别执行相 对应於

BEGIN或END的 action。

null 这是一个空的pattern，对於每个输入记录皆视为符合pattern。

5.2 Regular Expressions当作Patterns

一个 regular expression 可简写为 regexp，是一种描述字串的方法。一个 regular

expression 以斜线('/')包围当作 gawk 的 pattern。

如果输入记录含有 regexp 就视为符合。例如：pattern 为 /foo/，对於任何输入记录

含有'foo'则视为符合。

下面的例子会将含有'foo'的输入记录之第2个栏位印出。

gawk '/foo/ {print $2}' BBS-list

结果如下： 555-1234 555-6699 555-6480 555-2127

regexp 也能使用在比较的算式。

exp ~ /regexp/ 如果 exp 符合 regexp，则结果为真(true)。

exp !~ /regexp/ 如果 exp 不符合 regexp，则结果为真。

5.3 比较的算式当作Patterns

比较的 pattern 用来测试两个数字或字串的关系诸如大於、等於、小於。下面列出一

些比较的pattern：

x<="y" 如果 小於、等於 y，则结果为真。 x>y

如果 x 大於 y，则结果为真。 x>=y 如果 x 大於、等於 y，则结果为真。 x==y 如果

x 等於 y，则结果为真。 x!=y 如果 x 不等於 y，则结果为真。 x~y 如果 x 符合

regular expression y，则结果为真。 x!~y 如果 x 不符合 regular expression y，

则结果为真。

上面所提到的 x 与 y，如果二者皆是数字则视为数字之间的比较，否则它们会被转换

成字串且以字串的形式做比较。两个字串的比较，会先比较第一个字元，然後比较第二

个字元，依此类推，直到有不同的地方出现为止。如果两个字串在较短的一个结束之前

是相等，则视为长的字串比短的字串大。例如 "10" 比 "9" 小，"abc" 比 "abcd"

小。

5.4 使用布林运算的Patterns

一个布林(boolean) pattern 是使用布林运算"或"('||')，"及" ('&&')，"反"('!')来

组合其它的pattern。例如：

gawk '/2400/ && /foo/' BBS-list gawk '/2400/ || /foo/' BBS-list gawk '!

/foo/' BBS-list

第六章 算式(Expression)作为Actions的叙述

算式(Expression) 是gawk程式里面action的基本构成者。

6.1 算术运算

gawk 里的算术运算如下所示：

x+y 加 x-y 减 -x 负 +x 正。实际上没有任何影响。 x*y 乘 x/y 除 x%y 求馀数。例

如 5%3=2。 x^y x**y x 的 y 次方。例如2^3=8。

6.2 比较算式与布林算式

比较算式 (comparison expression) 用来比较字串或数字的关系，运算符号与 C 语言

相同。表列如下：

x<="y" x>y x>=y x==y x!=y x~y x!~y

比较的结果为真(true)则其值是 1。否则其值是 0。

布林算式(boolean expression)有下面三种：

boolean1 && boolean2 boolean1 || boolean2 ! boolean

6.3 条件算式(Conditional Expressions)

一个条件式算式是一种特别的算式，它含有3个运算元。 条件式算式与C语言的相同：

selector ? if-true-exp : if-false-exp

它有3个子算式。第一个子算式selector 首先会被计算。如果是真, 则if-true-exp会

被计算且它的值变成整个算式的值。否则if-false- exp 会被计算且它的值变成整个算

式的值。

例如下面的例子会产生x的绝对值：

x>0 ? x : -x

第七章 Actions里面的控制叙述

在 gawk 程式里面，控制叙述诸如 if、while 等控制程式执行的流程。在 gawk 里的

控制叙述与 C 的类似。

很多的控制叙述会包括其它的叙述，被包括的叙述称为 body。假如 body 里面包括一

个以上的叙述，必须以大括弧 { } 将这些叙述括起来，而各个叙述之间需以换行

(newline)或分号隔开。

7.1 if 叙述

if (condition) then-body [else else-body]

如果 condition 为真(true)，则执行 then-body，否则执行 else-body。

举一个例子如下：

if (x % 2 == 0) print "x is even" else print "x is odd"

7.2 while 叙述

while (condition) body

while 叙述做的第一件事就是测试 condition。假如 condition 为真则执行 body 的

叙述。body 的叙述执行完後，会再测试 condition，假如 condition 为真，则 body

会再度被执行。这个过程会一直被重复直到 condition 不再是真。如果 condition 第

一次测试就是伪(false)，则 body 从没有被执行。

下面的例子会印出每个输入记录(record)的前三个栏位。

gawk '{ i=1 while (i <= 3) { print $i i++ } }'

7.3 do-while 叙述

do body while (condition)

这个 do loop 执行 body 一次，然後只要 condition 是真则会重复执行 body。即使

开始时 condition 是伪，body 也会被执行一次。

下面的例子会印出每个输入记录十次。

gawk '{ i= 1 do { print $0 i++ } while (i <= 10) }'

7.4 for 叙述

for (initialization; condition; increment) body

此叙述开始时会执行initialization，然後只要 condition是真，它会重复执行body与

做increment 。

下面的例子会印出每个输入记录的前三个栏位。

gawk '{ for (i=1; i<=3; i++) print $i }'

7.5 break 叙述

break 叙述会跳出包含它的 for、while、do-while 回圈的最内层。

下面的例子会找出任何整数的最小除数，它也会判断是否为质数。

gawk '# find smallest divisor of num { num=$1 for (div=2; div*div <=num;

div++) if (num % div == 0) break if (num % div == 0) printf "Smallest

divisor of %d is %d\n", num, div else printf "%d is prime\n", num }'

7.6 continue 叙述

continue 叙述使用於 for、while、do-while 回圈内部，它会跳过回圈 body 的剩馀

部分，使得它立刻进行下一次回圈的执行。

下面的例子会印出 0 至 20 的全部数字，但是 5 并不会被印出。

gawk 'BEGIN { for (x=0; x<=20; x++) { if (x==5) continue printf ("%d",x) }

print "" }'

7.7 next 叙述、next file 叙述、exit 叙述

next 叙述强迫 gawk 立刻停止处理目前的记录(record)而继续下一个记录。

next file 叙述类似 next。然而，它强迫 gawk 立刻停止处理目前的资料档。

exit 叙述会使得 gawk 程式停止执行而跳出。然而，如果 END 出现，它会去执行 END

的 actions。

第八章 内建函数(Built-in Functions)

内建函数是 gawk 内建的函数，可在 gawk 程式的任何地方呼叫内建函数。

8.1 数值方面的内建函数

int(x) 求出 x 的整数部份，朝向 0 的方向做舍去。例如：int(3.9) 是 3，

int(-3.9) 是 -3。 sqrt(x) 求出 x 正的平方根值。例 sqrt(4)=2 exp(x) 求出 x 的

次方。例 exp(2) 即是求 e*e 。 log(x) 求出 x 的自然对数。 sin(x) 求出 x 的

sine 值，x 是弪度量。 cos(x) 求出 x 的 cosine 值，x 是弪度量。 atan2(y,x) 求

y/x 的 arctangent 值，所求出的值其单位是弪度量。 rand() 得出一个乱数值。此乱

数值平均分布在 0 和 1 之间。这个 值不会是 0，也不会是 1。 每次执行 gawk，

rand 开始产生数字从相同点或 seed。 srand(x) 设定产生乱数的开始点或 seed 为

x。如果在第二次你设 定相同的 seed 值，你将再度得到相同序列的乱数值。 如果省

略引数 x，例如 srand()，则现在的日期、时间会 被当成 seed。这个方法可使得乱数

值是真正不可预测的。 srand 的传回值(return value)是前次所设定的 seed 值。

8.2 字串方面的内建函数

index(in, find) 它会在字串 in 里面，寻找字串 find 第一次出现的地方，传回值是

字串 find 出现在字串 in 里面的位置。如果在字串 in 里面找不到字 串 find，则传

回值为 0。 例如： print index("peanut","an") 会印出 3。

length(string) 求出 string 有几个字元。 例如： length("abcde") 是 5。

match(string,regexp) match 函数会在字串 string 里面，寻找符合 regexp 的最

长、最靠 左边的子字串。传回值是 regexp 在 string 的开始位置，即 index 值。

match 函数会设定内在变数 RSTART 等於 index，它也会设定内在变 数 RLENGTH 等於

符合的字元个数。如果不符合，则会设定 RSTART 为 0、RLENGTH 为 -1。

sprintf(format,expression1,...) 举 printf 类似，但是 sprintf 并不印出，而是

传回字串。 例如： sprintf("pi = %.2f (approx.)',22/7) 传回的字串为"pi = 3.14

(approx.)"

sub(regexp, replacement,target) 在字串 target 里面，寻找符合 regexp 的最长、

最靠左边的地方， 以字串 replacement 代替最左边的 regexp。 例如： str =

"water, water, everywhere" sub(/at/, "ith",str) 结果字串str会变成 "wither,

water, everywhere"

gsub(regexp, replacement, target) gsub 与前面的 sub 类似。在字串 target 里

面，寻找符合 regexp 的 所有地方，以字串 replacement 代替所有的 regexp。 例如

： str="water, water, everywhere" gsub(/at/, "ith",str) 结果字串str会变成

'wither, wither, everywhere"

substr(string, start, length) 传回字串 string 的子字串，这个子字串的长度为

length 个字元， 从第 start 个位置开始。 例如： substr("washington",5,3) 传回

值为"ing" 如果 length 没有出现，则传回的子字串是从第 start 个位置开始 至结

束。 例如： substr("washington",5) 传回值为"ington"

tolower(string) 将字串string的大写字母改为小写字母。 例如： tolower("MiXeD

cAsE 123") 传回值为"mixed case 123"

toupper(string) 将字串string的小写字母改为大写字母。 例如： toupper("MiXeD

cAsE 123") 传回值为"MIXED CASE 123"

8.3 输入输出的内建函数

close(filename) 将输入或输出的档案 filename 关闭。

system(command) 此函数允许使用者执行作业系统的指令，执行完毕後将回到 gawk 程

式。 例如： BEGIN {system("ls")}

第九章 使用者定义的函数(User-defined Functions)

复杂的 gawk 程式常常可以使用自己定义的函数来简化。呼叫使用者定义的函数与呼叫

内建函数的方法一样。

9.1 函数定义的格式

函数的定义可以放在 gawk 程式的任何地方。

一个使用者定义的函数其格式如下：

function name (parameter-list) { body-of-function }

name 是所定义的函数之名称。一个正确的函数名称可包括一序列的字母、数字、下标

线 (underscores)，但是不可用数字做开头。

parameter-list 是列出函数的全部引数(argument)，各个引数之间以逗点隔开。

body-of-function 包含 gawk 的叙述 (statement)。它是函数定义里最重要的部份，

它决定函数实际要做何种事。

9.2 函数定义的例子

下面这个例子，会将每个记录的第一个栏位之值的平方与第二个栏位之值的平方加起

来。

{print "sum =",SquareSum($1,$2)} function SquareSum(x,y) { sum=x*x+y*y

return sum }

第十章 例子

这里将列出 gawk 程式的一些例子。

gawk '{if (NF > max) max = NF} END {print max}' 此程式会印出所有输入行之中，

栏位的最大个数。

gawk 'length($0) > 80' 此程式会印出一行超过 80 个字元的每一行。此处只有

pattern 被 列出，action 是采用内定的 print。

gawk 'NF > 0' 对於拥有至少一个栏位的所有行，此程式皆会印出。这是一个简 单的

方法，将一个档案里的所有空白行删除。

gawk '{if (NF > 0) print}' 对於拥有至少一个栏位的所有行，此程式皆会印出。这

是一个简 单的方法，将一个档案里的所有空白行删除。

gawk 'BEGIN {for (i = 1; i <= 7; i++) print int(101 * rand())}' 此程式会印出

范围是 0 到 100 之间的 7 个乱数值。

ls -l files | gawk '{x += $4}; END {print "total bytes: " x}' 此程式会印出所

有指定的档案之bytes数目的总和。

expand file | gawk '{if (x < length()) x = length()} END {print "maximum

line length is " x}' 此程式会将指定档案里最长一行的长度印出。expand 会将 tab

改 成 space，所以是用实际的右边界来做长度的比较。

gawk 'BEGIN {FS = ":"} {print $1 | "sort"}' /etc/passwd 此程式会将所有使用者

的login名称，依照字母的顺序印出。

gawk '{nlines++} END {print nlines}' 此程式会将一个档案的总行数印出。

gawk 'END {print NR}' 此程式也会将一个档案的总行数印出，但是计算行数的工作由

gawk 来做。

gawk '{print NR,$0}' 此程式印出档案的内容时，会在每行的最前面印出行号，它的

功 能与 'cat -n' 类似。

第十一章 结论

gawk 对於资料的处理具有很强的功能。它能够以很短的程式完成想要做的事，甚至一

或二行的程式就能完成指定的工作。同样的一件工作，以 gawk 程式来写会比用其它程

式语言来写短很多。

gawk 是 GNU 所做的 awk，它是公众软体(Public Domain) 可免费使用。




# awk详解

linux shell awk 流程控制语句（if,for,while,do)详细介绍
在linux awk的 while、do-while和for语句中允许使用break,continue语句来控制流程走向，也允许使用exit这样的语句来退出。break中断当前正在执行的循环并跳到循环外执行下一条语句。if 是流程选择用法。 awk中，流程控制语句，语法结构，与c语言类型。下面是各个语句用法。

 

一.条件判断语句(if)

if(表达式) #if ( Variable in Array )
语句1
else
语句2

格式中"语句1"可以是多个语句，如果你为了方便Unix awk判断也方便你自已阅读，你最好将多个语句用{}括起来。Unix awk分枝结构允许嵌套，其格式为：

if(表达式)

{语句1}

else if(表达式)
{语句2}
else
{语句3}

[chengmo@localhost nginx]# awk 'BEGIN{ 
test=100;
if(test>90)
{
    print "very good";
}
else if(test>60)
{
    print "good";
}
else
{
    print "no pass";
}
}'

very good

 

每条命令语句后面可以用“；”号结尾。

 

二.循环语句(while,for,do)

1.while语句

格式：

while(表达式)

{语句}

例子：

[chengmo@localhost nginx]# awk 'BEGIN{ 
test=100;
total=0;
while(i<=test)
{
    total+=i;
    i++;
}
print total;
}'
5050

2.for 循环

for循环有两种格式：

格式1：

for(变量 in 数组)

{语句}

例子：

[chengmo@localhost nginx]# awk 'BEGIN{ 
for(k in ENVIRON)
{
    print k"="ENVIRON[k];
}
}'

AWKPATH=.:/usr/share/awk
OLDPWD=/home/web97
SSH_ASKPASS=/usr/libexec/openssh/gnome-ssh-askpass
SELINUX_LEVEL_REQUESTED=
SELINUX_ROLE_REQUESTED=
LANG=zh_CN.GB2312

。。。。。。

说明：ENVIRON 是awk常量，是子典型数组。

格式2：

for(变量;条件;表达式)

{语句}

例子：

[chengmo@localhost nginx]# awk 'BEGIN{ 
total=0;
for(i=0;i<=100;i++)
{
    total+=i;
}
print total;
}'

5050

3.do循环

格式：

do

{语句}while(条件)

例子：

[chengmo@localhost nginx]# awk 'BEGIN{ 
total=0;
i=0;
do
{
    total+=i;
    i++;
}while(i<=100)
print total;
}'
5050

 

以上为awk流程控制语句，从语法上面大家可以看到，与c语言是一样的。有了这些语句，其实很多shell程序都可以交给awk，而且性能是非常快的。

 

break 当 break 语句用于 while 或 for 语句时，导致退出程序循环。
continue 当 continue 语句用于 while 或 for 语句时，使程序循环移动到下一个迭代。
next 能能够导致读入下一个输入行，并返回到脚本的顶部。这可以避免对当前输入行执行其他的操作过程。
exit 语句使主输入循环退出并将控制转移到END,如果END存在的话。如果没有定义END规则，或在END中应用exit语句，则终止脚本的执行。
   
三、性能比较

[chengmo@localhost nginx]# time (awk 'BEGIN{ total=0;for(i=0;i<=10000;i++){total+=i;}print total;}')
50005000

real    0m0.003s
user    0m0.003s
sys     0m0.000s
[chengmo@localhost nginx]# time(total=0;for i in $(seq 10000);do total=$(($total+i));done;echo $total;)
50005000

real    0m0.141s
user    0m0.125s
sys     0m0.008s



awk
awk '{if($9 ~ /502/){print $num}}' /data/nginxlogs/bbs.goapk.com.access.log
error=$(ssh $i "awk -v hour=`date -d "1 hours ago" +%Y:%H` 'BEGIN{\$sum=0}{if(\$4 ~ hour && \$9 ~ /502/)sum++}END{print sum}' /data/nginxlogs/bbs.goapk.com.access.log")
awk -v hour=`date -d "1 hours ago" +%Y:%H` 'BEGIN{$sum=0}{if($4 ~ hour && $9 ~ /502/)sum++}END{print sum}' /data/nginxlogs/bbs.goapk.com.access.log