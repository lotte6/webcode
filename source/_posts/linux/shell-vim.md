---
title: shell_vim
categories: linux
tag: hide
date: 2018-12-26 00:01:08
tags:
---
```
g/^$/d
%s/^#.*//g
g/^\s*$/d
#删除所有的空白行和空行
:g/^[  ][  ]*$/d
#在每行的开始插入两个空白f
:%s/^/>  /
—-
#在接下来的6行末尾加入.
:.,5/$/./
—-
#颠倒文件的行序
:g/.*/m0O  <=> :g/^/m0O
—-
#寻找不是数字的开始行,并将其移到文件尾部
:g!/^[0-9]/m$ <=> g/^[^0-9]/m$
—-
#将文件的第12到17行内容复制10词放到当前文件的尾部
:1,10g/^/12,17t$
~~~~重复次数的作用
—-
#将chapter开始行下面的第二行的内容写道begin文件中
:g/^chapter/.+2w>>begin
—-
:/^part2/,/^part3/g/^chapter/.+2w>>begin
—-
:/^part2/,/^part3/g/^chapter/.+2w>>begin|+t$

#在每行的开始插入两个空白
:%s/^/>  /
—-
#在接下来的6行末尾加入.
:.,5/$/./
—-
#颠倒文件的行序
:g/.*/m0O  <=> :g/^/m0O
—-
#寻找不是数字的开始行,并将其移到文件尾部
:g!/^[0-9]/m$ <=> g/^[^0-9]/m$
—-
#将文件的第12到17行内容复制10词放到当前文件的尾部
:1,10g/^/12,17t$
~~~~重复次数的作用
—-
#将chapter开始行下面的第二行的内容写道begin文件中
:g/^chapter/.+2w>>begin
—-
:/^part2/,/^part3/g/^chapter/.+2w>>begin
—-
:/^part2/,/^part3/g/^chapter/.+2w>>begin|+t$


 ctrl+v可进入列模式，选择多列就可进行编辑。按v选择多行，按 < >就可以自动缩进。在某行上按两次等号（==）变可以自动缩进。 n==从该行开始下面的n行一起做自动缩进。gg=G对真个文件进行自动缩进和排版，结合set shiftwidth=4,非常有用的自动缩进。比visual studio的format还灵活。

Vim 对特定行处理常用方法（五·完）：跨行操作及长文本分割基础

zecy · 1 年前

目录

- 奇偶行删除
- 合并奇偶行
- 奇偶行分离（及寄存器入门）
- 删除、压缩重复行
- 跨行操作及长文本分割基础（本文）

5. 跨行操作及长文本分割基础

在前面的篇章中，我们已经掌握了删除、合并奇偶数行、重复行、空白行的常用方法。 在此基础上，只要配合适当的 {pattern} 就能很方便地处理特定的行，但是具体的使用只能靠各位在实际情况下自行摸索，在这有限的文章里不可能一一提及、讲解。

但是，对于初学者，有一个门槛是不容易迈过去的，那就是跨行操作。在这一篇中，我们将会来认识一点多行操作的基本技巧，这样当各位在未来遇到更复杂的情况时，也能有一个思考解决方法的方向。

而在本文的最后，我们还会来接触一下比较高级的技巧：文本分割，在那里，我们会接触到通过 execute 拼接命令。

** 本篇命令 **

5.1 跨行删除

:%s/^\d\+楼\_.\{-}\n^\d\+\n^$\n//g
  使用 \_. 进行跨行匹配，然后删除。

:g/^\d\+楼/,/^\d\+\n^$\n/d
  和上一个命令效果一致，利用 [range]
  跨行删除。

5.2 跨行保留

:g!/{pattern}/d global 命令的逆操作，用于保留单独
:v/{pattern}/d 的特定行。

:%s/^\d\+楼.*\n\(.*\n\)\{5}\(.*\)\_.\{-}\n^$\n/\2\r
  通过群组来保留特定内容。

:%s/\v^\d+楼.*\n(.*\n){5}(.*)\_.{-}\n^$\n/\2\r
  和上条命令效果一致，使用了 \v 魔术
  开关，增强命令可读性。

5.3 跨行分离

:let @a="" 对第 3 章命令的一个小改造，让其可以
:g/^\d\+楼/,/^\d\+\n^$\n/d A 用于跨行文本。原理上和第 3 章的命令
:g/^\d\+楼/,/^\d\+\n^$\n/m$ 是一致的。

5.4 长文本分割

:g/^第.回/exe ",/下文分解/w! ~/Desktop/" . substitute(getline('.'),' ','\\ ','g') . "\.txt"
  使用了 execute 命令对 Ex 命令进行
  拼接，以达到自动为文件命名的效果。同
  时通过 substitute 命令对文件名进行
  修改，使命令可以正确运行。

** 本篇帮助 **

:h /\_
:h /\_.
:h :v:h :execute
:h :w:h substitute():h getline()

5.1 跨行删除

请看一个例子：

1楼2014-08-04 20:24

举报 |
个人企业举报
垃圾信息举报

×_××
xxx
3

挺不错的，支持下


2楼2014-08-05 02:33

举报 |
个人企业举报
垃圾信息举报

××××
xxxxxx
吧主
13

嗯不错，支持原创


3楼2014-08-05 12:18

举报 |
个人企业举报
垃圾信息举报

xxxxxxxx
l
1

支持原创,继续努力.


4楼2014-08-06 09:48

举报 |
个人企业举报
垃圾信息举报


xxxx
l
1

支持楼主！


这是在百度贴吧复制下来的内容，为了方便演示我作了修改。假设这是一篇比较长的帖子，我希望只把内容保留下来，其他都删掉，即达到这个效果：

挺不错的，支持下

嗯不错，支持原创

支持原创,继续努力.

支持楼主！


多余出来的是贴吧的用户信息，而其结构是固定的：

1楼2014-08-04 20:24 " 楼层及发帖时间，格式固定
  " 空行，内容固定
举报 | " 举报信息，格式固定
个人企业举报
垃圾信息举报
  " 空行，格式固定
×_×× " 用户名，不固定
xxx " 贴吧头衔，不固定3 " 贴吧头衔等级，格式固定
  " 空行，内容固定
xxxxxxxxxx " 正文内容

我们可以看到，作为起始部分的楼层信息格式是固定的，而作为结束部分的头衔等级、空行也是固定的。故此，我们我们可以用 {pattern} 轻易匹配到起始和结束的部分，而绕开难以处理的用户名和贴吧头衔。

:%s/^\d\+楼\_.\{-}\n^\d\+\n^$\n//g

:h /\_
:h /\_.


\_. 是这一个 {pattern} 的核心所在。我们前面已经知道，.* 这个元字符组合可以匹配到行内所有内容，唯独不包括行尾自字符。也就是说，.* 默认情况下是不匹配换行符的。

如果我们需要匹配起始行到结束行中间的任意多行，用 .* 就无法实现。但是，如果加上 \_. ，就可以匹配到行尾字符。这里使用的 \_.\{-} 意思就是匹配起始行和结束行中间的全部内容。因为 * 是贪婪匹配的，用 * 就会匹配到最远的结束行，因此这里用了 \{-} ，作最少的匹配，保证匹配到的是最近的结束行。

我们也能用 global 命令达到同样的效果：

:g/^\d\+楼/,/^\d\+\n^$\n/d

这个命令形式在第一篇的时候出现过，大家应该还有印象，利用了 ex 命令的 [range] 。「,/^\d\+\n^$\n/ 」实际上等同于「 .,/^\d\+\n^$\n/ 」，相当于是「 +1d 」中的「 +1 」。意思是「从当前行到×××」。

这是该 global 命令的结构：

  |pattern| [range] | ex 命令

:g
  / /
  ^\d\+楼 
  ,/^\d\+\n^$\n/
  d

5.2 跨行保留

好了，我们现在大概知道了怎样通过特定行的 {pattern} 再配合 \_. 进行跨行的匹配，并且通过这样的匹配来进行删除操作。

那如果我们跟进一步， 希望保留用户名呢？

在删除特定行的时候，我们可以简单地使用 global 命令：

:g/{pattern}/d

如果要保留特定行，使用 global 的逆操作就可以：

:g!/{pattern}/d:v/{pattern}/d

:h :v

但是，在这个例子中我们并不能这样做，因为用户名的部分并没有格式上的特征。

所以，我们就需要借助其他具有特征的行来定位到用户名所在的行，然而利用群组来保留我们需要的内容：

:%s/^\d\+楼.*\n\(.*\n\)\{5}\(.*\)\_.\{-}\n^$\n/\2\r

这个正则表达式很不友好，特别是用于转义的反斜杠太多了，我们先用 magic 开关 \v 改写一下，起码读起来稍微舒服点：

:%s/\v^\d+楼.*\n(.*\n){5}(.*)\_.{-}\n^$\n/\2\r

现在我们来看看其结构：

:%s/ / => 全局替换
  \v => 魔术开关
  ^\d+楼.*\n => 起始行，楼层信息
  (.*\n){5} => 起始行后5行
  (.*) => 第7行，用户名
  \_.{-}\n => 用户名后任意多行
  ^$\n => 结束行，空行
  \2\r => 换为用户名和换行


这里我们用了两个群组，分别是第 5 层和第 6 层。

第 5 层的群组 1 仅仅起到了分组的作用，便于我们用倍数项 {5} 设置匹配次数，第 6 层的群组 2（匹配内容的第 7 行）则是捕获到了用户名，可以看到群组 2 的引用 \2 在后面的替换中就派上了用场。

如果更严谨一点，群组 1 应该使用 \%(\) 来代替 \(\) ， \%(\) 仅起到分组的作用，无法比引用，所以性能上会好于 \(\) 。

5.3 跨行分离

在第 3 章中，我们利用了寄存器和 copy-move 命令来进行内容的分离，当时的命令稍作修改就能进行跨行的分离，原理上并没有什么不同，列在下面作为参考：

:let @a="":g/^\d\+楼/,/^\d\+\n^$\n/d A

:g/^\d\+楼/,/^\d\+\n^$\n/m$


5.4 文本分割

好了，经过前面的学习，我们来到了一个重头戏——怎么用 vim 来做文本分割。

在处理小说文本的时候，我们最容易接触到的一个需求就是文本分割。网上下载了一个小说 txt，我们通过大量的工作，把里面的广告信息、杂乱空行、重复段落统统删除掉以后，把文本放到手机里面阅读的最后一步工作就是文本分割了。

现在手机的性能强大了所以大家可能没有感觉，在诺基亚功能机时代，用手机来看 txt 并不是一件很简单的事情，你不能一次过把一个太大的 txt 文件放到手机里，这样的话打开一个文件的时间和很长，而且操作也会很混慢，有的时候还会识别不了那么大的 txt 文件。

因此，把一个小说 txt 文件按章节来划分就是一个很常见的需求。在以前，也有一些专门的文本分割工具。

但是，既然我们都已经用 vim 来处理完整个文件了，为什么不用 vim 来把文件分割也做好呢？

在开始之前，我们先来看一案例，这是笔者多年前入门 vim 的时候看到的：用Vim快速分割文本

这位朋友也希望用 vim 来进行文本分割，到最后想到了一个并不高明的方法，算是勉强搞定，不过在自动化方面还是没有解决。

笔者当时也是研究了很久，可惜当时对 [range] 完全没有理解，最终还是用脚本获取行数来解决：

function Booksplite()
  let starts=[]
  let starts_num = 0
  let starts_cou = -1
  let c = 1
  g/<title>.*<\/title>/let i=line(".")|call add(starts,i)|let starts_cou+=1
  while starts_num < starts_cou
  let ends_num = starts_num +1
  let ends_line = starts[ends_num] - 1
  execute "silent ".starts[starts_num].",".ends_line."w c".c.".xhtml"
  let starts_num += 1
  let c += 1
  endwhile
  execute "silent ".starts[starts_num].",$"."w c".c.".xhtml"endfunction

这个脚本思路很简单：通过 global 命令找到标题所在的行，然后把行号存进一个 list 变量里面，扫描完整个文档之后，就可以得到一个有标题行号的 list 。

这样标题行号，还有下一个标题行号 -1 的这两个行号之间的内容，就是一个章节的内容。以这两个行号为 [range] 进行保存就可以了。

但是，实际上只要对 [range] 有更进一步的理解，就知道完全不需要那么麻烦。

对于上面提到的《三国演义》案例，这样一个单行命令就足够了：

:g/^第.回/exe ",/下文分解/w! ~/Desktop/" . substitute(getline('.'),' ','\\ ','g') . "\.txt"

这条命令使用了两个技巧：通过 execute 进行命令拼接、通过 [range] 进行文本分割。

:h :execute


这个命令后面跟一个表达式，这个表达式可以是任何以文本形式编写的 Ex 命令。比如下面的两组命令是同样的：

:exe "d":d

:exe "let a = 1":let a = 1

我们现在来看看上面用到的 Ex 命令：

exe ",/下文分解/w! ~/Desktop/" . substitute(getline('.'),' ','\\ ','g') . "\.txt"

这个命令有 3 段，通过「 . 」链接，表达式用过「 . 」链接在之前的篇章已经多次看到：

01 => ,/下文分解/w! ~/Desktop/

02 => substitute(getline('.'),' ','\\ ','g')

03 => \.txt


这 3 段实际上就 w 命令的一个拼接：

:[range]w[rite] path/filename.suffix

:h :w

,/下文分解/ => [range]，意义5.1已有说明
  w! => w 命令，加上 ! 为强制写入
  ~/Desktop/ => 文件所在路径
  02 段 => 这部分实际上是对文件名的处理
  /\.txt => 文件后缀，\. 是转义


这样一看应该就很清晰了，最后一个 \. 的转义是因为上述内容是作为文本向 exe 命令输入的，所以「 . 」、「 」（空格）、「 \ 」、「 " 」等符号都要加「 \ 」转义，否则会出错。

现在我们来重点看看 02 段的命令：

substitute(getline('.'),' ','\\ ','g')

:h substitute():h getline()

这是 vim 的两个内建函数，getline() 大家在上一篇已经看过了，用途是获取特定行的内容，getline('.') 就是获取当前行内容。在本命令中，这个当前行实际上就是标题所在的行。

本案例中，标题中是有空格分隔的，属于这样的形式：

第二回 张翼德怒鞭督邮 何国舅谋诛宦竖


这也是案例中的作者使用 <cWORD> 获取标题后会出错的原因，w 后有空格会视为多个文件名，会报错。

这里有个粗暴的解决方案：

:exe "w ~Desktop/'".getline('.')."'\.txt"
  ^ ^


注意命令中的两个单引号，这两个引号包围了整个标题，会让 exe 忽视标题中的空格。不过缺点就是，保存成功的文件名称也会有单引号。

这不完美，所以我用了另外一个解决方案，用 substitute 命令把所有空格都加上反斜杠转义：

substitute(getline('.'),' ','\\ ','g')

替换前 => exe "w 第二回 张翼德怒鞭督邮 何国舅谋诛宦竖.txt" 

相当于 => w 第二回 张翼德怒鞭督邮 何国舅谋诛宦竖.txt

替换后 => exe "w 第二回\\ 张翼德怒鞭督邮\\ \\ 何国舅谋诛宦竖.txt" 

相当于 => w 第二回\ 张翼德怒鞭督邮\ \ 何国舅谋诛宦竖.txt


这里有两重转义，首先 w 命令中的空格需要用 \ 转义，变成「 \ 」，然后如果要让 w 命令中出现「 \ 」，就需要为 exe 命令中的 \ 转义，于是变成 \\ 。

如果我们假设 exe 中的转义符号是「 + 」，把么命令就要写成：

exe "w 第二回+\ 张翼德怒鞭督邮+\ +\ 何国舅谋诛宦竖.txt"



这样看起来会更清晰一点。但要注意，这只是让各位可以看清楚这里的双重转义，实际操作中还是要老老实实写反斜杠。

结语

一连五篇的 vim 特定行操作常用方法介绍文档就到此结束了，大致上介绍了一写使用 vim 处理文本行时会常见的需求和解决方案。因为是面向入门的文章，所以并没有涉及到太多深入的内容，而且原理性的东西会讲得更多一点，让刚刚接触的同好们对于所用的命令有更清晰的认识和理解，这对于日后自己搭配使用这种命令是十分重要的。

对于 vim 的三大高级功能，比如 Ex 命令、寄存器、Vim Language，基本上都是集中在 Ex 命令的部分，确实不太足够，在往后的其他文章中，我会把重心倾斜到寄存器和 Vim Language 这方面上来。

本篇命令及帮助回顾

** 本篇命令 **

5.1 跨行删除

:%s/^\d\+楼\_.\{-}\n^\d\+\n^$\n//g
  使用 \_. 进行跨行匹配，然后删除。

:g/^\d\+楼/,/^\d\+\n^$\n/d
  和上一个命令效果一致，利用 [range]
  跨行删除。

5.2 跨行保留

:g!/{pattern}/d global 命令的逆操作，用于保留单独
:v/{pattern}/d 的特定行。

:%s/^\d\+楼.*\n\(.*\n\)\{5}\(.*\)\_.\{-}\n^$\n/\2\r
  通过群组来保留特定内容。

:%s/\v^\d+楼.*\n(.*\n){5}(.*)\_.{-}\n^$\n/\2\r
  和上条命令效果一致，使用了 \v 魔术
  开关，增强命令可读性。

5.3 跨行分离

:let @a="" 对第 3 章命令的一个小改造，让其可以
:g/^\d\+楼/,/^\d\+\n^$\n/d A 用于跨行文本。原理上和第 3 章的命令
:g/^\d\+楼/,/^\d\+\n^$\n/m$ 是一致的。

5.4 长文本分割

:g/^第.回/exe ",/下文分解/w! ~/Desktop/" . substitute(getline('.'),' ','\\ ','g') . "\.txt"
  使用了 execute 命令对 Ex 命令进行
  拼接，以达到自动为文件命名的效果。同
  时通过 substitute 命令对文件名进行
  修改，使命令可以正确运行。

** 本篇帮助 **

:h /\_
:h /\_.
:h :v:h :execute
:h :w:h substitute():h getline()
```


