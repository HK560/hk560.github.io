---
title: 解决vscode终端执行hexo命令提示无法加载文件因为在此系统上禁止运行脚本
date: 2021-11-26 09:35:30
tags:
    - 教程
---
# 问题
今天在用vscode的终端创建hexo文章时候,执行hexo new 时候得到如下错误提示
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20211126093838.png)

# 原因
原因已经告诉我们了,因为当前系统上的powershell策略不允许我们执行脚本

# 解决
使用管理员身份打开powershell
输入命令 get-ExecutionPolicy
```powershell
PS C:\WINDOWS\system32> get-ExecutionPolicy
Restricted
```
返回结果为Restricted 微软doc说明如下:
> Restricted
The default execution policy for Windows client computers.
Permits individual commands, but does not allow scripts.
Prevents running of all script files, including formatting and configuration files (.ps1xml), module script files (.psm1), and PowerShell profiles (.ps1).

简单来说运行执行单独的命令,不允许执行脚本,阻止运行所有脚本文件,包括格式和配置文件(.ps1xml),模块脚本文件(.psm1)和powershell配置文件(.ps1)。

我们使用Set-ExecutionPolicy  RemoteSigned 修改策略
```powershell
PS C:\WINDOWS\system32> Set-ExecutionPolicy  RemoteSigned

执行策略更改
执行策略可帮助你防止执行不信任的脚本。更改执行策略可能会产生安全风险,如 https:/go.microsoft.com/fwlink/?LinkID=135170
中的 about_Execution_Policies 帮助主题所述。是否要更改执行策略?
[Y] 是(Y)  [A] 全是(A)  [N] 否(N)  [L] 全否(L)  [S] 暂停(S)  [?] 帮助 (默认值为“N”): Y

PS C:\WINDOWS\system32> get-ExecutionPolicy
RemoteSigned
```
这样就可以运行脚本了。回到vscode执行hexo new命令
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20211126095750.png)
发现已经可以正常执行了,问题解决!

## 说明
关于 RemoteSigned 说明:
> RemoteSigned
The default execution policy for Windows server computers.
Scripts can run.
Requires a digital signature from a trusted publisher on scripts and configuration files that are downloaded from the internet which includes email and instant messaging programs.
Doesn't require digital signatures on scripts that are written on the local computer and not downloaded from the internet.
Runs scripts that are downloaded from the internet and not signed, if the scripts are unblocked, such as by using the Unblock-File cmdlet.
Risks running unsigned scripts from sources other than the internet and signed scripts that could be malicious.

可以运行脚本,条件:
1.该脚本是在从互联网上下载的脚本和配置文件拥有可信发布者的数字签名,文件包括email和instant messaging programs.
2.本地的脚本则不需要数字签名也能运行
3.网上下载的没有签名的unblocked的脚本可以运行

谨防从Internet以外的来源运行未签名的脚本和可能是恶意的签名脚本。



-----


# Reference 
[microsoft.powershell.security](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/?view=powershell-7.2)

