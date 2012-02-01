---
layout: post
title: SVN�汾��ȫ�����������ݽű�(windowsƽ̨)
excerpt: SVN�汾��ȫ�����������ݽű��������˱�����ɷ����ʼ��Ĺ���
comments: true
---

��ǰд��linuxƽ̨��SVN���ݽű������ڵĵ�λ�İ汾������windowsƽ̨�ģ�������ʷԭ���ݲ���Ǩ�ƣ�����д��windowsƽ̨�ı��ݽű�

1 ȫ�����ݽű�
{% highlight dos %}
@echo off

rem ######################################################
rem # Subversion full backup script
rem # Env: Windows Server 2008
rem # Created by kenny on 2012-01-29
rem # Contact: nostream@163.com
rem # Version: 1.0.0
rem ######################################################

rem �������ڸ�ʽ
set time_hh=%time:~0,2%
if /i %time_hh% LSS 10 (set time_hh=0%time:~1,1%)
set DT=%date:~,4%%date:~5,2%%date:~8,2%
set DA=%date:~,4%%date:~5,2%%date:~8,2%_%time_hh%%time:~3,2%%time:~6,2%

rem �����ʼ���Ϣ
set from=scm@shunwang.com
set server=smtp.shunwang.com
set to=dc.wang@shunwang.com
set subj1=SVN�汾��ȫ���������
set subj2=SVN�汾��ȫ������ʧ��

rem Subversion�İ�װĿ¼
set SVN_HOME="C:\Progra~1\Visual~1\bin"
set SVN_LOOK=%SVN_HOME%\svnlook.exe
rem ���а汾��ĸ�Ŀ¼
set SVN_ROOT=D:\Reposi~1
rem ����Ŀ¼
set BACKUP_SVN_ROOT=E:\svn_full_backup
set BACKUP_DIRECTORY=%BACKUP_SVN_ROOT%\%DA%
rem ȫ�����ݽű�Ŀ¼
set FULL_PATH=E:\svn_back_scripts\fullbackup
rem �������ݽű�Ŀ¼
set INCRE_PATH=E:\svn_back_scripts\increment

echo "-------------------------------------" >> %FULL_PATH%/fullbackup.log
echo "-------------------------------------" >> %FULL_PATH%/fullbackup.log
echo "�������� %date:~0,10% " >> %FULL_PATH%/fullbackup.log

if exist %BACKUP_DIRECTORY% (
	echo ����Ŀ¼%BACKUP_DIRECTORY%�Ѿ����ڣ������>> %FULL_PATH%/fullbackup.log
) else (
	echo ��������Ŀ¼%BACKUP_DIRECTORY% >> %FULL_PATH%/fullbackup.log
	mkdir %BACKUP_DIRECTORY%
)

rem �����û���Ȩ���ļ�
xcopy %SVN_ROOT%\authz %BACKUP_DIRECTORY%
xcopy %SVN_ROOT%\htpasswd %BACKUP_DIRECTORY%

rem ��֤Ŀ¼�Ƿ�Ϊ�汾�⣬�������ȡ�����Ʊ���
for /r %SVN_ROOT% %%I in (.) do (
	if exist "%%I\conf\svnserve.conf" (
		call :hotcopy %%~fI %%~nI
	)
)

rem �����ʼ�֪ͨSCM�汾�ⱸ�ݽ��
if %errorlevel% == 0 (
   blat %FULL_PATH%\fullbackup.log -to %to% -base64 -charset GB2312 -subject %subj1% -server %server% -f %from%
) else (
   blat %FULL_PATH%\fullbackup.log -to %to% -base64 -charset GB2312 -subject %subj1% -server %server% -f %from%
)


:hotcopy
if "%2"=="" (
	echo �汾�ⱸ�����
) else (
	echo ׼����ʼȫ�����ݰ汾��%1 >> %FULL_PATH%/fullbackup.log
	%SVN_HOME%\svnadmin hotcopy %1 %BACKUP_DIRECTORY%\%2
	echo �汾��%2������� >> %FULL_PATH%/fullbackup.log
	if not exist %INCRE_PATH%\%2 mkdir %INCRE_PATH%\%2
	%SVN_LOOK% youngest %1 > %INCRE_PATH%\%2\%2_last_revision.txt
)
goto :EOF

{% endhighlight %}

2 �������ݽű�
{% highlight dos %}
@echo off

