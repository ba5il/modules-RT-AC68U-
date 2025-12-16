**!!!! ВАЖНО !!!! заходим в веб интерфейс и на вкладке (LAN -- Switch Control) обязательно отключаем NAT Acceleration**

## 1. Устанавливаем необходимые пакеты

`opkg install curl bind-tools git-http ipset iptables xtables-addons_legacy cron coreutils-id coreutils-sort coreutils-sleep gzip tar ncat procps-ng-pgrep procps-ng-sysctl` 

В файле `/opt/etc/init.d/S10cron` вместо ARGS="-s" оставляем просто ARGS="", чтобы не было постоянного флуда в системный лог.

## 2. Скачиваем недостающие модули и переносим их в раздел jffs (для прошивок ниже 386.14_2)

`git clone --depth 1 https://github.com/ba5il/modules-RT-AC68U- /opt/mod_add`

`mv /opt/mod_add/modules /jffs/modules`

`rm -r /opt/mod_add`

## 3. Добавляем подключение необходимых модулей в скрипт при старте роутера

`nano /jffs/scripts/init-start`

Мой файл выглядит так:
```
#!/bin/sh
modprobe xt_NFQUEUE
modprobe ip_set
modprobe ip_set_hash_ip
modprobe ip_set_hash_net
modprobe ip_set_bitmap_ip
modprobe ip_set_list_set
modprobe xt_set
# раскомментировать нижние строки для прошивок ниже 386.14_2
#insmod /jffs/modules/nfnetlink_queue.ko
#insmod /jffs/modules/xt_owner.ko
```
После этого выполняем `/jffs/scripts/init-start`, чтобы все необходимые модули подключились.

## 4. Теперь можно заняться настройкой и запуском основного скрипта

[В релизах](https://github.com/bol-van/zapret/releases/) выбираем нужную версию *.tar.gz

### 4.1) Скачиваем и разархивируем релиз
`cd /opt`
      
`curl -Lo - https://github.com/bol-van/zapret/releases/download/v72.4/zapret-v72.4.tar.gz | tar -xvzf -`
      
`mv /opt/zapret-v72.4 /opt/zapret`
      
### 4.2) Устанавливаем нужные бинарники 
`/opt/zapret/install_bin.sh`

### 4.3) Запускаем поиск возможных стратегий обхода
`/opt/zapret/blockcheck.sh | tee /opt/zapret/blockcheck.txt`

  Весь вывод работы скрипта будет записан в файл blockcheck.txt. Имя можно дать любое (например, сделать несколько запусков с разными доменами в разные файлы). А уже потом сравнить стратегии, которые работают для большинства сайтов, и добавить в файл `/opt/zapret/config`.

Ниже показана вполне нормальная работа скрипта:

![block_out](https://github.com/user-attachments/assets/4c646793-d763-46f9-a5cb-4646e69c9f6a)

`PR_SET_NO_NEW_PRIVS(prctl): Invalid argument` --Данная команда поддерживается только начиная с ядра 3.5 (у нас 2.6). На работу не влияет.
Остальные предупреждения на работу также не влияют.

### 4.4) Настройка основного конфига
`/opt/zapret/install_easy.sh`

Перед запуском узнаем имя WAN интерфейса `nvram get wan0_ifname` ( обычно [**eth0**], если [**vlan2**] - проверьте, что NAT Acceleration отключен), а также LAN интерфейса `nvram get lan_ifname` (в режиме роутера обычно [**br0**])

  В целом рекомендую хотя бы почитать мануал по настройке и описанию переменных основного конфига в репозитории zapret.

  У меня выбран режим фильтрации MODE_FILTER=autohostlist. Список для обхода формируется автоматически - после нескольких неудачных попыток открыть сайт он добавляется в файл `/opt/zapret/ipset/zapret-hosts-auto.txt`

  Рекомендую также в конфиге `/opt/zapret/config` потом включить режим отладки AUTOHOSTLIST_DEBUGLOG=1 Вся инфа о возможных заблокированных сайтах будет записываться в `/opt/zapret/ipset/zapret-hosts-auto-debug.log`
  
  ***В конце работы скрипта могут быть следующие ошибки (у меня по крайней мере были)***

  `rndc: neither /opt/etc/bind/rndc.conf nor /opt/etc/bind/rndc.key was found` На это сообщение внимание не обращаем (в функции из zapret/ipset/def.sh идут проверки на все типы служб dns).

  `You (your login) are not allowed to use this program (crontab)` Если получаете такую ошибку, нужно изменить права crontab

  `chmod 644 /opt/bin/crontab`

***После настройки запускаем*** `/opt/zapret/init.d/sysv/zapret start`

## 5. Добавляем в автозапуск при старте системы

`ln -fs /opt/zapret/init.d/sysv/zapret /opt/etc/init.d/S99zapret`

## (Дополнительно) установка cURL  с поддержкой QUIC
`opkg remove curl`

`curl -Lo /tmp/curl.tar.xz https://github.com/stunnel/static-curl/releases/download/8.11.0/curl-linux-armv5-musl-8.11.0.tar.xz`

`tar -xf /tmp/curl.tar.xz -C /opt/bin curl`

`rm /tmp/curl.tar.xz`
