Настройка астры

Установить openssh-server
/Etc/ssh/sshd-config
PermitRootLogin=yes
PasswordAuthentication=yes
Sudo service ssh restart


Ip route 
Ip a
ip route add default via 172.16.31.222
/etc/resolv.conf
nameserver 172.16.31.222
nameserver 172.16.31.218

/etc/apt/sources.list