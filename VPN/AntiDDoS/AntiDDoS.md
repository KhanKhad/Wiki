# 1 Способ

Защищаемся от SYN Flood 

1. Проверяем атакуют ли нас через netstat -ant | grep SYN_RECV или ss -t state SYN-RECV|wc -l, если видим много подключений - атакуют
2. Идём в файл /etc/sysctl.conf используя nano/vim и добавляем строку net.ipv4.tcp_syncookies = 2 (0 - Механизм SYN cookies отключен, 1 - включен, если много запросов на подключение, 2 - постоянно включен)
3. Применяем правила sudo sysctl -p
4. Профит, бразильцев больше нет

тестировалось не очень долго, но по первым ощущениям на тот же vless это не влияет

# 2 Способ
Защита от SYN FLOOD атак
Для начала проверяем, действительно ли это Syn Flood:
netstat -n | grep SYN_RECV | wc -l
Если больше 5 соединений, значит, это syn атака.

1) Вводим для iptables правила ниже:
iptables -N syn_flood
iptables -A INPUT -p tcp --syn -j syn_flood
iptables -A syn_flood -m limit --limit 30/s --limit-burst 100 -j RETURN
iptables -A syn_flood -j DROP

2) Устанавливаем лимиты в /etc/sysctl.conf:
net.ipv4.tcp_max_syn_backlog = 40000
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_syncookies = 1
net.core.somaxconn = 60000
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_keepalive_time = 15
net.core.netdev_max_backlog = 40000
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.conf.default.rp_filter = 1
net.ipv4.tcp_max_syn_backlog = 4096

net.ipv4.tcp_window_scaling = 0
net.ipv4.tcp_sack = 0
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1

net.netfilter.nf_conntrack_max=400000
net.netfilter.nf_conntrack_tcp_timeout_close = 5
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 5
net.netfilter.nf_conntrack_tcp_timeout_last_ack = 1
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 5
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 1
net.netfilter.nf_conntrack_tcp_timeout_established = 30
net.netfilter.nf_conntrack_tcp_timeout_syn_recv = 1
net.netfilter.nf_conntrack_tcp_timeout_syn_sent = 1
net.netfilter.nf_conntrack_tcp_loose = 0
Принимаем настройки:
sysctl -p

3) Через Iptables ставим ограничение числа одновременных коннектов к веб-серверу с одного ip:
upd 10/01/25: Внимание! Следующая команда также ограничивает работу vless соединений.
iptables -I INPUT -p tcp --syn --dport 443 -m connlimit --connlimit-above 3 --connlimit-mask 24 -j DROP

К использованию на сервере с vless-reality не рекомендуется.