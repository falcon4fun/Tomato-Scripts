# TomatoScripts

### Requirements
1. timeout packet from Entware

### Main mount goal is to:
1. Check script for duplicates
2. Launch Entware service only after time sync. Waits 120s with 0.5s checking
3. Enable swap

### Main unmount goal is to:
1. Dismount correctly
2. Kill dnscrypt monitoring script because I use entware version
3. Gracefully disable Netdata (Entware monitoring packet)
4. Check if it was stopped or wait 120s with 0.5s checking to continue scipt
5. Kill all logging because it's on /opt partition
6. Disable swap
7. Unmount partitions

### Run after mounting
```bash
[[ $(pgrep -f -c "`basename \"$0\"`") -gt 1 ]] && echo logger "====== DUP DETECTED: mount ======" && exit
logger "====== USB Mount Start ======"
/sbin/swapon /dev/sda2
logger "====== NTP_READY WAIT ======"
timeout 120 bash -c 'while [ $(nvram get ntp_ready) = 0 ]; do sleep 0.5; done'
logger "====== NTP_READY FINISH ======"
/opt/etc/init.d/rc.unslung start
#cp -f /opt/root/custom.css /opt/www/ext/
/opt/root/dnscrypt_restart &
logger "====== USB Mount Finish======"
```

### Run before unmounting
```bash
[[ $(pgrep -f -c "`basename \"$0\"`") -gt 1 ]] && echo logger "====== DUP DETECTED: unmount ======" && exit
logger "====== USB UNmount Start ======"
killall dnscrypt_restart
/opt/etc/init.d/rc.unslung stop

#Graceful netdata shutdown
logger "====== Netdata Shutdown Wait ======"
/opt/bin/timeout 120 bash -c 'while [ -n "$(pidof netdata && true)" ]; do sleep 0.5; done'
logger "====== Netdata Shutdown Finish ======"

killall -15 syslogd
killall -15 klogd
sleep 3

swapoff -a
umount /tmp/mnt/LOGS
umount /opt
logger "====== USB UNmount Finish ======"
```
