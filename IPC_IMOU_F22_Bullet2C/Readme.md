Summary:

Let me take you on a journal of what I did to a IMOU Bullet 2C camera (IPC-F22). This is just Part1 and more as running notes then a full pwn. 


Background: 

I don't trust anything I connect to a network. If possible I don't let anything access the internet unless it has to and even then I like to limit it's access to the least amount possible to function. 

When it comes with IoT devices like IP cameras from China I am even more untrusting. 

Some of the IP cameras I have poked have had hardcoded secret users and/or services that makes them very vulnerable vector on a network. They will also commonly enroll themself into P2P networks and tunnel traffic allowing instant footholds within a network. There have been hack in that past where by just knowing a serial number of a camera can allow you to remotely connect and control it anywhere in the world via it's P2P network. There is malware that exploits the security of these devices to enroll them into botnets. 





Phase 1: The target arrives. 

The target is a IMOU Bullet 2C camera from Aliexpress. https://www.imoulife.com/en/product/specs/bullet2c
It is claimed to support ONVIF, IP67 rating, Motion detection/Human Detection, night vision(30m), Wi-Fi:IEEE802.11b/g/n (Dual Antenna), 100Mb Ethernet, SDcard slot, H.265/H.264, internal microphone, 2MP(1080p)@30FPs,  
IMOU is a brand on Dahua. I have used Dahua cameras before and they have some nice features.  


Well with the cameras arrives, so time plug one in and see if it works. Can I connect it and use it as a camera offline? does it have all the features detailed? 


Well, No 

Using ONVIF manager it finds the camera, I can log into it using username 'admin' and the password of the safety code. 
I can see the RTSP streams it seems all the main ONVIF features are missing. I can not change the network settings or do anything other then access the RTSP steam. 
Trying to do anything else ends with a message "not implemented" or changes I make not being saved. 

ONVIF manager screenshots> 


The stream quality looks OK with the default settings but that watermark has to go. 

picture quality
VLC codec info screenshots

The default main video stream is 1080p 30fps h.265 VBR 1024kps
`rtsp://192.168.2.103:554/cam/realmonitor?channel=1&subtype=0&unicast=true&proto=Onvif`
The sub stream is is VGA 30fps h.265 VBR 512kps
`rtsp://192.168.2.103:554/cam/realmonitor?channel=1&subtype=1&unicast=true&proto=Onvif`
the Audio is AAC(mp4a) 32KHz, 32bits per sample (same on both streams) 
The stream delay is about 5 seconds  

Hmmm maybe I will check the web interface. 

Http no data

Nope there is no web interface

OK what services are running on the camera

```
D:\VMshared>nmap -sS 192.168.2.103 -p-
Starting Nmap 7.92 ( https://nmap.org ) at 2023-08-03 22:44 AUS Eastern Standard Time
Nmap scan report for 192.168.2.103
Host is up (0.0075s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE
80/tcp    open  http
554/tcp   open  rtsp
8086/tcp  open  d-s-n
37777/tcp open  unknown
```


OK look locked down a bit, Gone are the days of telnet running on the device with a root shell

Dahua has become a little more security mature over the last few years after a few backdoors became public and thier devices becoming the target of malware. 


maybe I should check the manual, nope it was useless. 

Time to check the software, I installed the IMOU software onto disposable Windows 10 VM in an isolated network with the camera. Nope need an internet connection and to sign up. will was avoiding that. 

##IMOU sign in

Maybe I can use the Dahua Config tool. Yep finds the camera but can not sign in ?!?. is the password on port 37777 different to the ONVIF password? I know password is correct as I am still watching the RTSP stream. 

##Config tool password error

Maybe Dahua's SmartPSS will have more luck. Nope

##SmartPSS login error

This is odd, I will try to do a factory reset on the camera. Held the button for 10 seconds while powering up the camera and now it shows up and un-initialized. Cool, I will use the Config tool to initialize the camera. 

Nope. Error  
##Config tool init error
Maybe SmartPSS Nope

RTSP still works using admin/safety code. 

Time for a bit more recon, googled most of the details I have so far and did not find much. Maybe someone should post some stuff about this camera. I might just have to do that to help others. 

Checked waybach machine to see if anything interesting on the IMOU site. nope

Next checked the FCC-ID to see if it has more info 
FCC report https://fcc.report/FCC-ID/2AVYF-IPC-FX2

WTF the second antenna is just cosmetic ?!? well atleast one of them is a real antenna, I have seen few fake plastic antennas in my time but this one atleast looks like a functional part.    

##FCC cosmetic antenna


