Скрипты для тестов:

# Проверка IP сервера на блокировки зарубежными сервисами:
bash <(curl -Ls IP.Check.Place) -l en

# Параметры сервера и проверка скорости к российским провайдерам:
wget -qO- speedtest.artydev.ru | bash

# Параметры сервера и проверка скорости к зарубежным провайдерам:
wget -qO- bench.sh | bash

# Проверка скорости проца
sysbench --threads=1 cpu run

# Проверка блокировки аудио в Instagram:
bash <(curl -L -s https://bench.openode.xyz/checker_inst.sh)

# Проверки на регионы
curl -s "https://raw.githubusercontent.com/vernette/ipregion/refs/heads/master/ipregion.sh" | bash

# Проверка ip маршрутов
bgp.tools, bgp.he.net

# Проверка на ддос
Проверяем атакуют ли нас через netstat -ant | grep SYN_RECV или ss -t state SYN-RECV|wc -l, если видим много подключений - атакуют

# Не пробовал но вроде норм
bash <(curl -Ls github.com/Davoyan/ipregion/raw/master/ipregion.sh)

bash <(curl -Ls check.unlock.media) -E en -R 0

bash <(curl -Ls yabs.sh)

curl -4 https://www.cloudflare.com/cdn-cgi/trace/ -x socks5:/127.0.0.1:40000


Тестирование VPS:
https://scarce-hole-1e2.notion.site/45ce01e8f0154f2b86afd3b1d9a5d5e0
Есть копия пдф в соседней папке.