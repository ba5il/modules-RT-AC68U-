**Устанавливаем необходимые пакеты**

`opkg install curl bind-tools git-http ipset iptables xtables-addons_legacy cron coreutils-id ` 

`cron` здесь нужен только
В файле /opt/etc/init.d/S10cron вместо ARGS="-s" оставляем просто ARGS=""

**Скачиваем недостающие модули и переносим их в раздел jffs**

`git clone --depth 1 https://github.com/ba5il/modules-RT-AC68U- /opt/gittmp`

`mv /opt/gittmp/modules /jffs/modules`

`rm -r /opt/gittmp`

**Добавляем подключение необходимых модулей в скрипт при старте роутера**
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
insmod /jffs/modules/xt_connbytes.ko
```

**Теперь можно заняться настройкой и запуском основного скрипта**

`git clone --depth 1 https://github.com/bol-van/zapret /opt/zapret`

Устанавливаем нужные бинарники  `/opt/zapret/install_bin.sh`

Настройка основного конфига `/opt/zapret/install_easy.sh`

rndc: neither /opt/etc/bind/rndc.conf nor /opt/etc/bind/rndc.key was found
На эту ошибку внимание не обращать (в функции идут проверки на все типы dns серверов zapret/ipset/def.sh).
You (your login) are not allowed to use this program (crontab)
Если получаете такую ошибку, нужно изменить права crontab
chmod 644 /opt/bin/crontab

**Добавляем в автозапуск при старте системы**

`ln -fs /opt/zapret/init.d/sysv/zapret /opt/etc/init.d/S99zapret`