Decided to send an email to the seller to see if they know of a way to configure the camera without internet access. 
Nope they were useless, they could get someone to email me the manual or help request a refund. 

Emailed IMOU support to see if they can help me with a way to configure the camera without an internet connection. 

Nope, as was as expected but 
```
Dear customer

Good day to you!

1, for your question, you can not add the device without the internet connection , when you try to add the device, we suggest that you can set the wifi at 2.4 GZ and have a try
2, you can not add the device in the pc version of the app , it has to be done in the phone, or you can not add it
3, you can try to set the ipc to your NVR and have a try ,  we  have  the  document  of  the adding  the  device of   the  NVR  of  imou  brand   the  adding  is  casi  the  same, i will send you relevant documents , you can have a check first

any further question, you can feel free to us , thanks for your cooperation

Best Wishes

IMOU Support Team
```

Hmmm will try the Android version of the app. Using an Andoid VM I installed the IMOU APP. 
Crap need to sign up, OK I will give it a email address they can already tie this device to me via the emails to IMOU support. 
##location permissions
Oh the app wants to access my location data and will not continue unless I allow. Well it is a VM and I am mocking the location. I will allow this time. 

Oh want access to my files too now?

Oh and system access?  
##system access

Rootkits don't even need this level of access. This app is not going anywhere near a real device of mine. 

OK now search for the camera, nope the app can not find the camera. 
but using a live stream menu it can see the camera (but can not do much or configure anything).  

Interestingly after connecting the the camera via the app the camera is now initialized and not SmartPSS and the Config tool can connect to the camera. 
But like with ONVIF I can not fully configure it, But I can the some of the settings using the configtool/SmartPSS(Ethernet IP settings, Encoder framerate). 


OK, so time for a summary so far:
-RTSP works
-ONVIF only lets me access the RTSP streams
-Very limited access to basic settings (encoder settings, IP settings)  
-No web interface 
-The IMOU PC app/Android app mostly useless and wants system access to my devices.





## Phase 2: Time for tools


Opening the camera is easy, after you pop the face cover off there are 4 screws. 

OK I can see there looks like a UART header on the PCB, time to solder a header to it. 
Remembering back to the FCC report photos I saw a header on the test board, I am going to replicate that. 


With a multimeter, finding the which pin is GND and 3.3volts was easy. For RX/TX I was going to make an educated guess, if I am wrong I am just need to reverse them. 
The pin order is 3v3, GND, TX, RX

I connected as follows (only connect GND, TX, RX), used a 3.3v USB UART adaptor

IPC    USB-UART(3v3)
GND----GND
TX-----RX
RX-----TX

##PCB UART photo

OK time to open serial port with putty 115200 8,1,N 


Yep it worked, can see the boot output

```
IPL 3b490ba
D-05
64MB
BIST0_0001-OK
Load IPL_CUST from NOR
MXP found at 0x0000a000
offset:00004800
Checksum OK

IPL_CUST 3b490ba
MXP found at 0x0000a000
runUBOOT()
[SPI_NOR]
offset:00010000
 -Verify CRC32 passed!
 -Decompress XZ
  u32HeaderSize=0x00000040
  u32Loadsize=0x000168f4
  decomp_size=0x00038b98
Disable MMU and D-cache before jump to UBOOT

U-Boot 2010.06-svn31228 (May 27 2022 - 11:32:49)

mount: mounting none on /proc/bus/usb failed: No such file or directory
backup have_backup_partition 1

config1 backup partion ok
keyboard = 1
/usr/etc/imod: line 1: #!/bin/sh: not found
Read RegisterPair set register: Connection timed out
real SensorType:gc2053_MIPI
AEWBCFGType:GC2053
err: FILE -> hwid.c, LINE -> 138:
Invalid HWID Info Command
aewcfg:IPC-C22C-GC2053
[: 1: unknown operand
8188fu 523609 0 - Live 0xbfa4a000 (O)
ehci_hcd 25988 0 - Live 0xbfa40000
usbcore 79336 2 8188fu,ehci_hcd, Live 0xbfa29000
usb_common 1728 1 usbcore, Live 0xbfa27000
pdc 160144 0 [permanent], Live 0xbf9f7000 (O)
xconfig 2648 1 pdc,[permanent], Live 0xbf9f3000 (O)
prc 45220 1 pdc,[permanent], Live 0xbf9e4000 (O)
binder 40608 0 [permanent], Live 0xbf9d7000 (O)
osa 40044 4 pdc,xconfig,prc,binder,[permanent], Live 0xbf9ca000 (O)
drv_ms_cus_gc2053_MIPI 5004 0 [permanent], Live 0xbf9c5000 (O)
drv_ms_cus_srcfg_drv 2332 0 [permanent], Live 0xbf9c1000 (O)
mi_ao 41648 0 [permanent], Live 0xbf9b3000 (O)
mi_ai 58816 0 [permanent], Live 0xbf9a1000 (O)
mi_divp 35012 0 [permanent], Live 0xbf995000 (O)
mi_venc 142436 0 [permanent], Live 0xbf96f000 (O)
mi_vif 26980 0 [permanent], Live 0xbf965000 (O)
mi_vpe 66360 0 [permanent], Live 0xbf951000 (O)
mi_rgn 72832 2 mi_divp,mi_vpe,[permanent], Live 0xbf93c000 (O)
mi_sensor 18008 0 [permanent], Live 0xbf934000 (O)
mi_sys 339780 8 mi_ao,mi_ai,mi_divp,mi_venc,mi_vif,mi_vpe,mi_rgn,mi_sensor,[permanent], Live 0xbf8de000 (O)
mi_common 4372 9 mi_ao,mi_ai,mi_divp,mi_venc,mi_vif,mi_vpe,mi_rgn,mi_sensor,mi_sys,[permanent], Live 0xbf8d9000 (O)
mhal 876300 10 drv_ms_cus_gc2053_MIPI,mi_ao,mi_ai,mi_divp,mi_venc,mi_vif,mi_vpe,mi_rgn,mi_sensor,mi_sys,[permanent], Live 0xbf800000 (O)
./usr/etc/user.sh: line 10: can't create /proc/sys/kernel/core_pattern: nonexistent directory

```



