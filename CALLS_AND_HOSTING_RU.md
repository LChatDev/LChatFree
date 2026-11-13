# LChat — бесплатный хостинг и звонки

## 1. Выберите схему

| Вариант | Что нужно | Для чего подходит |
|---|---|---|
| Свой ПК | ПК с Docker, интернет, HTTPS-доступ к сайту | Пробный запуск. Сайт зависит от питания ПК и интернета |
| Одна Linux VM | Публичный IPv4, DNS, PHP, MariaDB, coturn | Сайт и звонки работают без домашнего ПК |
| Раздельное размещение | PHP-хостинг для LChatWeb, сервер для LChatHost, доступный TURN | Если сайт уже размещён отдельно |

Бесплатная VM возможна, например, в Oracle Cloud Always Free при наличии квоты и свободной ёмкости. Проверьте условия регистрации, конфигурацию диска и оценку стоимости до создания ресурсов. Бесплатный пробный период и Always Free — разные программы. Неактивные VM могут быть изъяты. [Условия Oracle Always Free](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm).

## 2. Запустите сайт

**На одной VM:** разверните LChatRU с Apache, PHP и MariaDB по [пошаговой инструкции](FREE_HOSTING_RU.md). Настройте DNS для сайта и TURN. Выпустите сертификат для сайта и проверьте его с другого устройства. Для обычной проверки HTTP-01 серверу нужен входящий TCP 80. [Документация Let's Encrypt](https://letsencrypt.org/docs/challenge-types/).

**На своём ПК или с отдельным веб-хостингом:** запустите [LChatHost](https://github.com/LChatDev/LChatHost/releases) и разместите [LChatWeb](https://github.com/LChatDev/LChatWeb) на PHP-хостинге. В панели LChatWeb введите HTTPS-адрес LChatHost и общий секрет из панели хоста. Откройте install.php по адресу сайта, введите ключ установки, зарегистрируйте владельца и проверьте сохранение сообщения после перезагрузки. После установки закройте доступ к установщику и очистите ключ установки. [Панель LChatWeb](https://github.com/LChatDev/LChatWeb/releases/tag/web-panel-v1.0.1) · [Видео установки](https://www.youtube.com/watch?v=iObm-ysGBfA).

Временный HTTP-туннель можно использовать для пробы сайта. Его адрес меняется после перезапуска. Для постоянной установки нужен постоянный HTTPS-адрес. Если домашняя сеть блокирует соединение Cloudflare Tunnel, проверьте исходящий порт 7844 или разместите сайт на VM с публичным адресом. HTTP-туннель обслуживает сайт, TURN передаёт медиатрафик. [Cloudflare Quick Tunnels](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/trycloudflare/) · [Порты Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/configure-tunnels/tunnel-with-firewall/).

## 3. Настройте TURN

1. На Linux VM установите coturn. Сгенерируйте отдельный секрет командой `openssl rand -hex 32`.
2. В /etc/turnserver.conf укажите `listening-port=3478`, `use-auth-secret`, `static-auth-secret=ВАШ_СЕКРЕТ`, `realm=turn.example.com`, `min-port=49160` и `max-port=49200`. Полный пример и запуск службы — в [инструкции coturn для LChat](CALLS_RU.md).
3. Разрешите входящие TCP и UDP 3478 и UDP 49160–49200 в правилах облака **и** брандмауэре ОС. За NAT настройте `external-ip=ПУБЛИЧНЫЙ_IP/ЛОКАЛЬНЫЙ_IP` и отображение relay-портов один к одному. MariaDB оставьте доступной только приложению. [Конфигурация coturn](https://github.com/coturn/coturn/blob/master/examples/etc/turnserver.conf).
4. В админке LChat или приватном config.php на сервере укажите адрес `turn:turn.example.com:3478?transport=udp` и тот же секрет. Если сеть блокирует UDP до TURN, проверьте `?transport=tcp`. UDP-диапазон relay по-прежнему нужен на сервере. Начните с режима `all` и небольшой группы.

## 4. Проверьте звонок

1. Откройте сайт по HTTPS и разрешите микрофон. Возьмите два устройства и два аккаунта, одно подключите к Wi‑Fi, второе к мобильному интернету.
2. Создайте звонок по ссылке. Проверьте звук в обе стороны, затем видео.
3. Временно включите `relay` («Только TURN»), создайте новый звонок и повторите проверку. Если работает, TURN участвует в соединении. Верните `all`, если хотите разрешать прямой маршрут.
4. Если звонок работает только в Wi‑Fi, проверьте NAT и relay-порты. Если в `relay` совсем нет связи, проверьте адрес TURN, совпадение секретов, время на сервере и журнал `sudo journalctl -u coturn -n 80 --no-pager`. Если связи нет ещё до подключения к комнате, сначала проверьте сайт и LChatHost.

Установка на конкретном бесплатном аккаунте и доступность его портов требуют отдельной проверки.