rem ######################################################
rem # Subversion full backup script
rem # Env: Windows Server 2008
rem # Created by kenny on 2012-01-29
rem # Contact: nostream@163.com
rem # Version: 1.0.0
rem ######################################################

rem �������ڸ�ʽ
set time_hh=%time:~0,2%
if /i %time_hh% LSS 10 (set time_hh=0%time:~1,1%)
set DT=%date:~,4%%date:~5,2%%date:~8,2%
set DA=%date:~,4%%date:~5,2%%date:~8,2%_%time_hh%%time:~3,2%%time:~6,2%

rem �����ʼ���Ϣ
set from=scm@shunwang.com
set server=smtp.shunwang.com
set to=dc.wang@shunwang.com
set subj1=SVN�汾�������������
set subj2=SVN�汾����������ʧ��

rem Subversion�İ�װĿ¼��ִ���ļ�
set SVN_HOME="C:\Progra~1\Visual~1\bin"
set SVN_ADMIN=%SVN_HOME%\svnadmin.exe
set SVN_LOOK=%SVN_HOME%\svnlook.exe

rem ���ÿ�ֿ��Ŀ¼
set SVN_REPOROOT=D:\Reposi~1

rem ���������ļ����·��
set INCRE_BACKUP=E:\svn_increment_backup

rem �������ݽű�Ŀ¼
set INCRE_PATH=E:\svn_back_scripts\increment

echo "-------------------------------------" >> %INCRE_PATH%/incre.log
echo "-------------------------------------" >> %INCRE_PATH%/incre.log
echo "�������� %date:~0,10% " >> %INCRE_PATH%/incre.log

rem ��֤Ŀ¼�Ƿ�Ϊ�汾�⣬�������ȡ�����Ʊ���
for /r %SVN_ROOT% %%I in (.) do (
	if exist "%%I\conf\svnserve.conf" (
		call :incre_dump %%~nI
	)
)

rem �����ʼ�֪ͨSCM�汾�ⱸ�ݽ��
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
echo ��Ŀ��%PROJECT%��ʼ����>> %INCRE_PATH%/incre.log
%SVN_LOOK% youngest %SVN_REPOROOT%\%PROJECT%> %INCRE_PATH%\A.TMP
FOR /f %%D IN (%INCRE_PATH%\A.TMP) DO set UPPER=%%D
if %UPPER%==0 GOTO :N_EXIT
if not exist %INCRE_PATH%\%PROJECT%\%PROJECT%_last_revision.txt GOTO :BACKUP

rem ȡ���ϴα��ݺ�İ汾�ţ�������1����(ע����㷨δ��98ϵͳ��֤)
FOR /f %%C IN (%INCRE_PATH%\%PROJECT%\%PROJECT%_last_revision.txt) DO set LOWER=%%C
set /A LOWER=%LOWER%+1

rem ����Ҫ���ݣ�����ת����
IF %LOWER% gtr %UPPER% GOTO :N_EXIT

:BACKUP
SET FILENAME=%PROJECT%_%LOWER%_%UPPER%
echo ��ʼ������Ŀ�⣺%PROJECT%�������ļ�=%FILENAME% >> %INCRE_PATH%/incre.log
%SVN_ADMIN% dump %SVN_REPOROOT%\%PROJECT% -r %LOWER%:head --incremental >%FILENAME%.dmp

rem ׼��д������־��Ϣ
IF %LOWER% gtr 0 GOTO :WRITENOTE
echo %PROJECT%����ʱ��: %date% >> %INCRE_PATH%/incre.log
echo %PROJECT%����revision���� ��[%LOWER%]��[%UPPER%] >> %INCRE_PATH%/incre.log
GOTO :COMPLETE

:WRITENOTE
echo %date% >> %INCRE_PATH%/incre.log
echo �汾��%PROJECT%�ļ�Ϊ�� >> %INCRE_PATH%/incre.log

:COMPLETE
rem ����һ�����ڿ��������ļ���Ŀ���ַ
echo ��dump�����ļ�%FILENAME%.dmp ת����%INCRE_BACKUP%\%PROJECT% Ŀ¼�� >> %INCRE_PATH%/incre.log
move %FILENAME%.dmp %INCRE_BACKUP%\%PROJECT%\
del %INCRE_PATH%\A.TMP
echo %UPPER% > %INCRE_PATH%\%PROJECT%\%PROJECT%_last_revision.txt

:N_EXIT
echo ��Ŀ��%PROJECT%���������>> %INCRE_PATH%/incre.log
exit /B

:no_args
ECHO ON

{% endhighlight %}