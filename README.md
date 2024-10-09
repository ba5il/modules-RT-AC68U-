# modules-RT-AC68U-
Дополнитальные модули, необходимые для работы 'zapret'

Для автоматического подключения необходимых внешних модулей нужно изменить функцию запуска do_start() в файле '/opt/zapret/init.d/sysv/zapret':


do_start()
{
        if m=$(lsmod | grep "nfnetlink_queue"); then
            echo "nfnetlink_queue.ko has already been inserted in modules"
        else
            echo "inserting module: nfnetlink_queue.ko"
            $(insmod /opt/modules_add/nfnetlink_queue.ko)
        fi
        if m=$(lsmod | grep "xt_owner"); then
            echo "xt_owner.ko has already been inserted in modules"
        else
            echo "inserting module: xt_owner.ko"
            $(insmod /opt/modules_add/xt_owner.ko)
        fi
        if m=$(lsmod | grep "xt_connbytes"); then
            echo "xt_connbytes.ko has already been inserted in modules"
        else
            echo "inserting module: xt_connbytes.ko"
            $(insmod /opt/modules_add/xt_connbytes.ko)
        fi
        zapret_run_daemons
        [ "$INIT_APPLY_FW" != "1" ] || { zapret_apply_firewall; }
}
