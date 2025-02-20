1. Узнаем ip шлюза: ip route
2. ufw allow from IP шлюза
3. ufw deny from IP сервера  .0/ 24
4. Установить панель через скрипт из соседнего файла
5. Зайти в /etc/nginx/stream-enabled/stream.conf и сделать default 0;
6. Поменять порт ssh (и фаервол)
7. Монжо включить net.ipv4.tcp_syncookies = 2