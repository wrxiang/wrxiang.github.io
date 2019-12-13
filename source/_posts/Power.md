---
title: PowerDesigner 逆向工程
date: 2019-12-13 14:01:19
categories: PowerDesigner
tags: [PowerDesigner, 逆向工程]
---
PowerDesigner：16.5.0.3982
数据库：Oracle 11g

PowerDesigner使用反向工程功能连接Oracle数据库自动生成模型以及过程中出现的问题
<!--more-->
### 连接数据库
首先打开PowerDesigner，依次点击 <span id="inline-blue">文件</span> -->  <span id="inline-blue">反向工程</span> --> <span id="inline-blue">Database...</span> ，如下图所示：
![1576217565_1_.png](https://i.loli.net/2019/12/13/KC6ePAoFTSVQrxE.png)
然后弹出以下界面，模型名称随便取就好，DBMS选择ORACLE_Version_11g，然后点击确定
![20191213141601.png](https://i.loli.net/2019/12/13/WuxwMGQohPm52JT.png)
在弹出的界面中选择使用数据源，然后点击旁边的圆柱图标配置连接
![1576217565.png](https://i.loli.net/2019/12/13/SWzAlLcNyXnjgE3.png)
在弹出的界面中选择 'Connections profile'，然后点击<span id="inline-blue">Configure...</span>按钮
![1576217565.png](https://i.loli.net/2019/12/13/paSnWhyiQ96Tf4m.png)
在弹出的界面中选择第三个页签 Connetion Profiles，然后点击添加数据源按钮（快捷键 Ctrl + N）
![1576217565.png](https://i.loli.net/2019/12/13/SfiQ6tchBEgmHWy.png)
然后在弹出的界面中填写建立数据库连接的相关信息，这里我们选择JDBC的连接方式，主要是因为ODBC的连接方式需要安装Oracle完整版客户端，我电脑上使用的是绿色版的客户端，不想再安装了，JDBC方式需要使用Oracle驱动的jar包，需要提前准备好
![1576217565.png](https://i.loli.net/2019/12/13/wZ2sVFKlH9vtgSx.png)
配置完成点击 <span id="inline-blue">Test Connection</span> 按钮输入用户名以及密码可以测试连接是否成功。
测试连接时有可能会有报错，PowerDesigner控制台打印出 `Could not Initialize JavaVM!`，解决方案如下：
1. 检查安装的JDK是否时32位，需要使用32位JDK，64位报错不能使用，因为PowerDesigner是32位程序
2. 设置JAVA_HOME环境变量 set JAVA_HOME=C:\Program Files (x86)\Java\jdk1.8.0_25;
3. 设置CLASSPATH环境变量 set CLASSPATH=.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;
4. 将JAVA_HOME设置进系统环境变量中 set path=JAVA_HOME\bin;
5. PowerDesigner软件中，工具-->常规选项-->variables 中设置JAR,JAVA,JAVAC,JAVADOC路径，他们的路径就是安装的32位的jdk里面的bin目录下的 对应名称的可执行文件

上述设置完成后就可以连接成功了，然后点击确定回到数据库连接界面，选择配置好的Oracle数据连接，输入用户名以及密码，点击<span id="inline-blue">connect</span>连接数据库
![1576217565.png](https://i.loli.net/2019/12/13/IgEOzxSPKpq5aji.png)
![1576217565.png](https://i.loli.net/2019/12/13/ue2jsLR7pqYmJrf.png)
然后在弹出的数据库反向引擎界面选择需要的用户、表或者视图等，点击确定会自动生成模型。
![1576217565.png](https://i.loli.net/2019/12/13/9yp6OvIna3KMkeN.png)
默认生成的模型CODE和NAME都是表字段，没有将数据库字段的注释自动生成出来，需要进行以下操作：
点击 <span id="inline-blue">工具</span>--> <span id="inline-blue">Execute Commonds</span> --> <span id="inline-blue">Edit/Run Script...</span>，然后将以下内容输入弹出的窗口中，然后点击Run按钮就会将数据库注释添加到模型的Name上。
```
Option   Explicit 
ValidationMode   =   True 
InteractiveMode   =   im_Batch
 
Dim   mdl   '   the   current   model
 
'   get   the   current   active   model 
Set   mdl   =   ActiveModel 
If   (mdl   Is   Nothing)   Then 
  MsgBox   "There   is   no   current   Model " 
ElseIf   Not   mdl.IsKindOf(PdPDM.cls_Model)   Then 
  MsgBox   "The   current   model   is   not   an   Physical   Data   model. " 
Else 
  ProcessFolder   mdl 
End   If
 
Private   sub   ProcessFolder(folder) 
On Error Resume Next
  Dim   Tab   'running table 
  for   each   Tab   in   folder.tables 
if   not   tab.isShortcut   then 
  tab.name   =   tab.comment
  Dim   col   '   running   column 
  for   each   col   in   tab.columns 
  if col.comment="" then
  else
col.name=   col.comment 
  end if
  next 
end   if 
  next
 
  Dim   view   'running   view 
  for   each   view   in   folder.Views 
if   not   view.isShortcut   then 
  view.name   =   view.comment 
end   if 
  next
 
  '   go   into   the   sub-packages 
  Dim   f   '   running   folder 
  For   Each   f   In   folder.Packages 
if   not   f.IsShortcut   then 
  ProcessFolder   f 
end   if 
  Next 
end   sub
```

