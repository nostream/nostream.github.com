---
layout: post
title: SVN版本库全量和增量备份脚本(windows平台)
excerpt: SVN版本库全量和增量备份脚本，增加了备份完成发送邮件的功能
comments: true
---

以前写过linux平台的SVN备份脚本，现在的单位的版本库是在windows平台的，由于历史原因，暂不做迁移，所以写了windows平台的备份脚本

1 全量备份脚本
{% highlight dos %}
@echo off

rem ######################################################
rem # Subversion full backup script
rem # Env: Windows Server 2008
rem # Created by kenny on 2012-01-29
rem # Contact: nostream@163.com
rem # Version: 1.0.0
rem ######################################################

rem 定义日期格式
set time_hh=%time:~0,2%
if /i %time_hh% LSS 10 (set time_hh=0%time:~1,1%)
set DT=%date:~,4%%date:~5,2%%date:~8,2%
set DA=%date:~,4%%date:~5,2%%date:~8,2%_%time_hh%%time:~3,2%%time:~6,2%

rem 定义邮件信息
set from=scm@shunwang.com
set server=smtp.shunwang.com
set to=dc.wang@shunwang.com
set subj1=SVN版本库全量备份完成
set subj2=SVN版本库全量备份失败

rem Subversion的安装目录
set SVN_HOME="C:\Progra~1\Visual~1\bin"
set SVN_LOOK=%SVN_HOME%\svnlook.exe
rem 所有版本库的根目录
set SVN_ROOT=D:\Reposi~1
rem 备份目录
set BACKUP_SVN_ROOT=E:\svn_full_backup
set BACKUP_DIRECTORY=%BACKUP_SVN_ROOT%\%DA%
rem 全量备份脚本目录
set FULL_PATH=E:\svn_back_scripts\fullbackup
rem 增量备份脚本目录
set INCRE_PATH=E:\svn_back_scripts\increment

echo "-------------------------------------" >> %FULL_PATH%/fullbackup.log
echo "-------------------------------------" >> %FULL_PATH%/fullbackup.log
echo "备份日期 %date:~0,10% " >> %FULL_PATH%/fullbackup.log

if exist %BACKUP_DIRECTORY% (
	echo 备份目录%BACKUP_DIRECTORY%已经存在，请清空>> %FULL_PATH%/fullbackup.log
) else (
	echo 建立备份目录%BACKUP_DIRECTORY% >> %FULL_PATH%/fullbackup.log
	mkdir %BACKUP_DIRECTORY%
)

rem 备份用户和权限文件
xcopy %SVN_ROOT%\authz %BACKUP_DIRECTORY%
xcopy %SVN_ROOT%\htpasswd %BACKUP_DIRECTORY%

rem 验证目录是否为版本库，如果是则取出名称备份
for /r %SVN_ROOT% %%I in (.) do (
	if exist "%%I\conf\svnserve.conf" (
		call :hotcopy %%~fI %%~nI
	)
)

rem 发送邮件通知SCM版本库备份结果
if %errorlevel% == 0 (
   blat %FULL_PATH%\fullbackup.log -to %to% -base64 -charset GB2312 -subject %subj1% -server %server% -f %from%
) else (
   blat %FULL_PATH%\fullbackup.log -to %to% -base64 -charset GB2312 -subject %subj1% -server %server% -f %from%
)


:hotcopy
if "%2"=="" (
	echo 版本库备份完成
) else (
	echo 准备开始全量备份版本库%1 >> %FULL_PATH%/fullbackup.log
	%SVN_HOME%\svnadmin hotcopy %1 %BACKUP_DIRECTORY%\%2
	echo 版本库%2备份完成 >> %FULL_PATH%/fullbackup.log
	if not exist %INCRE_PATH%\%2 mkdir %INCRE_PATH%\%2
	%SVN_LOOK% youngest %1 > %INCRE_PATH%\%2\%2_last_revision.txt
)
goto :EOF

{% endhighlight %}

2 增量备份脚本
{% highlight dos %}
@echo off

rem ######################################################
rem # Subversion full backup script
rem # Env: Windows Server 2008
rem # Created by kenny on 2012-01-29
rem # Contact: nostream@163.com
rem # Version: 1.0.0
rem ######################################################

rem 定义日期格式
set time_hh=%time:~0,2%
if /i %time_hh% LSS 10 (set time_hh=0%time:~1,1%)
set DT=%date:~,4%%date:~5,2%%date:~8,2%
set DA=%date:~,4%%date:~5,2%%date:~8,2%_%time_hh%%time:~3,2%%time:~6,2%

