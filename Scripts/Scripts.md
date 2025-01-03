
# Скрипты

## Тестирование

### Ip

bash <(curl  -sL  IP.Check.Place) l en - тесты Ip адреса, как кем определяется, подозрительный ли и т.д.


### Железо

wget -q0- speedtest.artydev.ru | bash - Тесты I\O, скорости интернета, информация о системе

## Установка

3x-ui: bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)