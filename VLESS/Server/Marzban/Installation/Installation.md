Обновленный гайд, скрипт и плейбук на установку XRay с VLESS + TCP + Reality + Vision с получением сертификатов с помощью Caddy. Теперь еще есть и Marzban как вариант установки. 
Скрипт как и прошлый настраивает BBR, создает нового юзера для логина и запрещает вход от рута и с паролем и добавляет юзеру SSH-ключ, ну и iptables, только теперь вся часть с защитой на усмотрение пользователя. Также добавлена установка WARP с помощью warp-cli и yq. 
Гайд - тык(https://github.com/Akiyamov/xray-vps-setup/blob/main/install_in_docker.md).
Скрипт:
bash <(wget -qO- https://raw.githubusercontent.com/Akiyamov/xray-vps-setup/refs/heads/main/vps-setup.sh)