rem 定义邮件信息
set from=scm@shunwang.com
set server=smtp.shunwang.com
set to=dc.wang@shunwang.com
set subj1=SVN版本库增量备份完成
set subj2=SVN版本库增量备份失败

rem Subversion的安装目录及执行文件
set SVN_HOME="C:\Progra~1\Visual~1\bin"
set SVN_ADMIN=%SVN_HOME%\svnadmin.exe
set SVN_LOOK=%SVN_HOME%\svnlook.exe

rem 配置库仓库根目录
set SVN_REPOROOT=D:\Reposi~1

rem 增量备份文件存放路径
set INCRE_BACKUP=E:\svn_increment_backup

rem 增量备份脚本目录
set INCRE_PATH=E:\svn_back_scripts\increment

echo "-------------------------------------" >> %INCRE_PATH%/incre.log
echo "-------------------------------------" >> %INCRE_PATH%/incre.log
echo "备份日期 %date:~0,10% " >> %INCRE_PATH%/incre.log

rem 验证目录是否为版本库，如果是则取出名称备份
for /r %SVN_ROOT% %%I in (.) do (
	if exist "%%I\conf\svnserve.conf" (
		call :incre_dump %%~nI
	)
)

rem 发送邮件通知SCM版本库备份结果
if %errorlevel% == 0 (
   blat %INCRE_PATH%\incre.log -to %to% -base64 -charset GB2312 -subject %subj1% -server %server% -f %from%
) else (
   blat %INCRE_PATH%\incre.log -to %to% -base64 -charset GB2312 -subject %subj1% -server %server% -f %from%
)

:incre_dump
if "%1"=="" goto no_args
set PROJECT=%1
if not exist %INCRE_BACKUP%\%PROJECT% mkdir %INCRE_BACKUP%\%PROJECT%
cd %INCRE_BACKUP%\%PROJECT%
SET LOWER=0
SET UPPER=0

if not exist %INCRE_PATH%\%PROJECT% mkdir %INCRE_PATH%\%PROJECT%
echo 项目库%PROJECT%开始备份>> %INCRE_PATH%/incre.log
%SVN_LOOK% youngest %SVN_REPOROOT%\%PROJECT%> %INCRE_PATH%\A.TMP
FOR /f %%D IN (%INCRE_PATH%\A.TMP) DO set UPPER=%%D
if %UPPER%==0 GOTO :N_EXIT
if not exist %INCRE_PATH%\%PROJECT%\%PROJECT%_last_revision.txt GOTO :BACKUP

rem 取出上次备份后的版本号，并做＋1处理(注意此算法未在98系统验证)
FOR /f %%C IN (%INCRE_PATH%\%PROJECT%\%PROJECT%_last_revision.txt) DO set LOWER=%%C
set /A LOWER=%LOWER%+1

rem 不需要备份，则跳转结束
IF %LOWER% gtr %UPPER% GOTO :N_EXIT

:BACKUP
SET FILENAME=%PROJECT%_%LOWER%_%UPPER%
echo 开始备份项目库：%PROJECT%，生成文件=%FILENAME% >> %INCRE_PATH%/incre.log
%SVN_ADMIN% dump %SVN_REPOROOT%\%PROJECT% -r %LOWER%:head --incremental >%FILENAME%.dmp

rem 准备写备份日志信息
IF %LOWER% gtr 0 GOTO :WRITENOTE
echo %PROJECT%备份时间: %date% >> %INCRE_PATH%/incre.log
echo %PROJECT%备份revision区间 从[%LOWER%]到[%UPPER%] >> %INCRE_PATH%/incre.log
GOTO :COMPLETE

:WRITENOTE
echo %date% >> %INCRE_PATH%/incre.log
echo 版本库%PROJECT%文件为空 >> %INCRE_PATH%/incre.log

:COMPLETE
rem 下面一行用于拷贝备份文件到目标地址
echo 将dump备份文件%FILENAME%.dmp 转移至%INCRE_BACKUP%\%PROJECT% 目录下 >> %INCRE_PATH%/incre.log
move %FILENAME%.dmp %INCRE_BACKUP%\%PROJECT%\
del %INCRE_PATH%\A.TMP
echo %UPPER% > %INCRE_PATH%\%PROJECT%\%PROJECT%_last_revision.txt

:N_EXIT
echo 项目库%PROJECT%处理结束！>> %INCRE_PATH%/incre.log
exit /B

:no_args
ECHO ON

{% endhighlight %}