# Проверка IP сервера на блокировки зарубежными сервисами:
bash <(curl -Ls IP.Check.Place) -l en

# Параметры сервера и проверка скорости к российским провайдерам:
wget -qO- speedtest.artydev.ru | bash

# Параметры сервера и проверка скорости к зарубежным провайдерам:
wget -qO- bench.sh | bash

# Проверка блокировки аудио в Instagram:
bash <(curl -L -s https://bench.openode.xyz/checker_inst.sh)