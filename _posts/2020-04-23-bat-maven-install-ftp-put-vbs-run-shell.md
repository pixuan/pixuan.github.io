---
layout: post
title:  "use bat script for maven install & ftp put to remote server & vbs script to login by ssh"
date:   2020-04-23 01:00:00 +0800
categories: bat maven ftp vbs 
---

### maven
参考 <https://blog.csdn.net/turkeyzhou/article/details/9264431>  
mvn本身也是.bat脚本，使用call，否则程序打包结束会自动退出，可使用pause暂停，手动继续  
`maven脚本`
```bat
@echo off
cd /xx/xx
call mvn -Dmaven.test.skip -U clean package install
pause
```

### ftp
参考 <https://www.cnblogs.com/jjzd/p/6229160.html>  
脚本分为两部分：可执行bat脚本和ftp命令文件  
注意命名规范，最好不要叫ftp.bat,建议命名为upload.bat这样可能导致该脚本循环不停调用自身，因为脚本里会有ftp命令  
`可执行bat脚本`：
```bat
@echo off
call ftp -s:D:\ftp\ftp.txt
pause
```
lcd为本地目录，cd为远端目录，mput可上传多个，put上传单个  
`ftp命令ftp.txt`：
```txt
open 192.168.1.1
ftp-user
ftp-passwd
prompt off
lcd D:\ftp
cd /home/myftp
mput *
prompt on
bye
quit
```

### vbs
参考 <https://m.jb51.net/article/19399.htm>  
<https://stackoverflow.com/questions/10521300/what-is-correct-syntax-to-use-awk-in-vbscript/61378908#61378908>  
<https://www.52pojie.cn/thread-1003908-1-1.html>  
bat脚本通过ssh免输入密码登录，可以用vbs脚本实现, 如下脚本实现远程登录linux服务器，并杀掉旧应用进程，再重启该应用，最后退出命令行窗口功能  
{% raw %}
```vbs
Dim WshShell 
Set WshShell=WScript.CreateObject("WScript.Shell") 
WshShell.Run "cmd.exe"
WScript.Sleep 1500 
WshShell.SendKeys "ssh root@192.168.1.1"
WshShell.SendKeys "{ENTER}"
WScript.Sleep 1500 
WshShell.SendKeys "root"
WshShell.SendKeys "{ENTER}"
WScript.Sleep 1500 
WshShell.SendKeys "cd /home/xxx"
WshShell.SendKeys "{ENTER}"
WScript.Sleep 1500 
WshShell.SendKeys "kill -SIGTERM `ps -ef|grep 'java -jar'|grep -v grep|awk '{{}print $2{}}'`"
WshShell.SendKeys "{ENTER}"
WScript.Sleep 1500 
WshShell.SendKeys "java -jar xxx.jar >> run.log &"
WshShell.SendKeys "{ENTER}"
WScript.Sleep 1500 
WshShell.SendKeys "exit"
WshShell.SendKeys "{ENTER}"
WScript.Sleep 1500 
WshShell.SendKeys "exit"
WshShell.SendKeys "{ENTER}"
```
{% endraw %}
vbs对于特殊字符，可以使用{}包围该字符，如使用{&#123;}发送&#123;字符  
此处由于jekyll的正则规则`Liquid syntax error Variable  was not properly terminated with regexp: /\\}\\}/`, 导致vbs示例代码中只能被迫使用单括号，其实此处正确应该使用{}包裹住&#123;字符。 为了避免该问题，md文件中代码块采用`raw endraw`方式包裹。大括号到处都是坑啊 :( <https://stackoverflow.com/questions/3426182/how-to-escape-liquid-template-tags>   
bat脚本可通过`start run.vbs`运行run.vbs脚本。


### bat
对于整个bat脚本，可综合到一个文件进行执行
```bat
@echo off
cd /xx/xx
call mvn -Dmaven.test.skip -U clean package install
pause
call ftp -s:D:\ftp\ftp.txt
pause
cd /xx/xx
start run.vbs
```