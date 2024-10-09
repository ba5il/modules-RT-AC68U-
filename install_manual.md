opkg install curl bind-tools unzip ipset iptables xtables-addons_legacy cron coreutils-id  
В файле /opt/etc/init.d/S10cron вместо ARGS="-s" оставляем просто ARGS="" иначе в логах будет мусор каждую минуту.


curl -L -o /tmp/zapret.zip https://github.com/bol-van/zapret/archive/refs/heads/master.zip
unzip /tmp/zapret.zip -d /opt
rm /tmp/zapret.zip
mv /opt/zapret-master /opt/zapret

curl -L -o /tmp/modules.zip https://github.com/ba5il/modules-RT-AC68U-/archive/refs/heads/main.zip
unzip /tmp/modules.zip -d /opt
rm /tmp/modules.zip
mv /opt/modules-RT-AC68U--main/ /opt/modules_add/
/opt/modules_add/ins_mod
ln -fs /opt/modules_add/ins_mod  /opt/etc/init.d/S98insmod

/opt/zapret/install_bin.sh
/opt/zapret/install_easy.sh
rndc: neither /opt/etc/bind/rndc.conf nor /opt/etc/bind/rndc.key was found
На эту ошибку внимание не обращать (в функции идут проверки на все типы dns серверов zapret/ipset/def.sh).
You (your login) are not allowed to use this program (crontab)
Если получаете такую ошибку, нужно изменить права crontab
chmod 644 /opt/bin/crontab

ln -fs /opt/zapret/init.d/sysv/zapret /opt/etc/init.d/S99zapret


**Подключите необходимые модули**

nano /jffs/scripts/init-start

Добавьте в конце:



#!/bin/sh
modprobe xt_NFQUEUE
modprobe ip_set
modprobe ip_set_hash_ip
modprobe ip_set_hash_net
modprobe ip_set_bitmap_ip
modprobe ip_set_list_set
modprobe xt_set