pressing * during the U-Boot stage I can gain access to the bootloader

Note: I have edited the SN, and MAC addresses
```
U-Boot 2010.06-svn31228 (May 27 2022 - 11:32:49)


disable wdt
> printenv
bootcmd=kload 0x22000000; bootm 0x22000000
bootdelay=1
baudrate=115200
ipaddr=192.168.1.108
serverip=192.168.1.1
gatewayip=192.168.1.1
netmask=255.255.255.0
bootfile="uImage"
da=tftp 0x22000000 dhboot.bin.img; flwrite
dr=tftp 0x22000000 romfs-x.squashfs.img; flwrite
dk=tftp 0x22000000 kernel.img; flwrite
du=tftp 0x22000000 user-x.squashfs.img; flwrite
dw=tftp 0x22000000 web-x.squashfs.img; flwrite
dp=tftp 0x22000000 partition-x.cramfs.img;flwrite
dc=tftp 0x22000000 custom-x.squashfs.img; flwrite
up=tftp 0x22000000 update.img; flwrite
tk=tftp 0x22000000 uImage; bootm
loglevel=4
pd=tftp 0x22000000 pd-x.squashfs.img; flwrite
ethact=sstar_emac
BSN=8L01DC3PBV00000
HWID=IPC-F22:01:02:03:AC:39:00:01:10:01:00:04:320:00:02:00:00:00:01:00:00:40
hwidEx=00:03:00:00:00:00:00:00:00:00:00:00:00:00:00:00
COUNTRYCODE=DE
devalias=IPC-F22
ID=8K051AEPAZ00000
SC=L2700000
wifiaddr=3C:E3:6B:00:00:00
ethaddr=3C:E3:6B:00:00:00
bootargs=mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000
filesize=71B97A
fileaddr=22000000
appauto=1
autolip=192.168.1.251
dh_keyboard=1
mp_autotest=0
stdin=serial
stdout=serial
stderr=serial
ver=U-Boot 2010.06-svn31228 (May 27 2022 - 11:32:49)

Environment size: 1246/65532 bytes
```


OK can I change the settings.  
```
> help
Unknown command 'help' - try 'help'
> setenv dh_keyboard=0
Failed to comper your cmd dh_keyboard=0, please input correct cmd
cmd fail
```
Hmmm they have locked U-BOOT shell down too. (Now I didn't read the error correctly but will get back to that later)


Time to test out my SPI flash programmer on it. 
I removed the heatsink and located the flash IC. 
The print on the IC was so faint that I had to play around with different lighting options and angles to try and read it. 


The SoC I could make out and find the datasheet for 

The flash I could not make out, but by looking at it and the tracks on the PCB can see that it is most likely a SPI flash IC so connected the programmer to it

Success. The programmer identified the SPI IC as a XM25QH64 8MB
datasheet https://datasheet.lcsc.com/lcsc/1811072025_XMC-XM25QH64AHIG_C328461.pdf

So now time to take a backup of the flash before I change anything. 
and then make a backup of the backup ;)


