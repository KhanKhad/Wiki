Скрипты для тестов:

# Проверка IP сервера на блокировки зарубежными сервисами:
bash <(curl -Ls IP.Check.Place) -l en

# Параметры сервера и проверка скорости к российским провайдерам:
wget -qO- speedtest.artydev.ru | bash

# Параметры сервера и проверка скорости к зарубежным провайдерам:
wget -qO- bench.sh | bash

# Проверка блокировки аудио в Instagram:
bash <(curl -L -s https://bench.openode.xyz/checker_inst.sh)


Тестирование VPS:
https://scarce-hole-1e2.notion.site/45ce01e8f0154f2b86afd3b1d9a5d5e0
Есть копия пдф в соседней папке.