# vimrc 习惯配置

```
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

" 显示相关  

""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"set shortmess=atI   " 启动的时候不显示那个援助乌干达儿童的提示  

"winpos 5 5          " 设定窗口位置  

"set lines=40 columns=155    " 设定窗口大小  

"set nu              " 显示行号  

set go=             " 不要图形按钮  

"color asmanian2     " 设置背景主题  

set guifont=Courier_New:h10:cANSI   " 设置字体  

"syntax on           " 语法高亮  

autocmd InsertLeave * se nocul  " 用浅色高亮当前行  

autocmd InsertEnter * se cul    " 用浅色高亮当前行  

"set ruler           " 显示标尺  

set showcmd         " 输入的命令显示出来，看的清楚些  

"set cmdheight=1     " 命令行（在状态行下）的高度，设置为1  

"set whichwrap+=<,>,h,l   " 允许backspace和光标键跨越行边界(不建议)  

"set scrolloff=3     " 光标移动到buffer的顶部和底部时保持3行距离  

set novisualbell    " 不要闪烁(不明白)  

set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%l,%v][%p%%]\ %{strftime(\"%d/%m/%y\ -\ %H:%M\")}   "状态行显示的内容  

set laststatus=1    " 启动显示状态行(1),总是显示状态行(2)  

set foldenable      " 允许折叠  

set foldmethod=manual   " 手动折叠  

"set background=dark "背景使用黑色 

set nocompatible  "去掉讨厌的有关vi一致性模式，避免以前版本的一些bug和局限  

" 显示中文帮助

if version >= 603

    set helplang=cn

    set encoding=utf-8

endif

" 设置配色方案

"colorscheme murphy

"字体 

"if (has("gui_running")) 

"   set guifont=Bitstream\ Vera\ Sans\ Mono\ 10 

"endif 


 
set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936

set termencoding=utf-8

set encoding=utf-8

set fileencodings=ucs-bom,utf-8,cp936

set fileencoding=utf-8



"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"""""新文件标题""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"新建.c,.h,.sh,.java文件，自动插入文件头 

autocmd BufNewFile *.cpp,*.[ch],*.sh,*.java exec ":call SetTitle()" 

""定义函数SetTitle，自动插入文件头 

func SetTitle() 

    "如果文件类型为.sh文件 

    if &filetype == 'sh' 

        call setline(1,"\#########################################################################") 

        call append(line("."), "\# File Name: ".expand("%")) 

        call append(line(".")+1, "\# Author: ma6174") 

        call append(line(".")+2, "\# mail: ma6174@163.com") 

        call append(line(".")+3, "\# Created Time: ".strftime("%c")) 

        call append(line(".")+4, "\#########################################################################") 

        call append(line(".")+5, "\#!/bin/bash") 

        call append(line(".")+6, "") 

    else 

        call setline(1, "/*************************************************************************") 

        call append(line("."), "    > File Name: ".expand("%")) 

        call append(line(".")+1, "    > Author: ma6174") 

        call append(line(".")+2, "    > Mail: ma6174@163.com ") 

        call append(line(".")+3, "    > Created Time: ".strftime("%c")) 

        call append(line(".")+4, " ************************************************************************/") 

        call append(line(".")+5, "")

    endif

    if &filetype == 'cpp'

        call append(line(".")+6, "#include<iostream>")

        call append(line(".")+7, "using namespace std;")

        call append(line(".")+8, "")

    endif

    if &filetype == 'c'

        call append(line(".")+6, "#include<stdio.h>")

        call append(line(".")+7, "")

    endif

    "新建文件后，自动定位到文件末尾

    autocmd BufNewFile * normal G

endfunc 

""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"键盘命令

""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""



nmap <leader>w :w!<cr>

nmap <leader>f :find<cr>



" 映射全选+复制 ctrl+a

map <C-A> ggVGY

map! <C-A> <Esc>ggVGY

map <F12> gg=G

" 选中状态下 Ctrl+c 复制

vmap <C-c> "+y

"去空行  

nnoremap <F2> :g/^\s*$/d<CR> 

"比较文件  

nnoremap <C-F2> :vert diffsplit 

"新建标签  

map <M-F2> :tabnew<CR>  

"列出当前目录文件  

map <F3> :tabnew .<CR>  

"打开树状文件目录  

map <C-F3> \be  

"C，C++ 按F5编译运行

map <F5> :call CompileRunGcc()<CR>

func! CompileRunGcc()

    exec "w"

    if &filetype == 'c'

        exec "!g++ % -o %<"

        exec "! ./%<"

    elseif &filetype == 'cpp'

        exec "!g++ % -o %<"

        exec "! ./%<"

    elseif &filetype == 'java' 

        exec "!javac %" 

        exec "!java %<"

    elseif &filetype == 'sh'

        :!./%

    endif

endfunc

"C,C++的调试

map <F8> :call Rungdb()<CR>

func! Rungdb()

    exec "w"

    exec "!g++ % -g -o %<"

    exec "!gdb ./%<"

endfunc

""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

""实用设置

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

" 设置当文件被改动时自动载入

set autoread

" quickfix模式

autocmd FileType c,cpp map <buffer> <leader><space> :w<cr>:make<cr>

"代码补全 

set completeopt=preview,menu 

"允许插件  

filetype plugin on

"共享剪贴板  

set clipboard+=unnamed 

"从不备份  

set nobackup

"make 运行

:set makeprg=g++\ -Wall\ \ %

"自动保存

set autowrite

set ruler                   " 打开状态栏标尺

set cursorline              " 突出显示当前行

set magic                   " 设置魔术

set guioptions-=T           " 隐藏工具栏

set guioptions-=m           " 隐藏菜单栏

"set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)\

" 设置在状态行显示的信息

set foldcolumn=0

set foldmethod=indent 

set foldlevel=3 

set foldenable              " 开始折叠

" 不要使用vi的键盘模式，而是vim自己的

set nocompatible

" 语法高亮

set syntax=on

" 去掉输入错误的提示声音

set noeb

" 在处理未保存或只读文件的时候，弹出确认

set confirm

" 自动缩进

set autoindent

set cindent

" Tab键的宽度

set tabstop=4

" 统一缩进为4

set softtabstop=4

set shiftwidth=4

" 不要用空格代替制表符

set noexpandtab

" 在行和段开始处使用制表符

set smarttab

" 显示行号

set number

" 历史记录数

set history=1000

"禁止生成临时文件

set nobackup

set noswapfile

"搜索忽略大小写

set ignorecase

"搜索逐字符高亮

set hlsearch

set incsearch

"行内替换

set gdefault

"编码设置

set enc=utf-8

set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936

"语言设置

set langmenu=zh_CN.UTF-8

set helplang=cn

" 我的状态行显示的内容（包括文件类型和解码）

"set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%l,%v][%p%%]\ %{strftime(\"%d/%m/%y\ -\ %H:%M\")}

"set statusline=[%F]%y%r%m%*%=[Line:%l/%L,Column:%c][%p%%]

" 总是显示状态行

set laststatus=2

" 命令行（在状态行下）的高度，默认为1，这里是2

set cmdheight=2

" 侦测文件类型

filetype on

" 载入文件类型插件

filetype plugin on

" 为特定文件类型载入相关缩进文件

filetype indent on

" 保存全局变量

set viminfo+=!

" 带有如下符号的单词不要被换行分割

set iskeyword+=_,$,@,%,#,-

" 字符间插入的像素行数目

set linespace=0

" 增强模式中的命令行自动完成操作

set wildmenu

" 使回格键（backspace）正常处理indent, eol, start等

set backspace=2

" 允许backspace和光标键跨越行边界

set whichwrap+=<,>,h,l

" 可以在buffer的任何地方使用鼠标（类似office中在工作区双击鼠标定位）

set mouse=a

set selection=exclusive

set selectmode=mouse,key

" 通过使用: commands命令，告诉我们文件的哪一行被改变过

set report=0

" 在被分割的窗口间显示空白，便于阅读

set fillchars=vert:\ ,stl:\ ,stlnc:\

" 高亮显示匹配的括号

set showmatch

" 匹配括号高亮的时间（单位是十分之一秒）

set matchtime=1

" 光标移动到buffer的顶部和底部时保持3行距离

set scrolloff=3

" 为C程序提供自动缩进

set smartindent

" 高亮显示普通txt文件（需要txt.vim脚本）

au BufRead,BufNewFile *  setfiletype txt

"自动补全

:inoremap ( ()<ESC>i

:inoremap ) <c-r>=ClosePair(')')<CR>

:inoremap { {<CR>}<ESC>O

:inoremap } <c-r>=ClosePair('}')<CR>

:inoremap [ []<ESC>i

:inoremap ] <c-r>=ClosePair(']')<CR>

:inoremap " ""<ESC>i

:inoremap ' ''<ESC>i

function! ClosePair(char)

    if getline('.')[col('.') - 1] == a:char

        return "\<Right>"

    else

        return a:char

    endif

endfunction

filetype plugin indent on 

"打开文件类型检测, 加了这句才可以用智能补全

set completeopt=longest,menu

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

" CTags的设定  

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

let Tlist_Sort_Type = "name"    " 按照名称排序  

let Tlist_Use_Right_Window = 1  " 在右侧显示窗口  

let Tlist_Compart_Format = 1    " 压缩方式  

let Tlist_Exist_OnlyWindow = 1  " 如果只有一个buffer，kill窗口也kill掉buffer  

let Tlist_File_Fold_Auto_Close = 0  " 不要关闭其他文件的tags  

let Tlist_Enable_Fold_Column = 0    " 不要显示折叠树  

autocmd FileType java set tags+=D:\tools\java\tags  

"autocmd FileType h,cpp,cc,c set tags+=D:\tools\cpp\tags  

"let Tlist_Show_One_File=1            "不同时显示多个文件的tag，只显示当前文件的

"设置tags  

set tags=tags  

"set autochdir 



""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"其他东东

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"默认打开Taglist 

let Tlist_Auto_Open=1 

"""""""""""""""""""""""""""""" 

" Tag list (ctags) 

"""""""""""""""""""""""""""""""" 

let Tlist_Ctags_Cmd = '/usr/bin/ctags' 

let Tlist_Show_One_File = 1 "不同时显示多个文件的tag，只显示当前文件的 

let Tlist_Exit_OnlyWindow = 1 "如果taglist窗口是最后一个窗口，则退出vim 

let Tlist_Use_Right_Window = 1 "在右侧窗口中显示taglist窗口

" minibufexpl插件的一般设置

let g:miniBufExplMapWindowNavVim = 1

let g:miniBufExplMapWindowNavArrows = 1

let g:miniBufExplMapCTabSwitchBufs = 1
let g:miniBufExplModSelTarget = 1
```