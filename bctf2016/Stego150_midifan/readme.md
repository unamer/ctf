#Stego_MIDI   
> 	这道题应该都知道是魂斗罗＾＿＾，经过小小的google，找到了其源文件（然而头信息还是对不上），想到了可能在音符方面有古怪，然而不知道如何找。Audition打不开MIDI文件（后来知道Cakewalk、Cubase可以，一看就知道问题），于是就死在这里了。  

##1.MIDI:
>	MIDI用音符的数字控制信号来记录音乐，能包含数十条音乐轨道，MIDI 传输的不是声音信号， 而是音符、控制参数等指令, 它指示MIDI 设备要做什么，怎么做， 如演奏哪个音符、多大音量等。它们被统一表示成MIDI 消息(MIDI Message) 。MIDI音符以微秒做计时单位。 

##2.问题
>	”所有的MIDI文件，应指定节奏和拍号。如果他们不这样做，拍号假设为4 / 4 ，节奏和节拍120每分钟。“也就是说，MIDI文件是要以节拍为分组的，再高超的艺术家也不会在演奏时弹出1/10^6秒的音符。如果有，那就是故意的。＾＿＾  

##3.看懂MIDI
>	MIDI可以被解析成各种格式：XML、CSV等。我们用MIDICSV将MIDI解析成CSV格式的文件（百度就好）。第一列是音轨，第二列就是音符时长了。观察文件，一系列行的音符时长不正常。因此，提取即可。（代码还没弄好，正在学Python中）  

##4.MIDI => CSV
> midi转换成csv之后是这个样子的  
```
1,48, Note_off_c,0,72,64,
1,48, Note_off_c,1,77,64,
1,60, Note_on_c,2,63,80,
1,60, Note_off_c,2,65,64,
1,60, Note_on_c,1,75,80,
1,61, Note_on_c,0,70,80,
1,108, Note_off_c,0,70,64,
1,108, Note_off_c,1,75,64,
1,120, Note_on_c,0,67,80,
1,120, Note_on_c,1,72,80,
1,120, Note_off_c,2,63,64,
1,120, Note_on_c,2,60,80,
1,168, Note_off_c,0,67,64,
1,168, Note_off_c,1,72,64,
1,180, Note_on_c,0,65,80,
1,180, Note_on_c,1,70,80,
1,180, Note_off_c,2,60,64,
1,180, Note_on_c,2,58,80,
1,228, Note_off_c,0,65,64,
1,228, Note_off_c,1,70,64,
1,240, Note_on_c,0,67,80,
1,240, Note_on_c,1,72,80,
1,240, Note_off_c,2,58,64,
1,240, Note_on_c,2,60,80,
1,288, Note_off_c,0,67,64,
1,288, Note_off_c,1,72,64,
1,300, Note_on_c,0,65,80,
1,300, Note_on_c,1,70,80,
1,300, Note_off_c,2,60,64,
1,300, Note_on_c,2,58,80,
1,348, Note_off_c,0,65,64,
......
```
可以看到第二列有一些奇怪的数字，比如61 481 ....
首先替换掉最后一个不是80的行，因为只有80才是有用的，我们发现除了80就是64  
使用以下正则表达式替换，替换完删除头尾一些无用的数据，然后用第二个正则替换，现在就留下一些有用的数据了。
```regex
^.+64,$\r\n
^.+Note_on_c,(1|2|9).+$\r\n
```
正则表达式是相当神奇的，现在只留下这样的东西了
```
1,61, Note_on_c,0,70,80,
1,120, Note_on_c,0,67,80,
1,180, Note_on_c,0,65,80,
1,240, Note_on_c,0,67,80,
1,300, Note_on_c,0,65,80,
1,361, Note_on_c,0,63,80,
1,420, Note_on_c,0,62,80,
1,481, Note_on_c,0,63,80,
1,541, Note_on_c,0,62,80,
1,600, Note_on_c,0,60,80,
1,660, Note_on_c,0,58,80,
1,720, Note_on_c,0,58,80,
1,780, Note_on_c,0,55,80,
1,841, Note_on_c,0,58,80,
1,900, Note_on_c,0,59,80,
1,960, Note_on_c,0,60,80,
1,1440, Note_on_c,0,67,80,
1,1561, Note_on_c,0,65,80,
1,1620, Note_on_c,0,67,80,
1,1741, Note_on_c,0,68,80,
1,1800, Note_on_c,0,70,80,
1,2881, Note_on_c,0,60,80,
1,3360, Note_on_c,0,67,80,
1,3480, Note_on_c,0,65,80,
1,3541, Note_on_c,0,67,80,
1,3661, Note_on_c,0,69,80,
1,3720, Note_on_c,0,63,80,
1,4800, Note_on_c,0,60,80,
1,5280, Note_on_c,0,67,80,
...
```
OK,继续提取，替换下面正则表达式为\1
```
^1,\d+(\d),.+$
```
然后变成这个样子
```
1
0
0
0
0
1
0
1
1
0
0
0
0
1
0
0
0
1
0
1
0
1
0
0
1
1
0
....
```
OK,继续，替换掉全部换行，然后在最前面加个0
于是我们得到了这么个字符串
```
010000101100001000101010011000101101111001010110101011100110110000101110111110100000110001110110101001101111101001000110100011000010111011111010111101100110011011111010001001101000110001100010011000101011111000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```

##5.提取flag
> 用以下vbs脚本提取flag
```vbs
function BinToDec(binStr)
    dim i
    for i = 1 to Len(binStr)
        BinToDec = BinToDec + (CInt(Mid(binStr, i, 1)) * (2 ^ (Len(binStr) - i)))
    next
end function
r="010000101100001000101010011000101101111001010110101011100110110000101110111110100000110001110110101001101111101001000110100011000010111011111010111101100110011011111010001001101000110001100010011000101011111000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
For i=1 To Len(r) Step 8
	b=chr(BinToDec(StrReverse(Mid(r,i,8))))
	flag=flag+b
Next
wsh.echo flag
```
最终flag: BCTF{ju6t_0ne_b1t_of_d1FF}