OK so now searched for the U-Boot parameters and found them at offset 0x00030000, values look NULL terminated

Changed from: 
dh_keyboard=1
appauto=1
bootargs mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000

to: 
dh_keyboard=0
appauto=0
bootargs mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000,single,init=/bin/sh

I then removed the write protection flags and copied my changes back to the SPI flash IC. 

Hmmm the camera booted into the limited Dahua DSH shell. 
Also my bootargs got changed on boot. 


Doticed that the camera tries to auto update from the network on every boot. 
```
SF: Detected nor0 with total size 16 MiB
partition file version 2
rootfstype squashfs root /dev/mtdblock4
can not search bootargs settings
fail to parse bootargsParametersV2.text info
fail to init bootargsParametersV2
In:    serial
Out:   serial
Err:   serial
TEXT_BASE:21000000
Net:   fail to load bootargsParameters.txt
fail to load bootargsParameters.txt file
MAC Address 3C:E3:6B:00:00:00
Auto-Negotiation...
Link Status Speed:100 Full-duplex:1

MMC:   MMC:   END
MDrv_EMAC_start
RBQP_BASE=0x2106c000
Using sstar_emac device
TFTP from server 192.168.254.254; our IP address is 192.168.1.251; sending through gateway 192.168.1.1
Download Filename 'upgrade_info_7db780000000.txt'.
Download to address: 0x22000000
Downloading: *
Retry count exceeded; starting again
cmd fail
Try again use backup_serverip
MDrv_EMAC_start
RBQP_BASE=0x2106c000
Using sstar_emac device
TFTP from server 192.168.254.254; our IP address is 192.168.1.251; sending through gateway 192.168.1.1
Download Filename 'upgrade_info_7db780000000.txt'.
Download to address: 0x22000000
Downloading: * ààà ü
IPL 3b490ba
D-05
```



Time to dig into the firmware:
As I have a copy of the firmware directly from the camera I fired up Kali and ran 

binwalk -e firmware.bin

This extrated out the root SqashyFS filesystem, and a few other files.

Now to go looking for what secrets it hides. 

There is a sshd user in /etc/passwd but login disabled. 
```
root:x:0:0:root:/:/bin/sh
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
admin:x:0:502:Linux User,,,:/:/bin/dsh
```

Well looks like telnetd is still possible, if I can add a file to /usr/data/
Ref: /etc/init.d/rcS
```
# 生产程序启动telnetd
if [ -f /usr/data/imgFlag ]; then
    /sbin/telnetd &
fi
``` 

It looks like they are also not storing the private keys plain text still. 
```
└─$ cat privkey.pem 
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
..<not including key>..
```

Also has a few other public keys, and makes use of SSL pinning
```
┌──(kali㉿kali)-[/mnt/…/squashfs-root/usr/bin/ssl]
└─$ ls     
cacert.pem  logPubkey.pem  privkey.pem  pwdreset.pem
```

It looks like the all the cameras functons are handled by a 9MB binary called 'sonia', 
```
└─$ ls -lha
total 9.4M
drwxrwx--- 1 root vboxsf 4.0K May 27  2022 .
drwxrwx--- 1 root vboxsf 4.0K May 27  2022 ..
drwxrwx--- 1 root vboxsf    0 May 27  2022 cloud_ca
-rwxrwx--- 1 root vboxsf 247K May 27  2022 hostapd
-rwxrwx--- 1 root vboxsf 5.7K May 27  2022 key
-rwxrwx--- 1 root vboxsf  34K May 27  2022 ptzAutoCheck
-rwxrwx--- 1 root vboxsf  37K May 27  2022 qr
drwxrwx--- 1 root vboxsf    0 May 27  2022 secboot
-rwxrwx--- 1 root vboxsf 9.1M May 27  2022 sonia
drwxrwx--- 1 root vboxsf    0 May 27  2022 ssl
-rwxrwx--- 1 root vboxsf  18K May 27  2022 utils_lcdPipe.bin
```



Another file found in the firmware was bootargsParameterV2.txt

