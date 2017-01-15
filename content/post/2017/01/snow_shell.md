+++
date = "2017-01-15T22:08:38+08:00"
title = "一段让屏幕下雪的shell脚本"
categories = ["Technology", "Hacking"]
tags = ["shell", "script"]

+++
![snow](/images/2017/01/snow.gif)

在网上一段[shell](http://jerrygamblin.com/2016/12/21/making-it-snow-in-your-terminal/ "Jerry Gamblin" target="_blank")脚本，运行后可以在屏幕上实现下雪的效果，这段只有一行的代码实在让人佩服。

{{< highlight shell "linenos=inline,style=manni" >}}
for((I=0;J=--I;))do clear;for((D=LINES;S=++J**3%COLUMNS,--D;))do printf %*s.\\n $S;done;sleep .1;done
{{< /highlight >}}


不过原代码虽然简洁但不太容易一下子看明白工作原理，而且只能在命令行上运行无法保存成脚本文件，所以自己按照理解重新写了一遍：
{{< highlight shell "linenos=inline,style=manni" >}}
#!/bin/sh

rows=$(tput lines)
cols=$(tput cols)

i=0
while true
do
    clear
    ((x=i,i++))
    for((r=0;r<rows;r++))
    do
        printf "%*s*\n" $((x**3%cols))
        ((x--))
    done
    sleep .1
done
{{< /highlight >}}
