﻿Для удаленного доступа к роутеру без белого адреса есть варианты с обратным туннелем, например:
ngrok
tailscale
zerotier
Tunnelmole.
Подробный список

Все они работают по похожему принципу - поднимают туннель до сервиса в интернете и выставляют на том адресе порты твоего сервера. дают возможность подключиться по адресу типа https://user@service.name
Но не все эти сервисы подходят для openwrt. 