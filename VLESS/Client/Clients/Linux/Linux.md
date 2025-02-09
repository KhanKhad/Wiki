Единственный клиент с UI, который у меня заработал под Linux - это Nekoray. https://github.com/MatsuriDayo/nekoray/

Всё так же работает как и под Windows. Но есть одно различие, это запуск от рута для tun режима. В Windows это указывается в свойствах ярлыка или самого exe-шника.

А, например, в Gnome, если приложению нужно что-то запустить\изменить от sudo, выводится окошко с запросом пароля. Тут при активации tun-режима это окно выводится три раза подряд. Это подбешивает 🙂

Изучаем issues на гитхабе проекта: https://github.com/MatsuriDayo/nekoray/issues/759

У человека Ubuntu. Закрыто со статусом “not planned” без объяснений.

https://github.com/MatsuriDayo/nekoray/issues/769

У человека Fedora. Закрыто со статусом “not planned”, но тут разработчик поясняет, что виновата неправильно настроенная ОС.

https://github.com/MatsuriDayo/nekoray/issues/926

У человека Arch. Закрыто со статусом “not planned”. Здесь “виновато криво настроенное ядро в OS”.

Заметьте, все три issues от разных людей, сидящих на трёх разных OS Linux. С каким флагом надо собирать ядро так и не понятно 🙁 Вопрос, достойный скрининга в Яндекс!

Очевидный костыль - это запуск приложения всегда от sudo. Как делается это для классической Ubuntu 22.04:

Ярлыки приложений лежат в /usr/share/applications/, открываем /usr/share/applications/nekoray.desktop и в Exec перед sh ставим sudo. Получаем:

Exec=sudo sh -c "PATH=/opt/nekoray:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin /opt/nekoray/nekoray -appdata"
Ещё можно запускать из консоли:

sudo /opt/nekoray/nekoray -appdata
Теперь будет один запрос пароля, либо с настроенным visudo вообще без него.

Ещё меня напрягает использование beta версий sing-box в релизах. Например, в 3.23 更新 sing-box 1.6.0-beta1.

Из хорошего есть deb пакет. Даже для Windows нет установщика, а для debian-like есть. Также пакет есть в AUR .