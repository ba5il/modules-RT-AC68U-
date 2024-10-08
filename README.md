# modules-RT-AC68U-
Additional modules for Asus-Merlin that are not compiled in the firmware. Must help to make 'zapret' (from bol-van) work on your router.

Use 'insmod /path/to/module' command to insert the modules

Example: insmod /opt/modules_add/nfnetlink_queue.ko

Add this code to the beginning of the script 'zapret':

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
