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
使用以下正则表达式替换，替换完删除头尾一些无用的数据
```regex
^.+64,$\r\n
```
然后使用cmd命令提取第二列。
```cmd
for /f "tokens=2 delims=," %a in (midifan.csv) do echo %a>>column2.txt
```
最后使用一下vbs去除重复行，只需要把txt拖到vbs上就行了 
```vbs
on error resume next
Const adOpenStatic = 3 
Const adLockOptimistic = 3 
Const adCmdText = &H0001 
set args = wscript.arguments
Set objConnection = CreateObject("ADODB.Connection") 
set f=createobject("scripting.filesystemobject")
Set objRecordSet = CreateObject("ADODB.Recordset") 
strPathToTextFile =f.GetParentFolderName(args(0))
strFile = f.GetFileName(args(0)) 
objConnection.Open "Provider=Microsoft.Jet.OLEDB.4.0;Data Source=" & strPathtoTextFile & ";Extended Properties=""text;HDR=NO;FMT=Delimited""" 
objRecordSet.Open "Select DISTINCT * FROM " & strFile,objConnection, adOpenStatic, adLockOptimistic, adCmdText 
set of=f.opentextfile(mid(args(0),1,instrrev(args(0),".")-1)&"_.txt",8,true)
Do Until objRecordSet.EOF 
    of.WriteLine objRecordSet.Fields.Item(0).Value    
    objRecordSet.MoveNext 
Loop 
Set of=Nothing
MsgBox "OK!"
```