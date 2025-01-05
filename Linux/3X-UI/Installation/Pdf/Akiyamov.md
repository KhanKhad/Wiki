В данном гайде будет рассмотрена установка прокси-сервера XRay и Caddy для маскировки под свой сайт со своим доменом. Если вы попали на эту статью не из репозитория скрипта, то вы всегда можете перейти в него по этой [ссылке](https://github.com/Akiyamov/xray-vps-setup).  
Гайд сделан для Ubuntu и Debian.  

## Домен
Для установки потребуется свой домен, лучше покупать не .ru и .рф, так как для них требуется паспорт.  
После покупки домена советую перенести его на Cloudflare ради удобства. Как это сделать можно посмотреть [здесь](https://wiki.bisquit.host/transfer-domain).  
Добавьте A-запись, которая указывает на ваш виртуальный сервер, выключив при этом проксирование от Cloudflare, CDN нам не нужен. Если вы не понимаете как это сделать, то можете посмотреть это [здесь](https://community.cloudflare.com/t/dns/400199).  
Если вы купили домен, который исползьует punycode, например, в зоне .рф, то стоит сразу перевести его в правильный формат. Для этого скачиваем пакет `idn` и выполняем следующую команду:
```bash
echo "домен" | idn
```

## Установка Caddy 
Для начала установим веб-сервер caddy. Чтобы это сделать нужно добавить репозитории caddy  
```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install -y caddy
```

## Установка XRay 
После этого можно установить XRay, это можно сделать официальным скриптом.
```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

## Конфиг Caddy
Конфиг Caddy располагается в `/etc/caddy/Caddyfile`.  
В Caddy нам нужно установить кастомный порт для https и сделать так, чтобы он слушал только на локалхосте, также запретим HTTP/3. Первая секция отвечает за это.  
Второй блок отвечает за то, под что мы маскируемся. Если вы не будете менять конфиг, то будет файл-сервер. Если же вы хотите его поменять, то, скорее всего, вы сами знаете как это сделать.  
Третий блок просто перенаправляет порт 80 на 443, так выглядит естественнее.  
`$VLESS_DOMAIN` нужно заменить на свой домен.
```
{
  https_port 4123
  default_bind 127.0.0.1
  servers {
    protocols h1 h2
    listener_wrappers {
      proxy_protocol {
        allow 127.0.0.1/32
      }
      tls
    }
  }
  auto_https disable_redirects
}
https://$VLESS_DOMAIN {
  root * /srv
  file_server browse
  log {
    output file /var/lib/caddy/access.log {
      roll_size 10mb
      roll_keep 5
  }
}
}
http://$VLESS_DOMAIN {
  bind 0.0.0.0
  redir https://{host}{uri} permanent
}
```

## Конфиг XRay
После установки XRay нам нужно создать данные для конфига. Сначала создадим Public и Private key, для этого введем команду `xray x25519`, ее вывод будет в таком виде:
```
Private key: yLTMMfcsJJ3Fh4t_Pjf777iDhCYx0-PSmksmKBrpRAI
Public key: IGoeCjQm24_wA9M0N3WMVV3ze8NKXb_Yv7CEU_HKMwQ
```
Создадим short id, создается командой `openssl rand -hex 8`, а также uuid для пользователя xray командой `xray uuid`.

`$VLESS_DOMAIN` - домен, который мы купили. Если используется кириллический в зоне .рф, то поставьте пакет `idn` и выполните следующую команду:

Конфиг XRay находится в `/usr/local/etc/xray/config.json`. Нам требуется настроить `inbound` таким образом, чтоб Reality перенаправлял все запросы на localhost:4123 и использовал TLS-сертификат от Caddy. Caddy сам его получает и настраивает, поэтому, нам ничего не надо делать. 
Также добавим для XRay DNS, установим правило в ipv4 и запретим качать торренты.
```json
{
  "log": {
    "loglevel": "none"
  },
  "inbounds": [
    {
      "listen": "0.0.0.0",
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "$XRAY_UUID",
            "email": "default",
            "flow": "xtls-rprx-vision"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "dest": "127.0.0.1:4123",
          "serverNames": [
            "$VLESS_DOMAIN"
          ],
          "privateKey": "$XRAY_PIK",
          "shortIds": [
            "$XRAY_SID"
          ],
          "spiderX": "/"
        }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls",
          "quic"
        ],
        "routeOnly": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "tag": "direct",
      "settings": {
        "domainStrategy": "UseIPv4"
      }
    },
    {
      "protocol": "blackhole",
      "tag": "block"
    }
  ],
  "routing": {
    "rules": [
      {
        "protocol": "bittorrent",
        "outboundTag": "block"
      }
    ],
    "domainStrategy": "IPIfNonMatch"
  },
  "dns": {
    "servers": [
      "1.1.1.1",
      "8.8.8.8"
    ],
    "queryStrategy": "UseIPv4",
    "disableFallback": false,
    "tag": "dns-aux"
  }
}
```
`$XRAY_PIK` - Private key, который мы получали ранее.  
`$VLESS_DOMAIN` - домен, который мы купили.  
`$XRAY_SID` - Short id, создавался ранее.  
`$XRAY_UUID` - UUID для клиента.  

После того, как вы поменяли конфиги, можно перезапускать Caddy и XRay:
```bash
systemctl restart xray
systemctl restart caddy
```

## Донастройка

Сейчас сервер уже может работать, но лучше закрыть лишние порты и закрыть вход по паролю, оставив только ключ и включим bbr.
С помощью iptables закроем порты:
```bash
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
iptables -P INPUT DROP
sudo apt-get install -y iptables-persistent 
```
Loopback внутри сети нам нужен, так как без этого мы запрещаем работу Caddy и XRay.  

Для того, чтобы запретить вход по паролю, добавим public key на сервер.
Если у вас еще нет ключа, то создать его можно с помощью команды `ssh-keygen -t ed25519`. После этого на GNU/Linux и WSL можно передать их на сервер командой `ssh-copy-id -i ~/.ssh/id_ed25519.pub root@айпи`.
На чистом Windows используется следующая команда:
```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh root@айпи "cat >> .ssh/authorized_keys"
```
После переноса ключей в конфиге ssh, который расположен в `/etc/ssh/sshd_config` убираем комментарий со строки: `PasswordAuthentication yes` и меняем ее на `PasswordAuthentication no`
Сохраняем файл и перезагружаем ssh: `systemctl restart sshd`

Включить bbr проще всего, достаточно выполнить следующие команды:
```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

## Используем файлопомойку

Если вы не хотите менять ваш сервис для маскировки на сайт, который вы сделали сами или что-то еще, то почему бы не использовать файл сервер, который мы подняли?
Для этого скачайте [Filezilla](https://filezilla-project.org/) и запустив нажмите Ctrl+S. Добавьте новый сайт, который использует протокол SFTP и в "Тип входа" выберите "Файл с ключом", ниже выберите ваш ключ и укажите пользователя root.
После подключения в правом окне у вас видна система сервера. Если вы настраивали по гайду, то ваш сайт отображает директорию `/srv`, поэтому закидывайте в нее файлы.