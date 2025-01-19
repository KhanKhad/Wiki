
# Кладезь знаний

1.  [Тесты для VPS](#vps_test)
2.  [Устранения утечки WebRTC](#webrtc)
3.  [Решение проблемы с сайтами под Cloudflare](#cloudflare)
4.  [Конфиг для резервирования между двумя vps на openwrt в sing-box](#openwrt_singbox_rezerv)
5.  [Точечная маршрутизация основанная на fakeip Sing-box](#openwrt_singbox_fakeip)
6.  [Добавление своих доменов для getdomains](#getdomains_add_domains)
7.  [Блокировка рекламы с Adguard Home](#adguard_home)
8.  [Отключение безопасного DNS](#disable_dns)
9.  [Добавление еще одного ipset для getdomains](#getdomains_ipset_add)
10. [Реализация sing-box через TProxy](#singbox_tproxy)
11. [Переключение dns сервера singbox/xray на stubby, в случае недоступности](#switch_dns)
12. [Заворачиваем трафик роутера в TProxy](#tproxy_wrap)
13. [Отправляем весь трафик с конкретного устройства в туннель](#send_specific_device)
14. [Конвертер URL VLESS+Reality в Outbound sing-box для тех, кто не может составить конфиг ручками](#singbox_outbound_converter)
15. [Настройка NFC-модуля на Xiaomi AX3000T](#nfc_ax3000t)
16. [Использование Raspberry Pi с sing-box в качестве шлюза для устройств локальной сети](#raspberry_gateway)
17. [Отключаем IPv6 в OpenWRT](#disable_ipv6)
18. [Чиним сломанный dnsmasq в snapshot](#broken_dnsmasq)
19. [Альтернативный список заблокированных сайтов](#alternative_domains_list)
20. [Включение Bbr на сервере](#bbr_enable)
21. [Отключение ECH в локальной сети с роутером OpenWRT](#ech_disable)
22. [Конфиг sing-box, чтоб получить резервирование, русский ютуб](#singbox_backup)
23. [Настройка пакета FRP для получения доступа к вебинтерфейсу роутера](#frp_settings)
24. [Установка XRAY+VLESS+TCP+REALITY+CADDY со своим доменом](#xray_vless_caddy)
25. [Скрипт для определения доменов под zapret/туннель](#zapret_or_proxy)
26. [SSH туннель VPS — OpenWRT с использованием Dropbear](#dropbear)
27. [Подключение к UART роутера Xiaomi ax3000t координатора zigbee](#ax3000t_zigbee_uart)
28. [Управление доменами в Sing-box с использованием FakeIP и TProxy на роутерe](#singbox_fakeip_tproxy)
29. [Бот для генерации конфигов WARP под Wireguard и AmneziaWG](#warp_wg_awg_bot)
30. [Организация удаленного доступа к web - интерфейсу роутера на Openwrt с серым ip за NAT с помощью пакета Zerotier](#openwrt_zerotier)
31. [Решение проблемы воспроизведения видео при использовании ReVanced](#revanced_quic)
32. [Получение сертификатов LetEncrypt через DNS запись, используя Certbot](#certbot_dnsapi)
33. [Настройка сервера OCserv с камуфляжем для клиентов openconnect](#ocserv_settings)
34. [Добавление репозитория ImmortalWRT в OpenWRT](#immortalwrt_repo)
35. [Быстрый и грубый способ прибить quic на роутере](#quic_fast_disable)
36. [3X-UI вход по SSH](#3xui_ssh)
37. [AmneziaWG для роутера Gl.iNet MT3000](#awg_mt3000)
38. [Vless URL Builder](#vless_url_builder)
39. [FriendlyElec NanoPi R3S - установка Podkop на FriendlyWRT 23.05](#r3s_friendlywrt_podkop)
40. [Доступ в локальную сеть роутера при помощи Xray reverse proxy](#xray_reverse_proxy)
41. [Настройка IPTV на openwrt](#iptv_openwrt)
42. [Минимальный вывод и комиссия для пользователей Bybit](#bybit)
43. [Валидатор XRAY конфига](#xray_validator)
44. [Скрипт автоматизированной настройки 3x-ui](#3xui_auto_reverse)

***


## Тесты для VPS <a name="vps_test"></a>
*Автор:* Konstantin Benko

*Ссылка:* <https://t.me/itdogchat/44512/59105> 

**Проверка IP сервера на блокировки зарубежными сервисами:**
```bash
bash <(curl -Ls IP.Check.Place) -l en
```
**Параметры сервера и проверка скорости к российским провайдерам:**
```bash 
wget -qO- speedtest.artydev.ru | bash
```
**Параметры сервера и проверка скорости к зарубежным провайдерам:**
```bash 
wget -qO- bench.sh | bash
```
**Проверка блокировки аудио в Instagram:**
```bash 
bash <(curl -L -s https://bench.openode.xyz/checker_inst.sh)
```

## Устранения утечки WebRTC: <a name="webrtc"></a>

*Автор:* Oleg K

*Ссылка:* <https://t.me/itdogchat/44512/66296>

Может кому будет полезно, блокировка stun протокола с помощью самого роутера 

```bash
nft insert rule inet fw4 forward_lan meta l4proto udp @th,96,32 0x2112a442 counter drop
```

Дополню. При старте сети или рестарте файервола правила не будет.
Следующие действия 
```bash 
nano /etc/custom_rules.nft 
meta l4proto udp @th,96,32 0x2112a442 counter drop

nano /etc/config/firewall
config include
        option type 'nftables'
        option path '/etc/custom_rules.nft'
        option position 'chain-pre'
        option chain 'forward_lan'
```

Есть также более сложный вариант можно маркировать stun протокол и отправлять в таблицу маршрутизации.
```bash 
ifname "br-lan" meta l4proto udp @th,96,32 0x2112a442 counter mark set 0x1

        option position 'chain-post'
        option chain 'mangle_prerouting'
```
И рулить через sing-box отправлять в прокси или блокировать

## Решение проблемы с сайтами под Cloudflare: <a name="cloudflare"></a>
*Автор:* Nikita Skryabin

*Ссылка:* <https://t.me/itdogchat/44512/68478>

Написал костыль для решения проблемы с сайтами под Cloudflare (когда незаблокированный домен начинает ходить через туннель из-за заблокированного домена).

Кратко: выкачивается список подсетей Cloudflare, конвертируется в список префиксов из определённого числа октетов (первые 2 или 3 комбинации цифр из четырёх) и уже каждый IP из vpn_domains сравнивается с каждым префиксом. Если совпадает - то IP удаляется.

Ниже дам два варианта скрипта: первый будет сравнивать по 2 октетам, а второй по 3, поэтому выбирайте сами.

Сравнение по 2 октетам:
```bash
#!/usr/bin/env sh

cloudflare_subnets=$(curl -s https://raw.githubusercontent.com/ircfspace/cf-ip-ranges/refs/heads/main/export.ipv4)
cloudflare_prefixes=$(echo "$cloudflare_subnets" | cut -d'/' -f1 | cut -d'.' -f1,2 | sort -u)
existing_ips=$(nft list set inet fw4 vpn_domains | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+')

for ip in $existing_ips; do
  ip_prefix=$(echo "$ip" | cut -d'.' -f1,2)
  if echo "$cloudflare_prefixes" | grep -q "^$ip_prefix$"; then
    echo "[$(date)]: Cloudflare IP $ip was detected and removed" >>/tmp/delcf.log
    nft delete element inet fw4 vpn_domains { $ip }
  fi
done
```

Плюсы: быстро работает, почти не нагружает процессор
Минусы: могут быть ложные срабатывания на IP не из подсети CF

Сравнение по 3 октетам:
```bash
#!/usr/bin/env sh

cloudflare_subnets=$(curl -s https://raw.githubusercontent.com/ircfspace/cf-ip-ranges/refs/heads/main/export.ipv4)
cloudflare_prefixes=$(echo "$cloudflare_subnets" | cut -d'/' -f1 | cut -d'.' -f1,2,3 | sort -u)
existing_ips=$(nft list set inet fw4 vpn_domains | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+')

for ip in $existing_ips; do
  ip_prefix=$(echo "$ip" | cut -d'.' -f1,2,3)
  if echo "$cloudflare_prefixes" | grep -q "^$ip_prefix$"; then
    echo "[$(date)]: Cloudflare IP $ip was detected and removed" >>/tmp/delcf.log
    nft delete element inet fw4 vpn_domains { $ip }
  fi
done
```
Плюсы: уменьшает шанс ложных срабатываний
Минусы: работает медленнее, выше нагрузка на процессор

Сохраняем куда-нибудь, например в /root/delcf.sh, делаем исполняемым chmod +x /root/delcf.sh

Прописываем запуск в crontab командой crontab -e:

```bash
# Тут я указал запуск через каждые 6 часов, но подстройте под себя
0 */6 * * * /root/delcf.sh
```
При работе скрипт логгирует какие именно айпишники он нашёл и удалил в файл по пути /tmp/delcf.log, пример лога:
```
[Sun Sep 29 06:00:02 MSK 2024]: Cloudflare IP 172.64.146.215 was detected and removed
[Sun Sep 29 06:00:03 MSK 2024]: Cloudflare IP 188.114.98.229 was detected and removed
[Sun Sep 29 06:00:03 MSK 2024]: Cloudflare IP 188.114.99.229 was detected and removed
```

## Конфиг для резервирования между двумя vps на openwrt в sing-box:<a name="openwrt_singbox_rezerv"></a>
*Автор:* PavelDBA

*Ссылка:* <https://t.me/itdogchat/44512/69081>

Сюда можно просто подставить свои параметры и заработает. Кому-то это сэкономит время.
```json 
{
    "log": {
      "level": "debug"
    },
    "inbounds": [
      {
        "type": "tun",
        "interface_name": "tun0",
        "domain_strategy": "ipv4_only",
        "inet4_address": "172.16.250.1/30",
        "auto_route": false,
        "strict_route": false,
        "sniff": true
     }
    ],

      "outbounds": [
  {
          "tag": "vpn",
          "type": "urltest",
          "outbounds": [
            "vpn1" ,
            "vpn2"
          ],
      "interval": "10m",
      "interrupt_exist_connections": false
    },
    {"tag": "vpn1",
         "flow": "xtls-rprx-vision",
         "multiplex": {
            "brutal": {
               "down_mbps": 100,
               "enabled": false,
               "up_mbps": 1000
            },
            "enabled": false,
            "max_streams": 32,
            "padding": true,
            "protocol": "h2mux"
         },
         "packet_encoding": "xudp",
         "server": "мой_ip_address1",
         "server_port": 443,
         "tls": {
            "enabled": true,
            "reality": {
               "enabled": true,
               "public_key": "мой_public_key1",
               "short_id": "мой_short_id1"
            },
            "server_name": "www.microsoft.com",
            "utls": {
               "enabled": true,
               "fingerprint": "chrome"
            }
         },
         "type": "vless",
         "uuid": "мой_uuid1"
      },
    {"tag": "vpn2",
         "flow": "xtls-rprx-vision",
         "multiplex": {
            "brutal": {
               "down_mbps": 100,
               "enabled": false,
               "up_mbps": 1000
            },
            "enabled": false,
            "max_streams": 32,
            "padding": true,
            "protocol": "h2mux"
         },
         "packet_encoding": "xudp",
         "server": "мой_ip_address2",
         "server_port": 443,
         "tls": {
            "enabled": true,
            "reality": {
               "enabled": true,
               "public_key": "мой_public_key2",
               "short_id": "мой_short_id2"
            },
            "server_name": "www.microsoft.com",
            "utls": {
               "enabled": true,
               "fingerprint": "chrome"
            }
         },
         "type": "vless",
         "uuid": "мой_uuid2"
      }
      ],
    "route": {
      "auto_detect_interface": true
    }
  }
```

## Точечная маршрутизация основанная на fakeip Sing-box:<a name="openwrt_singbox_fakeip"></a>
*Автор:* Андрей Петелин

*Ссылка:* <https://t.me/itdogchat/44512/71399>

Для тех, кто не хочет точечную маршрутизацию реализованную на уровне dnsmasq-full, вот пример реализации, основанный на fakeip sing-box

Дано: роутер с openwrt 23 на дефолтных настройках

1) Устанавливаем sing-box и отдаем ему конфиг, со своими настройками vless (либо своим outbound другого типа). Конфиг включает в себя проксирование доменов youtube (а также <ifconfig.me> для проверки работоспособности маршрутизации)
```json 
{
    "log": {
        "level": "info",
        "timestamp": true,
        "disabled": false
    },
    "dns": {
        "servers": [
            {
                "tag": "cloudflare-doh-server",
                "address": "https://1.1.1.1/dns-query",
                "detour": "direct-out"
            },
            {
                "tag": "google-doh-server",
                "address": "https://8.8.8.8/dns.query",
                "detour": "direct-out"
            },
            {
                "tag": "fakeip-server",
                "address": "fakeip"
            }
        ],
        "fakeip": {
            "enabled": true,
            "inet4_range": "198.18.0.0/15",
            "inet6_range": "fc00::/18"
        },
        "rules": [
            {
                "query_type": [
                    "A",
                    "AAAA"
                ],
                "domain_suffix": [
                    "ifconfig.me"
                ],
                "rule_set": [
                    "youtube"
                ],
                "server": "fakeip-server"
            }
        ]
    },
    "inbounds": [
        {
            "type": "tun",
            "tag": "tun-in",
            "interface_name": "tun0",
            "inet4_address": "10.0.0.1/30",
            "mtu": 9000,
            "auto_route": false,
            "strict_route": false,
            "sniff": true,
            "sniff_override_destination": false
        },
        {
            "type": "direct",
            "tag": "dns-in",
            "listen": "127.0.0.1",
            "listen_port": 5353,
            "sniff": true
        }
    ],
    "outbounds": [
        {
            "type": "direct",
            "tag": "direct-out"
        },
        {
            "type": "dns",
            "tag": "dns-out"
        },
        {
            "type": "vless",
            "tag": "vless-out",
            "server": "xxxxxxxxxxxxxxxxx",
            "server_port": 443,
            "uuid": "xxxxxxxxxxxxxxxxx",
            "flow": "xtls-rprx-vision",
            "network": "tcp",
            "tls": {
                "enabled": true,
                "server_name": "xxxxxxxxxxxxxxxxx",
                "utls": {
                    "enabled": true,
                    "fingerprint": "xxxxxxxxxxxxxxxxx"
                },
                "reality": {
                    "enabled": true,
                    "public_key": "xxxxxxxxxxxxxxxxx",
                    "short_id": "xxxxxxxxxxxxxxxxx"
                }
            },
            "multiplex": {},
            "transport": {}
        }
    ],
    "route": {
        "rule_set": [
            {
                "type": "remote",
                "tag": "youtube",
                "format": "binary",
                "url": "https://github.com/MetaCubeX/meta-rules-dat/raw/sing/geo/geosite/youtube.srs",
                "download_detour": "direct-out"
            }
        ],
        "rules": [
            {
                "protocol": [
                    "dns"
                ],
                "outbound": "dns-out"
            },
            {
                "inbound": [
                    "tun-in"
                ],
                "clash_mode": "direct",
                "outbound": "direct-out"
            },
            {
                "inbound": [
                    "tun-in"
                ],
                "clash_mode": "global",
                "outbound": "vless-out"
            }
        ],
        "auto_detect_interface": true
    },
    "experimental": {
        "clash_api": {
            "external_ui": "ui",
            "external_controller": "192.168.1.1:9090",
            "external_ui_download_detour": "direct-out",
            "default_mode": "global"
        }
    }
}
```
2) Конфигурируем dnsmasq на использование sing-box в качестве dns сервера
```bash 
nano /etc/config/dhcp

config dnsmasq
  ...
  # Sing-box DNS
  list server '127.0.0.1#5353'
```
3) Отключаем dns сервера получаемые от провайдера (делал через luci на всякий случай, не знаю, что для dnsmasq в приоритете)
4) В /etc/hotplug.d/iface/30-vpnroute делаем маршрут, заворачивающий всю подсеть fakeip в tun0
```bash 
#!/bin/sh

sleep 5
ip route add 198.18.0.0/15 dev tun0
```

P.S. В clash_api, доступном по адресу 192.168.1.1:9090 реализована возможность переключения режима проксирования. По-умолчанию Mode установлен в Global, чтобы весь трафик, попадающий на tun, уходил через VPS. Если по какой-то причине вам необходимо это отключить, можно переключить Mode в Direct, тогда весь трафик, который fakeip заворачивает в outbound, пойдет через провайдера. (возможно, придется убить соединения в разделе Conns, пока я не нашел способа автоматически грохать их при переключении режима)

P.S.S Автозапуск и запуск от root sing-box'a, а также конфигурирование фаервола для tun целенаправленно опустил, все это можно глянуть в статье ITDog (<https://habr.com/ru/articles/767458/>)

P.S.S.S Выражаю благодарность ITDog за статью о настройке sing-box на openwrt, а также @vanes032 за советы касаемые конфигурирования sing-box


## Добавление своих доменов для getdomains: <a name="getdomains_add_domains"></a>
*Автор:* Nikita Skryabin

*Ссылка:* <https://t.me/itdogchat/44512/71563>

[Раздел](https://itdog.info/tochechnaya-marshrutizaciya-po-domenam-na-routere-s-openwrt/#%D1%81%D0%B2%D0%BE%D0%B8-%D0%B4%D0%BE%D0%BC%D0%B5%D0%BD%D1%8B) в статье

Открываем конфиг /etc/config/dhcp своим любимым текстовым редактором и добавляем в конец:
```bash
config ipset
        list name 'vpn_domains'
```
Это основная конструкция, чтобы добавить свои домены, добавляем следующие строчки, где example.com - нужный вам домен:
```bash
config ipset
        list name 'vpn_domains'
        list domain 'example.com'
```
Каждый необходимый домен добавляем новой строкой. Dnsmasq работает по wildcard, поэтому субдомены добавлять не нужно.

Для применения этой настройки нужно рестартануть dnsmasq:
```bash
service dnsmasq restart
```
Также, если требуется удалить ранее добавленный домен, который уже был отрезовлвен и его IP был в добавлен в vpn_domains, то нужно либо перезапустить firewall, либо очистить vpn_domains:
```bash
service firewall restart
# Или
nft flush set inet fw4 vpn_domains
```
Если после этого домен не хочет ходить через туннель, то зафорсите получение его IP через роутер с помощью nslookup или dig, чтобы добавить его в set vpn_domains (замените IP вашего роутера, если он отличается). Это нужно делать с любого клиента, но не с самого роутера:
```bash
nslookup example.com 192.168.1.1
# Или
dig example.com @192.168.1.1
```
Для всего вышеописанного (кроме ручного резолвинга) можно использовать мой [скрипт](https://github.com/vernette/domainctl), который автоматически пропишет в /etc/config/dhcp конфигурацию, а также позволяет добавлять домен (один/список по ссылке/список из файла), удалять домен, получать список добавленных доменов, экспортировать этот список в файл, а также перезапускать dnsmasq и очищать vpn_domains.

## Блокировка рекламы с Adguard Home:<a name="adguard_home"></a>
*Автор:* Kirill

*Ссылка:* <https://t.me/itdogchat/44512/72056> 

<https://telegra.ph/Nastrojka-AdGuard-Home-na-routere-s-OpenWrt-i-tochechnoj-marshrutizaciej-sing-box-09-03>

## Отключение безопасного DNS:<a name="disable_dns"></a>
*Автор:* Nikita Skryabin

*Ссылка:* <https://t.me/itdogchat/44512/75039>

На десктопном Chrome, macOS, iOS: <https://adguard-dns.io/kb/ru/private-dns/solving-problems/known-issues/>

На Android: <https://support.google.com/pixelphone/answer/2819583?hl=ru#zippy=%2C%D0%BF%D0%B5%D1%80%D1%81%D0%BE%D0%BD%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B9-dns-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80>

На десктопном Firefox: поставить "Отключено" <https://support.mozilla.org/ru/kb/nastrojka-urovnej-zashity-dns-cherez-https-v-firef#w_nastroika-doh-zashchity>

В новых версиях Windows 11 тоже появился DoH для подключения по кабелю и WiFi. Ниже шаги для отключения, если вдруг он у вас включен.

**Для Ethernet подключения:**

*Параметры > Сеть и Интернет > Ethernet > Назначение DNS-сервера > Изменить > Автоматически*

**Для Wifi:**

*Параметры > Сеть и Интернет > Wi-Fi > Свойства [название_беспроводной_сети] > Назначение DNS-сервера > Изменить > Автоматически*

## Добавление еще одного ipset для getdomains:<a name="getdomains_ipset_add"></a>
*Автор:* ITDog

*Ссылка:* <https://t.me/itdogchat/44512/77320>

Пример для социальных сетей с картинками и теперь голосовухами

в /etc/config/firewall 
```bash
config ipset
  option name 'vpn_subnet'
  option match 'dst_net'
  list entry '31.13.24.0/21'
  list entry '31.13.64.0/18'
  list entry '45.64.40.0/22'
  list entry '57.141.0.0/24'
  list entry '57.141.3.0/24'
  list entry '57.141.5.0/24'
  list entry '57.141.7.0/24'
  list entry '57.141.8.0/24'
  list entry '57.141.10.0/24'
  list entry '57.141.13.0/24'
  list entry '57.144.0.0/14'
  list entry '66.220.144.0/20'
  list entry '69.63.176.0/20'
  list entry '69.171.224.0/19'
  list entry '74.119.76.0/22'
  list entry '102.132.96.0/20'
  list entry '103.4.96.0/22'
  list entry '129.134.0.0/17'
  list entry '157.240.0.0/17'
  list entry '157.240.192.0/18'
  list entry '163.70.128.0/17'
  list entry '173.252.64.0/18'
  list entry '179.60.192.0/22'
  list entry '185.60.216.0/22'
  list entry '185.89.216.0/22'
  list entry '204.15.20.0/22'

config rule
  option name 'mark_subnet'
  option src 'lan'
  option dest '*'
  option proto 'all'
  option ipset 'vpn_subnet'
  option set_mark '0x1'
  option target 'MARK'
  option family 'ipv4'
```
Для голосовых discord докинуть:
```bash
  list entry '138.128.136.0/21'
  list entry '162.158.0.0/15'
  list entry '172.64.0.0/13'
  list entry '34.0.0.0/15'
  list entry '34.2.0.0/16'
  list entry '34.3.0.0/23'
  list entry '34.3.2.0/24'
  list entry '35.192.0.0/12'
  list entry '35.208.0.0/12'
  list entry '35.224.0.0/12'
  list entry '35.240.0.0/13'
  list entry '5.200.14.128/25'
  list entry '66.22.192.0/18'
```
После этого **service firewall restart**

Список для Discord взят отсюда:
<https://iplist.opencck.org/?format=text&data=cidr4&site=discord.gg&site=discord.media>
<https://t.me/itdogchat/40264/76941>

## Реализация sing-box через TProxy:<a name="singbox_tproxy"></a>
*Автор:* Nikita Skryabin

*Ссылка:* <https://t.me/itdogchat/44512/81764> 

[Настройка TProxy в sing-box.pdf](https://t.me/itdogchat/44512/81764?single)
Отдельная благодарность @fiyerot, @ampetelin и @aquiloa!

config-example.json
```json
{
  "log": {
    "level": "info",
    "timestamp": true
  },
  "dns": {
    "servers": [
      {
        "tag": "cloudflare-doh-server",
        "address": "https://1.1.1.1/dns-query",
        "detour": "direct-out"
      },
      {
        "tag": "google-doh-server",
        "address": "https://8.8.8.8/dns.query",
        "detour": "direct-out"
      }
    ]
  },
  "inbounds": [
    {
      "tag": "tproxy-in",
      "type": "tproxy",
      "listen": "127.0.0.1",
      "listen_port": 4444,
      "tcp_fast_open": true,
      "udp_fragment": true,
      "sniff": true,
      "domain_strategy": "prefer_ipv4"
    }
  ],
  "outbounds": [
    {
      "tag": "direct-out",
      "type": "direct"
    },
    {
      "tag": "dns-out",
      "type": "dns"
    },
    {
      "tag": "wireguard-out",
      "type": "direct",
      "bind_interface": "wg0"
    }
  ],
  "route": {
    "rules": [
      {
        "protocol": ["dns"],
        "outbound": "dns-out"
      },
      {
        "inbound": ["tproxy-in"],
        "outbound": "wireguard-out",
        "domain_suffix": ["ifconfig.me", "fast.com", "nflxvideo.net"]
      }
    ],
    "auto_detect_interface": true
  }
}
```

## Переключение dns сервера singbox/xray на stubby, в случае недоступности<a name="switch_dns"></a>
*Автор:* Oleg K

*Ссылка:* <https://t.me/itdogchat/44512/81766>

Кто использует в качестве dns сервера singbox/xray, накидал скрипт который будет переключать на stubby в случаи недоступности 
```bash
#!/bin/sh

LOG_TAG="DNS_CHECK"
LOG_FILE="/tmp/dnscheck.log"

log() {
    logger -t "$LOG_TAG" "$1"
    echo "$(date) - $1" >> $LOG_FILE
}

# Текущий DNS из конфигурации dnsmasq
CURRENT_DNS=$(grep "list server" /etc/config/dhcp | awk '{print $3}')

SINGBOX_DNS="127.0.0.1:5353"
STUBBY_DNS="127.0.0.1#5453"
TEST_DOMAIN="www.google.com"

check_dns() {
    nslookup $TEST_DOMAIN $1 > /dev/null 2>&1
    return $?
}

log "Current DNS: $CURRENT_DNS"
log "Checking DNS $SINGBOX_DNS..."

# Если текущий DNS равен sing-box, проверяем его доступность
if [ "$CURRENT_DNS" = "'127.0.0.1#5353'" ]; then
    if check_dns "$SINGBOX_DNS"; then
        :
        #log "Resolver from singbox healthy"
    else
        # Если не удается разрешить домен, переключаем на stubby
        uci -q delete dhcp.@dnsmasq[0].server
        uci add_list dhcp.@dnsmasq[0].server="$STUBBY_DNS"
        uci commit && reload_config
        log "Resolver from singbox failed. Switched to stubby."
    fi
else
    # Если текущий DNS не равен sing-box, проверяем его и при успешной проверке переключаем обратно
    if check_dns "$SINGBOX_DNS"; then
        uci -q delete dhcp.@dnsmasq[0].server
        uci add_list dhcp.@dnsmasq[0].server="127.0.0.1#5353"
        uci commit && reload_config
        log "Resolver from singbox healthy. Switched back to sing-box."
    else
        :
        #log "Singbox resolver failed. No changes made."
    fi
fi
```

## Заворачиваем трафик роутера в TProxy<a name="tproxy_wrap"></a>
*Автор:* Sergey Popov

*Ссылка:* <https://t.me/itdogchat/44512/82897>

Всю необходимую информацию по настройке TProxy можно найти [здесь](https://t.me/itdogchat/44512/81763).

Надежнее всего (<https://xtls.github.io/ru/document/level-2/iptables_gid.html>) фильтровать трафик xray или sing-box на основе GID правил.
Для этого создаем специального пользователя tproxy
```bash
grep -qw tproxy /etc/passwd || echo "tproxy:x:0:23333:::" >> /etc/passwd
```

Если используете xray, то в файле /etc/init.d/xray в функцию start_service() добавьте строчку
```bash
procd_set_param user tproxy
```

В случае с sing-box добавьте в /etc/config/sing-box строчку
```bash
option user 'tproxy'
```
Правила nftables будут выглядеть следующим образом
```bash
chain tproxy_prerouting {
    type filter hook prerouting priority mangle; policy accept;

    meta l4proto tcp socket transparent 1 mark set 1 accept
    meta l4proto tcp socket transparent 0 socket wildcard 0 return

    ip daddr { 0.0.0.0/8, 10.0.0.0/8, 127.0.0.0/8, 169.254.0.0/16, 172.16.0.0/12, 192.168.0.0/16, 224.0.0.0/3 } return
    ip6 daddr != 2000::/3 return

    meta l4proto { tcp, udp } tproxy to :$TPROXY_PORT meta mark set 1 accept
}

chain tproxy_output {
    type route hook output priority mangle; policy accept;

    meta skgid 23333 return

    ip daddr { 0.0.0.0/8, 10.0.0.0/8, 127.0.0.0/8, 169.254.0.0/16, 172.16.0.0/12, 192.168.0.0/16, 224.0.0.0/3 } return
    ip6 daddr != 2000::/3 return

    meta l4proto { tcp, udp } meta mark set 1 accept
}
```

## Отправляем весь трафик с конкретного устройства в туннель:<a name="send_specific_device"></a>
*Автор:* Святослав

*Ссылка:* <https://t.me/itdogchat/44512/85008>

Ответы на все эти вопросы есть в конце [статьи](https://itdog.info/podnimaem-na-openwrt-klient-proksi-vless-shadowsocks-shadowsocks2022-nastrojka-sing-box-i-tun2socks/) по настройке sing-box. Если речь про конкретное устройство, сначала нужно задать ему статичный IP. Это можно сделать в luci на главной странице (status/overview) в разделе Active DHCP Leases - просто нажать кнопку Set static у нужного устройства. Далее открываете конфиг фаерволла:
```bash
nano /etc/config/firewall
```
В конце статьи есть 5 примеров правил маркировки - копируете нужное, меняете IP на нужный вам и добавляете в конец этого конфига. Сохраняете изменения и перезапускаете фаерволл:
```bash
service firewall restart
```

## Конвертер URL VLESS+Reality в Outbound sing-box для тех, кто не может составить конфиг ручками:<a name="singbox_outbound_converter"></a>
*Автор:* Андрей Петелин

*Ссылка:* <https://t.me/itdogchat/44512/87398>

<https://singboxconverter.amphub.cloudns.be/>

HTML+JS, всё на стороне клиента

## Настройка NFC-модуля на Xiaomi AX3000T:<a name="nfc_ax3000t"></a>
*Автор:* Dmitry 

*Ссылка:* <https://t.me/itdogchat/44512/87725> 

Отвечу сам себе на первый вопрос. Для записи в nfc берем отсюда [архив](https://github.com/Caian/xinfc/releases/download/v2024-01-29/xinfc_2024-01-29_openwrt_ax3000t.zip) папочку закидываем в root, дальше идем в папочку и делаем команду такого вида:
```bash
./xinfc-wsc 0 0x57 my_ssid_name my_wifi_password sae-mixed 
```
Последний параметр - это шифрование, на примере mixed wpa2/wp3 psk sae. Это прописывается в модуль nfc роутера. Если смените пароль или метод шифрования прописать заново придется. На андроид телефонах работает. З.Ы. Еще наверное понадобится i2c-tools поставить

## Использование Raspberry Pi с sing-box в качестве шлюза для устройств локальной сети:<a name="raspberry_gateway"></a>
*Автор:* Андрей Петелин 

*Ссылка:* <https://t.me/itdogchat/44512/94077>

Изложил свой опыт использования Raspberry Pi с sing-box в качестве шлюза для устройств локальной сети в небольшую шпаргалку по настройке подобного

ОС: Ubuntu
sing-box inbound: TProxy

<https://github.com/ampetelin/sing-box-examples/blob/master/ubuntu-tproxy-gateway.md>

## Отключаем IPv6 в OpenWRT<a name="disable_ipv6"></a>
*Автор:* voper

*Ссылка:* <https://t.me/itdoghat/44512/97002>

1.Отключите поддержку IPv6 в локальной сети и от провайдера :
```bash
uci set 'network.lan.ipv6=0'
uci set 'network.wan.ipv6=0'
uci set 'dhcp.lan.dhcpv6=disabled'
```
2.Отключите RA и DHCPv6, чтобы не раздавались IP-адреса IPv6 :
```bash
uci -q delete dhcp.lan.dhcpv6
uci -q delete dhcp.lan.ra
```
3.Отключить делегирование локальной сети:
```bash
uci set network.lan.delegate="0"
```
4.Удалить префикс IPv6 ULA:
```bash
uci -q delete network.globals.ula_prefix
```
5.Отключите odhcpd
```bash
/etc/init.d/odhcpd disable
/etc/init.d/odhcpd stop
```
6.Сохраните изменения и перезапустите сеть:
```bash
uci commit
/etc/init.d/network restart
```
7. Полностью отключить IPv6 в системе:
```bash
sysctl -w net.ipv6.conf.all.disable_ipv6=1
echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
sysctl -w net.ipv6.conf.default.disable_ipv6=1
sysctl -w net.ipv6.conf.lo.disable_ipv6=1
```
8. Заставляем dnsmasq возращать только IPv4 записи
```bash
uci set dhcp.@dnsmasq[0].filter_aaaa='1'
service dnsmasq restart
```
ЭКСТРА:
1. Отключаем IPv6 при сборке OpenWRT под устройство:
в 
```bash
make menuconfig 
1.Global build settings  --->
      *** Package build options ***
  [ ] Enable IPv6 support in packages
2.Base system  --->  <*> busybox  --->
  [*] Customize busybox options
      *** Applets ***
      Networking Utilities  --->
        [ ] Enable IPv6 support (снять)
```
А также
```bash
LuCI --->
Protocols --->
< > luci-proto-ipv6(также снять)
Network --->
< > odhcpd (по желанию тоже можно снять)
```
2. Такое же можно провернуть и с firmware-selector, нужно убрать пакеты odhcp6c и odhcpd-ipv6only

**Дополнение от Triton_Mgn:**
Писали еще что в /etc/rc.local добавить:
```bash```
sysctl -w net.ipv6.conf.all.disable_ipv6=1
echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
sysctl -w net.ipv6.conf.default.disable_ipv6=1
sysctl -w net.ipv6.conf.lo.disable_ipv6=1

**Дополнение от L:**
Еще можно дополнить:
dnsmasq/stubby можно запретить  резолвить в ipv6

Люси (веб-морда роутера) -> Network -> DHCP & DNS -> Filter -> галку Filter IPv6 AAAA records поставить

Save & Apply

## Чиним сломанный dnsmasq в snapshot: <a name="broken_dnsmasq"></a>
*Автор:* Fiyero Tigelaar

*Ссылка:* <https://t.me/itdoghat/44512/101523>

Для тех, кто никак не может установить стабильную OpenWRT 23.05 и пользуется последним снапшотом со сломанным dnsmasq (а также для любителей экспериментов) написал небольшой скрипт, который:

1) устанавливает xray-core и шаблон конфигурации для него
2) конфигурация xray включает в себя dns-сервер с FakeIP
3) настраивает маркировку FakeIP и TProxy маркированного трафика на порт 127.0.0.1:12701, прослушиваемый xray
4) FakeIP выдаются для доменов из списка itdog, ежедневно обновляемых @unidcml в формате .dat для xray
5) параметры вашего vless-подключения нужно вписать в config.json вручную

Пользуйтесь на свой страх и риск (запускать лучше на чистой системе):
```bash
sh <(wget -O - https://raw.githubusercontent.com/FiyeroT/getdomains-snap/main/install.sh)
```
Благодарности: @unidcml, @escanor087, @voper_not_whopper и конечно владельцу этого замечательного чата

Скрипт будет выпилен после появления аналогичного функционала в Podkop

## Альтернативный список заблокированных сайтов: <a name="alternative_domains_list"></a>
*Автор:* Леша

*Ссылка:* <https://t.me/itdoghat/44512/108106>

Альтернативный [список заблокированных сайтов](https://github.com/Akiyamov/singbox-ech-list) на основе [Re:filter](https://github.com/1andrevich/Re-filter-lists) + последующая отборка сайтов с работающим ECH. Это позволяет обходить около половины сайтов без прокси.
GetDoamins/DNSMasq+NFTables:
Меняем скрипт в /etc/init.d/getdomains следующим образом:
```bash
#!/bin/sh /etc/rc.common

START=99

start () {
#    DOMAINS=https://raw.githubusercontent.com/itdoginfo/allow-domains/main/Russia/inside-dnsmasq-nfset.lst
    DOMAINS=https://github.com/Akiyamov/singbox-ech-list/releases/latest/download/domains_noech_dnsmasq.lst
    count=0
    while true; do
        if curl -m 3 github.com; then
            curl -Lf $DOMAINS --output /tmp/dnsmasq.d/domains.lst
            break
        else
            echo "GitHub is not available. Check the internet availability [$count]"
            count=$((count+1))
        fi
    done

    if dnsmasq --conf-file=/tmp/dnsmasq.d/domains.lst --test 2>&1 | grep -q "syntax check OK"; then
        /etc/init.d/dnsmasq restart
    fi
}
```
После этого запускаем с помощью service getdomains start.
Sing-box:
Для sing-box делаются два списка: domains_noech.srs и domains_ech.srs:
```json
{
    "route": {
        "rule_set": [
            {
                "download_detour": "bypass",
                "format": "binary",
                "tag": "no_ech",
                "type": "remote",
                "url": "https://github.com/Akiyamov/singbox-ech-list/releases/latest/download/domains_noech.srs"
            }
        ],
    }
}
{
    "route": {
        "rule_set": [
            {
                "download_detour": "bypass",
                "format": "binary",
                "tag": "ech",
                "type": "remote",
                "url": "https://github.com/Akiyamov/singbox-ech-list/releases/latest/download/domains_ech.srs"
            }
        ],
    }
}
```
После этого можно использовать в полноценной маршрутизации внутри коробки или же fakeip. Для работы с tun.route_address_set не подойдет так как внутри srs нет айпи, позже исправлю.
Xray:
Данная часть сделана @fiyerot
Список: <https://github.com/Akiyamov/singbox-ech-list/releases/latest/download/ech.dat>
Внутри ech.dat два списка: domains_noech и domains_ech. Первый нужно проксировать, второй не надо.
Пример маршрутизации внутри xray (https://t.me/itdogchat/40277/105867):
```json
"routing": {
  "rules": [
    {
      "type": "field",
      "outboundTag": "proxy",
      "domain": [
        "ext:ech.dat:domains_noech"
      ]
    },    
    {
      "type": "field",
      "outboundTag": "freedom",
      "domain": [
        "ext:ech.dat:domains_ech"
      ]
    }
  ]
}
```
Если используется FakeDNS, то требуется изменить конфигурацию `outbound` для `dns`, добавив `"nonIPQuery": "skip"`: (<https://t.me/itdogchat/40268/107919>)
```json
    {
      "tag": "dns-out",
      "protocol": "dns",
      "settings": {
        "nonIPQuery": "skip",
        "address": "1.1.1.1",
        "port": 53
      }
```


## Включение Bbr на сервере:<a name="bbr_enable"></a>
*Автор:* Yarowrath

*Ссылка:* <https://t.me/itdoghat/44512/111442>
```bash
sh -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf' 
sh -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```


## Отключение ECH в локальной сети с роутером OpenWRT: <a name="ech_disable"></a>
*Автор:* Fiyero Tigelaar

*Ссылка:* <https://t.me/itdoghat/44512/112429>

у кого не открываются различные сайты, например:

- sing-box.sagernet.org
- ifconfig.co
- anifilm.net
- rknweb.ru
- school329.spb.ru
- wylsa.com
- snob.ru
- 2x2tv.ru
- openstreetmap.org

Проблема скорее всего в блокировке ECH (а именно зашифрованных tlshello на cloudflare-ech.com) 
Решается так: запустите команду
```bash 
echo 'filter-rr=HTTPS' >> /etc/dnsmasq.conf && service dnsmasq restart
```
сбросьте кэш днс на устройствах, сайты должны заработать
Проверять вот так:
```bash
dig ifconfig.co HTTPS @192.168.1.1  # ip роутера
```
если все отключилось ответ будет без ANSWER SECTION

если нет, то такой:
```bash
;; ANSWER SECTION:
ifconfig.co.    300  IN  HTTPS  1 . alpn="h3,h2" ipv4hint=104.21.54.91,172.67.168.106 ech=AEX+DQBBfwAgACCyKzUDEGt3JAe/zqlUNNjllSpf9u08Btpufu56PxZnCgAEAAEAAQASY2xvdWRmbGFyZS1lY2guY29tAAA= ipv6hint=2606:4700:3030::ac43:a86a,2606:4700:3037::6815:365b
 ```


## Конфиг sing-box, чтоб получить резервирование, русский ютуб: <a name="singbox_backup"></a>
*Автор:* Konstantin

*Ссылка:* <https://t.me/itdoghat/44512/114373>

P.s. Эти конфиги дополняют точечную маршрутизацию от ITDog

Резервирование через sing-box с несколькими VPS
<https://t.me/itdogchat/40259/54487>

Ру ютуб без рекламы с использованием warp (чистый wg) и/или VPS с российским IP (но без блокировок РКН) с резервированием через sing-box
<https://t.me/itdogchat/40259/54488>

Автор: @k_benko


## Настройка пакета FRP для получения доступа к вебинтерфейсу роутера: <a name="frp_settings"></a>
*Автор:* Fiyero Tigelaar

*Ссылка:* <https://t.me/itdoghat/44512/117394>

Инструкция (с улучшениями по безопасности подключения) , как получить доступ к вебинтерфейсу роутера на Openwrt за NAT и серым ip, используя пакет FRP (fast reverse proxy).
<https://t.me/itdoghat/44512/117392?single>


## Установка XRAY+VLESS+TCP+REALITY+CADDY со своим доменом:<a name="xray_vless_caddy"></a>
*Автор:* Леша

*Ссылка:* <https://t.me/itdoghat/44512/131056>

Написал гайд, скрипт и плейбук для установки XRay+VLESS+TCP+Reality+Caddy со своим доменом. Также скрипт и плейбук настроят iptables и включат bbr.
[Гайд](https://gist.github.com/Akiyamov/bf39613c8e38451e9eaa9fad22f4f40a), но скрипт и плейбук лучше, так как выдадут вам данные для входа в подкоп/некобокс, а также outbounds для xray и sing-box. После скрипта предыдущие креды слетят, так что аккуратнее.
Скрипт:
```bash
bash <(wget -qO- https://raw.githubusercontent.com/Akiyamov/xray-vps-setup/refs/heads/main/vps-setup.sh)
```
Плейбук находится в этом [репозитории](https://github.com/Akiyamov/xray-vps-setup), там же скрипт.
Если хотите просто установить, то выполняете:
```bash
ansible-galaxy role install Akiyamov.xray-vps-setup
```
 
и после этого делаете такой плейбук:
```yaml
---
- name: Setup vps
  hosts: specified_hosts 
# Лучше не ставьте all, так как скрипт и плейбук каждый раз генерируют новые данные для хрей, а значит ваш конфиг слетит
  roles:
    - Akiyamov.xray-vps-setup
  vars:
    vless:
      domain: домен_после_idn
```
**Upd.**
Так как редактировать старое сообщение не могу, то придется отвечать так. А то боюсь будут вопросы еще что за ключ он там просит
Добавил в скрипт смену порта + отключение входа по паролю и входа от рута по ssh. Он там сгенерирует нового юзера и пароль к нему + обновит пароль от рута на всякий. Чтоб был ключ на винде в powershell вводим следующую команду:
```bash
ssh-keygen -t ed25519
```
три раза жмем энтер и вводим следующую команду:
```bash
cat C:\Users\$Env:UserName/.ssh/id_ed25519.pub
```
Для юзеров линукса я, думаю, уже не так трудно будет с этим. Обновленный пример ансибла в репозитории будет как закину в гэлэкси.
По вопросам пишите в тупые вопросы или флуд или сам [репозиторий](https://github.com/Akiyamov/xray-vps-setup).

## Скрипт для определения доменов под zapret/туннель: <a name="zapret_or_proxy"></a>
*Автор:* vverty

*Ссылка:* <https://t.me/itdoghat/44512/131161>

На скорую руку написал [скриптик](https://t.me/itdogchat/44512/131161), который пробегается по списку nfset доменов и делает к каждому из них запрос. если таймаут (или страница с блокировкой) -> заблокировано в россии => добавляется в новый список доменов для обфускации. иначе -> вероятно, блок по ip (ну или другая ошибка) => добавить в новый список nfset для туннелирования
Класть в корневую директорию allow-domains. вверху указываются кастомные правила для доменов (если хотим изменить список для домена принудительно)
Таймаут настроил 2 секунды, поэтому обработка всего списка занимает минут 10.
Понятное дело, можно было оптимизировать + добавить асинхронные запросы/в несколько потоков обрабатывать. но сейчас времени особо нет. если кому-то будет полезно, позже доработаю. либо это может сделать кто-то другой))
Ну, и, очевидно, перед запуском надо отключить все методы обхода

## SSH туннель VPS — OpenWRT с использованием Dropbear: <a name="dropbear"></a>
*Автор:* Bender

*Ссылка:* <https://t.me/itdoghat/44512/137387>

Написал небольшую памятку как можно просто прокинуть порты до устройств в локальной сети используя обратный туннель, в том числе и к роутеру. Пока используя Dropbear, позже добавлю вариант с openssh.
<https://vkust.ru/archives/393>

## Подключение к UART роутера Xiaomi ax3000t координатора zigbee: <a name="ax3000t_zigbee_uart"></a>
*Автор:* Dmitry

*Ссылка:* <https://t.me/itdoghat/44512/138993>

По идее должно работать и на других роутерах:
<https://telegra.ph/Ustanovka-zigbee-koordinatora-cc2652p7-v-router-Xiaomi-ax3000t-s-proshivkoj-OpenWRT-11-25>

## Управление доменами в Sing-box с использованием FakeIP и TProxy на роутерe: <a name="singbox_fakeip_tproxy"></a>
*Автор:* Nikita Skryabin

*Ссылка:* <https://t.me/itdoghat/44512/139432>

Описал свою текущую реализацию управления доменами в sing-box с использованием FakeIP и TProxy на роутере с OpenWRT

Бонусом описал про:
- Фикс голосовых чатов в Discord
- Фикс (или отключение) ECH
- Полное проксирование определённого устройства в TProxy
- Мои списки доменов

<https://gist.github.com/vernette/67466961ed5882b3ff21222d1b964929>

Отдельная благодарность @unidcml и @ampetelin за основу всей схемы

## Бот для генерации конфигов WARP под Wireguard и AmneziaWG: <a name="warp_wg_awg_bot"></a>
*Автор:* Андрей Петелин

*Ссылка:* <https://t.me/itdoghat/44512/140388>

Собственно, как и говорил ранее, набросал бота для генерации конфигов WARP под WireGuard и AmneziaWG: 
<https://t.me/@WarpGenerator_bot>

Спасибо @vernettte за реверс реверс-инжиниринг эндпоинтов варпа

Пожелания/предложения - welcome

## Организация удаленного доступа к web - интерфейсу роутера на Openwrt с серым ip за NAT с помощью пакета Zerotier: <a name="openwrt_zerotier"></a>
*Автор:* L

*Ссылка:* <https://t.me/itdoghat/44512/141129>

[Установка ZeroTier на роутер OpenWRT.pdf](https://t.me/itdogchat/44512/141112)

## Решение проблемы воспроизведения видео при использовании ReVanced: <a name="revanced_quic"></a>
*Автор:* Zverev E.V.

*Ссылка:* <https://t.me/itdoghat/44512/144267>

Если кто-то использует на смартфоне приложение ReVanced вместо стандартного YouTube, то после настройки [точечной маршрутизации](https://itdog.info/tochechnaya-marshrutizaciya-po-domenam-na-routere-s-openwrt/), видео на смартфоне очень плохо воспроизводиться. Постоянные подгрузы начиная от Full HD качества. Чтобы это исправить в настройках приложения зайдите в ReVanced Extended -> Разное -> Отключить протокол QUIC


## Получение сертификатов LetEncrypt через DNS запись, используя Certbot: <a name="certbot_dnsapi"></a>
*Автор:* VecH.Pro

*Ссылка:* <https://t.me/itdoghat/44512/146703>

Нет необходимости держать 80 и 443 порт открытыми для автообновления ботом certbot:
```bash
apt install certbot
wget https://github.com/joohoi/acme-dns-certbot-joohoi/raw/master/acme-dns-auth.py
chmod +x acme-dns-auth.py
nano acme-dns-auth.py
```
В первой строке нужно настроить использование Python 3. Для этого добавьте 3 в конец первой строки:
```bash
#!/usr/bin/env python3
. . .
```
Это гарантирует, что скрипт использует последнюю поддерживаемую версию Python 3, а не устаревшую версию Python 2.
```bash
sudo mv acme-dns-auth.py /etc/letsencrypt/
sudo certbot certonly --manual --manual-auth-hook /etc/letsencrypt/acme-dns-auth.py --preferred-challenges dns --debug-challenges -d domain.com -d \*.domain.com
```

Далее он выдаст строку:
```bash
_acme-challenge.domain.com CNAME 859eazaz-5678-1234-xxxx-beeb29be9864.auth.acme-dns.io.
```
Вот эту CNAME запись надо будет внести на DNS сервере и продолжить скрипт (кажется там enter нажать надо).
И всё, он будет автоматом проверять наличие записи и обновлять сертификат.
Это для создания сертификата через проверку в DNS записи домена (нет необходимости держать 80 и 443 порты свободными на сервере.
Далее в надо изменить докер контейнер и пересоздать его.
В докер контейнере 3x-ui прописываете прокидывание папки с сертификатами:
```yaml
# cat docker-compose.yml
services:
  3x-ui:
    image: ghcr.io/mhsanaei/3x-ui:latest
    container_name: 3x-ui
    hostname: domain.com
    volumes:
      - $PWD/db/:/etc/x-ui/
      - /etc/letsencrypt/:/etc/letsencrypt/
    environment:
      XRAY_VMESS_AEAD_FORCED: "false"
    tty: true
    network_mode: host
    restart: unless-stopped
```
Ключевая строка тут вот эта:
```yaml
      - /etc/letsencrypt/:/etc/letsencrypt/:rw \
```
В панели 3x-ui указать путь к public и private key и всё, само подтягивает, внутри /live/ это симлинки на обновленные сертификаты:

*Public Key Path:  /etc/letsencrypt/live/domain.com/cert.pem*
*Private Key Path: /etc/letsencrypt/live/domain.com/privkey.pem*

Включится httpS в панели, сохраняете, перезапускаете панель и теперь она работает по TLS сертификатам httpS

## Настройка сервера OCserv с камуфляжем для клиентов openconnect: <a name="ocserv_settings"></a>
*Автор:* L

*Ссылка:* <https://t.me/itdoghat/44512/150167>

[Инструкция по установке OCSERV_1.docx](https://t.me/itdogchat/44512/150165?single)
[Инструкция создания сертификатов для клиентов ocserv.docx](https://t.me/itdogchat/44512/150166?single)

Во второй статье, как соединяться с сервером по сертификатам; если по каким то причинам логин-пароль не подходит.
Если кто знает как сделать лучше или надежнее, пжта напишите, внесу дополнения.
Еще собираюсь сделать инструкцию по настройке клиентов


## Добавление репозитория ImmortalWRT в OpenWRT: <a name="immortalwrt_repo"></a>
*Автор:* Spyke Anem

*Ссылка:* <https://t.me/itdoghat/44512/152899>

```bash
VERSION_ID=$(grep "VERSION_ID" /etc/os-release | awk -F '"' '{print $2}')
ARCH=$(grep "OPENWRT_ARCH" /etc/os-release | awk -F '"' '{print $2}')
sed -i 's/option check_signature/# option check_signature/g' /etc/opkg.conf
echo "src/gz immortalwrt_routing https://downloads.immortalwrt.org/releases/$VERSION_ID/packages/$ARCH/routing" >> /etc/opkg/customfeeds.conf
echo "src/gz immortalwrt_packages https://downloads.immortalwrt.org/releases/$VERSION_ID/packages/$ARCH/packages" >> /etc/opkg/customfeeds.conf
echo "src/gz immortalwrt_luci https://downloads.immortalwrt.org/releases/$VERSION_ID/packages/$ARCH/luci" >> /etc/opkg/customfeeds.conf
echo "src/gz immortalwrt_base https://downloads.immortalwrt.org/releases/$VERSION_ID/packages/$ARCH/base" >> /etc/opkg/customfeeds.conf
```
Если версия OpenWRT 23.05.5:
```bash
ARCH=$(grep "OPENWRT_ARCH" /etc/os-release | awk -F '"' '{print $2}')
sed -i 's/option check_signature/# option check_signature/g' /etc/opkg.conf
echo "src/gz immortalwrt_routing https://downloads.immortalwrt.org/releases/23.05.4/packages/$ARCH/routing" >> /etc/opkg/customfeeds.conf
echo "src/gz immortalwrt_packages https://downloads.immortalwrt.org/releases/23.05.4/packages/$ARCH/packages" >> /etc/opkg/customfeeds.conf
echo "src/gz immortalwrt_luci https://downloads.immortalwrt.org/releases/23.05.4/packages/$ARCH/luci" >> /etc/opkg/customfeeds.conf
echo "src/gz immortalwrt_base https://downloads.immortalwrt.org/releases/23.05.4/packages/$ARCH/base" >> /etc/opkg/customfeeds.conf
```

Затем:
```bash
opkg update
```
Теперь у вас репо OpenWRT и ImmortalWRT. Кто как этим будет распоряжаться, оставлю на ваш выбор.


## Быстрый и грубый способ прибить quic на роутере: <a name="quic_fast_disable"></a>
*Автор:* voper

*Ссылка:* <https://t.me/itdoghat/44512/158408>

(https://t.me/itdoghat/44512/158408)

## 3X-UI вход по SSH: <a name="3xui_ssh"></a>
*Автор:* Ezeez M

*Ссылка:* <https://t.me/itdoghat/44512/158513>

Если уже было то прошу прощения :)
На панель 3x-UI можно ходить теперь через SSH туннель, появилась опция такая (22) по команде x-ui.
Сделано для того, что-бы не ходить на панель по HTTP. Если у вас нет домена и собственно сертификата, то лучше заходить на панель через SSH туннель. 
```bash
ssh -L PORT1:127.0.0.1:PORT2 USER@SERVER_IP -p PORT3
```
PORT1 - локальный порт для SSH туннеля, например 2222
PORT2 - порт панели который вы установили или порт по умолчанию
PORT3 - порт SSH на сервере. Обычно это 22, но если вы его сменили (а лучше его сменить, что-бы боты не долбили порт), то укажите тот адрес порта который вы установили в */etc/ssh/sshd_config*
SERVER_IP - Публичный IP вашего сервера
USER - пользователь для подключения по SSH, это или root, или тот что вы создавали (если) отдельно.

## AmneziaWG для роутера Gl.iNet MT3000: <a name="awg_mt3000"></a>
*Автор:* Святослав

*Ссылка:* <https://t.me/itdoghat/44512/174547>

Для роутера GL.iNet MT3000 есть пакеты amneziaWG под заводскую прошивку (если правильно понял, под 24 версию заводской прошивки):
<https://t.me/amnezia_vpn/204428/397981>


## Vless URL Builder: <a name="vless_url_builder"></a>
*Автор:* Александр

*Ссылка:* <https://t.me/itdoghat/44512/174817>

Если вдруг кому json амнезии xray перевести в vless ссылку нашёл такую приблуду <https://github.com/indx0/Vless-Url-Builder> вдруг кому пригодится

## FriendlyElec NanoPi R3S - установка Podkop на FriendlyWRT 23.05: <a name="r3s_friendlywrt_podkop"></a>
*Автор:* Fiyero Tigelaar

*Ссылка:* <https://t.me/itdoghat/44512/181082>

Если вы по какой-то причине не можете перейти на OpenWRT 24.10-rc4, а Podkop не работает, нужно выпилить 95 пакетов, связанных с IPTABLES:
```bash
opkg update
opkg remove --force-removal-of-dependent-packages iptables* kmod-ipt* kmod-ip6tables kmod-nf-ipt kmod-nf-ipt6 tc-mod-iptables kmod-br-netfilter
opkg install dnsmasq-full
reboot
```
После этого можно устанавливать Podkop, проблема с прохождением пакетов через tproxy до sing-box уйдёт

Кроме перечисленных в remove пакетов удалятся следующие (возможно, для вас это важно):
```bash
kmod-gre (Generic Routing Encapsulation support)
kmod-ipsec4 (Kernel modules for IPsec support in IPv4)
kmod-pptp (PPtP support)
ppp-mod-pptp (This package contains a PPtP plugin for ppp)
```

## Доступ в локальную сеть роутера при помощи Xray reverse proxy: <a name="xray_reverse_proxy"></a>
*Автор:* Sergey Popov

*Ссылка:* <https://t.me/itdoghat/44512/181836>

Настройка роутера (bridge).
Добавляем секцию reverse со следующим содержимым:
```json
  "reverse": {
    "bridges": [
      {
        "tag": "bridge",
        "domain": "reverse.proxy"
      }
    ]
  }
```
Правила роутинга должны выглядеть так:
```json
  "routing": {
    "rules": [
      {
        "inboundTag": ["bridge"],
        "domain": ["full:reverse.proxy"],
        "outboundTag": "vless-out"
      },
      {
        "inboundTag": ["bridge"],
        "outboundTag": "direct"
      }
    ]
  }
```
Настройка удаленного сервера (portal).
Добавляем секцию reverse со следующим содержимым:
```json
  "reverse": {
    "portals": [
      {
        "tag": "portal",
        "domain": "reverse.proxy"
      }
    ]
  }
```
Правила роутинга должны выглядеть так (пользователя lan раздаем на устройства, которым нужен доступ в локальную сеть):
```json
  "routing": {
    "rules": [
      {
        "inboundTag": ["vless-in"],
        "domain": ["full:reverse.proxy"],
        "outboundTag": "portal"
      },
      {
        "inboundTag": ["vless-in"],
        "user": ["lan"],
        "outboundTag": "portal"
      }
    ]
  }
```
Отдельный респект китайцам за такой хороший нейминг и превосходную документацию...

## Настройка IPTV на openwrt: <a name="iptv_openwrt"></a>
*Автор:* Spyke Anem

*Ссылка:* <https://t.me/itdoghat/44512/186274>

Настройка IPTV на openwrt <https://openwrt.org/docs/guide-user/network/wan/udp_multicast> или коротко ниже:
```bash
opkg install igmpproxy
```
Редактируем /etc/config/igmpproxy
```bash
config igmpproxy
  option quickleave 1

config phyint
  option network wan
  option zone wan
  option direction upstream
  list altnet 0.0.0.0/0

config phyint
  option network lan
  option zone lan
  option direction downstream
```
В /etc/config/network проверяем, если нет, то добавляем:
```bash
config device
  option type 'bridge'
  option igmp_snooping '1'
```
Далее:
```bash
service igmpporxy restart
```
На всякий случай.

Можно для страховки в etc/config/firewall добавить, но должно и так работать:
```bash
config rule
        option src      wan
        option proto    igmp
        option target   ACCEPT
config rule
        option src      wan
        option proto    udp
        option dest_ip  224.0.0.0/4
        option target   ACCEPT
```
Потом: 
```bash
service firewall restart
```


## Минимальный вывод и комиссия для пользователей Bybit: <a name="bybit"></a>
*Автор:* Spyke Anem

*Ссылка:* <https://t.me/itdoghat/44512/186337>

Тем, кто пользуется биржой Bybit и оплачивает хостинги в крипте. Информация по минимальному выводу и комиссиям. 
1) Смотрим через какую сеть принимает хостинг
2) Выбираем минимальную комиссию и минимум
3) Производим вывод средств с кошелька финансирования
Например: хостинг стоит 4 USDT, нам по минимуму доступно TRC20, Arbitrum One и TON. Если хостинг принимает TRC20, то итоговая сумма оплаты будет 4 USDT + 1.6 USDT (комиссия) = 5.6 USDT к оплате. Самое невыгодное платить через сеть ERC20, самое выгодное через TON (если принимают). Если покупка выше 10 USDT то можно через BSC(BEP20), принимают много где.

## Валидатор XRAY конфига: <a name="xray_validator"></a>
*Автор:* regexp

*Ссылка:* <https://t.me/itdoghat/44512/191845>

<https://mmmray.github.io/xray-online>

## Скрипт автоматизированной настройки 3x-ui: <a name="3xui_auto_reverse"></a>
*Автор:* BLACKO

*Ссылка:* <https://t.me/itdoghat/44512/192888>

Включает в себя:

    1.  Конфигурация сервера Xray с 3X-UI:
        - VLESS-TCP-XTLS-Vision и VLESS-TCP-REALITY (Steal oneself).
        - Подключение подписки и JSON подписки для автоматического обновления конфигураций.
    2.  Настройку обратного прокси NGINX на порт 443.
    3.  Обеспечение безопасности:
        - Автоматические обновления системы через unattended-upgrades.
    4.  Настройка SSL сертификатов Cloudflare с автоматическим обновлением для защиты соединений.
    5.  Настройка WARP для защиты трафика.
    6.  Включение BBR — улучшение производительности TCP-соединений.
    7.  Настройка UFW (Uncomplicated Firewall) для управления доступом.
    8.  Настройка SSH, для обеспечения минимально необходимой безопасности.
    9.  Отключение IPv6 для предотвращения возможных уязвимостей.
    10. Шифрование DNS-запросов с использованием systemd-resolved (Dot) или AdGuard Home (Dot, DoH).
    11. Выбор случайного веб-сайта из массива для добавления дополнительного уровня конфиденциальности и сложности для анализа трафика.

<https://github.com/cortez24rus/xui-reverse-proxy/blob/main/README_RU.md>
