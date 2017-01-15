+++
date = "2017-01-15T22:08:38+08:00"
title = "解析一段让屏幕下雪的shell"
categories = []
tags = []

+++
在网上一段shell脚本，运行后可以在屏幕上实现下雪的效果，这段只有一行的代码实在让人佩服。

<pre><code>
for((I=0;J=--I;))do clear;for((D=LINES;S=++J**3%COLUMNS,--D;))do printf %*s.\\n $S;done;sleep .1;done
</code></pre>


不过原代码虽然简洁但不太容易一下子看明白工作原理，所以自己按照理解重新写了一遍：
<pre><code>
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
</code></pre>

最终的运行效果如下：
[snow](/images/2017/01/snow.gif)
