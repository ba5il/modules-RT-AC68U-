**1. Устанавливаем необходимые пакеты**

`opkg install curl bind-tools git-http ipset iptables xtables-addons_legacy cron coreutils-id coreutils-sort coreutils-sleep gzip ncat procps-ng-pgrep procps-ng-sysctl` 

В файле `/opt/etc/init.d/S10cron` вместо ARGS="-s" оставляем просто ARGS="", чтобы не было постоянного флуда в системный лог.

**2. Скачиваем недостающие модули и переносим их в раздел jffs**

`git clone --depth 1 https://github.com/ba5il/modules-RT-AC68U- /opt/mod_add`

`mv /opt/mod_add/modules /jffs/modules`

`rm -r /opt/mod_add`

**3. Добавляем подключение необходимых модулей в скрипт при старте роутера**

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
insmod /jffs/modules/nfnetlink_queue.ko
insmod /jffs/modules/xt_owner.ko
```
После этого выполняем `/jffs/scripts/init-start`, чтобы все необходимые модули подключились.

**4. Теперь можно заняться настройкой и запуском основного скрипта**

4.1) Клонируем репозиторий `git clone --depth 1 https://github.com/bol-van/zapret /opt/zapret`

4.2) Устанавливаем нужные бинарники  `/opt/zapret/install_bin.sh`

  После этого рекомендую почитать `https://github.com/bol-van/zapret/blob/master/docs/quick_start.txt` с 7-го пункта.

4.3) Запускаем поиск возможных стратегий обхода `/opt/zapret/blockcheck.sh | tee /opt/zapret/blockcheck.txt`

  Весь вывод работы скрипта будет записан в файл blockcheck.txt. Имя можно дать любое (например, сделать несколько запусков с разными доменами в разные файлы). А уже потом сравнить стратегии, которые работают для большинства сайтов, и добавить в файл `/opt/zapret/config`.

Ниже показана вполне нормальная работа скрипта:

![block_out](https://github.com/user-attachments/assets/4c646793-d763-46f9-a5cb-4646e69c9f6a)

`PR_SET_NO_NEW_PRIVS(prctl): Invalid argument` --Данная команда поддерживается только начиная с ядра 3.5 (у нас 2.6). На работу не влияет.
Остальные предупреждения на работу также не влияют.

4.4) Настройка основного конфига `/opt/zapret/install_easy.sh` Перед запуском рекомендую через команду `ifconfig` узнать имя WAN интерфейса (с вашим внешним IP, у меня был eth0), а также LAN интерфейсов.

  В целом рекомендую хотя бы почитать мануал по настройке и описанию переменных основного конфига в репозитории zapret.

   У меня выбран режим фильтрации MODE_FILTER=hostlist. Обход применяется ко всем доменам, кроме списка `/opt/zapret/ipset/zapret-hosts-user-exclude.txt` Сюда можно добавлять сайты, которые не заблокированы, но при запуске обхода ломаются.
  Если вам нужен именно такой режим - нужно сделать пустым файл `/opt/zapret/ipset/zapret-hosts-users.txt`.
  
  ***В конце работы скрипта могут быть следующие ошибки (у меня по крайней мере были)***

  `rndc: neither /opt/etc/bind/rndc.conf nor /opt/etc/bind/rndc.key was found` На это сообщение внимание не обращаем (в функции из zapret/ipset/def.sh идут проверки на все типы служб dns).

  `You (your login) are not allowed to use this program (crontab)` Если получаете такую ошибку, нужно изменить права crontab

  `chmod 644 /opt/bin/crontab`

***После настройки запускаем*** `/opt/zapret/init.d/sysv/zapret start`

**5. Добавляем в автозапуск при старте системы**

`ln -fs /opt/zapret/init.d/sysv/zapret /opt/etc/init.d/S99zapret`
