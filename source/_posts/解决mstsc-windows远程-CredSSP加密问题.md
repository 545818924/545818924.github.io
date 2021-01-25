---
title: 解决mstsc(windows远程)CredSSP加密问题
date: 2021-01-13 14:27:09
tags:
---


Win10远程桌面出现身份验证错误：要求的函数不受支持，这可能是由于CredSSP加密Oracle修正 解决方法



![img](/images/CredSSP.png)



网上方案很多，不过最合适的方案是：https://github.com/stascorp/rdpwrap/issues/480



开始输入cmd，使用管理员打开，输入 

`REG ADD HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\CredSSP\Parameters\ /v AllowEncryptionOracle /t REG_DWORD /d 2`

回车即可。