```
#hwidÒÔ{½áÊø Ã¿ÐÐÒÔ»Ø³µ½áÊø
#²ÎÊý·ÅÔÚ""ÖÐ£¬¸ñÊ½£ºxxx="xxx"
IPC-C22C-LC:01:02:0F:AC:39:00:01:10:01:00:04:320:00:02:00:00:0A:01:00:00:40{ bootargs:"mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000" sdupdate:"gpio=66 ledtype=state" sdledctrl:"gpiopower=79 gpiostatus=61" }
IPC-C22C:01:02:0F:AC:39:00:01:10:01:00:04:320:00:02:00:00:0A:01:00:00:40{ bootargs:"mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000" sdupdate:"gpio=66 ledtype=state" sdledctrl:"gpiopower=79 gpiostatus=61" }
IPC-C22S-LC:01:02:0F:AC:39:00:01:10:01:01:04:320:00:02:00:00:0A:01:00:00:40{ bootargs:"mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000" sdupdate:"gpio=66 ledtype=state" sdledctrl:"gpiopower=79 gpiostatus=61" }
IPC-C22SP-imou:01:02:0F:AC:39:00:01:10:01:01:04:320:00:02:00:00:0A:01:00:00:40{ bootargs:"mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000" sdupdate:"gpio=66 ledtype=state" sdledctrl:"gpiopower=79 gpiostatus=61" }
IPC-G22F-LC:01:02:02:AC:39:00:01:10:01:00:04:320:00:02:00:00:00:01:00:00:40{ bootargs:"mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000" sdupdate:"gpio=66 ledtype=state" sdledctrl:"gpiopower=79 gpiostatus=61" }
IPC-G22FE-LC:01:02:02:C1:39:00:01:10:01:01:04:320:00:02:00:00:08:01:00:00:40{ bootargs:"mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000" sdupdate:"gpio=66 ledtype=state" sdledctrl:"gpiopower=79 gpiostatus=61" }
IPC-F22-LC:01:02:03:AC:39:00:01:10:01:00:04:320:00:02:00:00:00:01:00:00:40{ bootargs:"mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000" sdupdate:"gpio=66 ledtype=state" sdledctrl:"gpiopower=79 gpiostatus=61" }
IPC-F22:01:02:03:AC:39:00:01:10:01:00:04:320:00:02:00:00:00:01:00:00:40{ bootargs:"mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000" sdupdate:"gpio=66 ledtype=state" sdledctrl:"gpiopower=79 gpiostatus=61" }
IPC-A26L-LC:03:02:08:AC:3C:00:01:10:01:01:04:320:00:02:00:00:00:00:00:00:40{ bootargs:"mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000" sdupdate:"gpio=66 ledtype=state" sdledctrl:"gpiopower=79 gpiostatus=61" }
IPC-A26LP-imou:03:02:08:AC:3C:00:01:10:01:01:04:320:00:02:00:00:00:00:00:00:40{ bootargs:"mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000" sdupdate:"gpio=66 ledtype=state" sdledctrl:"gpiopower=79 gpiostatus=61" }
"bootargsParameter.text file end"
```

Using Cyberchef I converted the comment text encoding to 51936 and translated with google to  
```
#hwid ends with { and each line ends with a carriage return
#Parameters are placed in "", format: xxx="xxx"
```

So it looks like the u-boot HWID paramater is used to update the bootargs on boot. 


Well time to have another look and U-Boot
Now this is where I realised a mistake I made before, I don't set `setenv` with a `=`  

So I changed HWID to a different value and I was right. My bootargs were not overwritten but still ended up in the DSH limited shell. 

```
setenv HWID zPC-F22:01:02:03:AC:39:00:01:10:01:00:04:320:00:02:00:00:00:01:00:00:40
saveenv
boot
```

in the boot message I did notice a message `/bin/sh` not found. Maybe a BusyBox ?
tried telnetd (also BusyBox) same thing. 
```
setenv bootargs mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000 single init=/sbin/telnetd
```

And at this point I could not type into DSH, but could in U-Boot. 
hmmmm time to revert some changes
```
setenv HWID IPC-F22:01:02:03:AC:39:00:01:10:01:00:04:320:00:02:00:00:00:01:00:00:40
setenv bootargs mem=65304K console=ttyS0,115200 root=/dev/mtdblock4 rootfstype=squashfs LX_MEM=0x3fc6000 mma_heap=mma_heap_name0,miu=0,sz=0x1000000
setenv dh_keyboard 1
setenv appauto 1
saveenv
boot
```

nope still not working. 
OK use the SPI programmer and write the flash back to the IC 
after writting I read the chip to check somethings but notice that there is a lot of garbble where there was text before. 

try again, write to IC, same thing, garbbled. 

Erase first then flash? Yep that worked. All looks good now. 


OK time to call it a day for now, will continue later



So for my things to try later. 
--Look into ways of writing to /usr/data/ to enable telnetd 
--/etc/passwd details a sshd user, enable SSH
--Extract SquashyFS to SDcard as ext4 and boot from that (so I can edit the root FS with easy)
--Configure U-Boot to boot from NFS? 
--Edit the SquashFS (unpack, edit, repack) 

