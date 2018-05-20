---
layout: article
title: "Slackware Porting Notes"
modified: 2018-05-20T21:56:30+0800
categories: presentation
excerpt: "Slackware Porting Notes"
tags: []
image:
  feature:
  teaser: skinny-bones-demo.gif
  thumb:
date: 2014-12-16
---

### Category

1.  [Intall Slackware64](#install)
2.  [Add user](#adduser)
3.  [Edit profile (PATH)](#profile)
4.  [System upgrade to Slackware64 current version](#upgrade)
5.  [Change lilo timeout](#timeout)
6.  [Install necessary softwares](#instsoft)
7.  [Configuring Graphical Logins](#xlogin)
8.  [Network configuration](#init1)
9.  [Install our own source and Customize our own system](#mkbp)
10. [Porting](#porting)
11. [User Mode Linux](#uml)
12. [diff Debian Slackware](#diffDS)

--------

1.  Intall Slackware64
    -   [Install Help](http://www.slackware.com/install/)
        -   [Software Sets](http://www.slackware.com/install/softwaresets.php)

|   A   | The base system. Contains enough software to get up and running and have a text editor and basic communications programs.                                                                          |
|:-----:|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   AP  | Various applications that do not require the X Window System.                                                                                                                                      |
|   D   | Program development tools. Compilers, debuggers, interpreters, and man pages. It's all here.                                                                                                       |
|   E   | GNU Emacs. Yes, Emacs is so big it requires its own series.                                                                                                                                        |
|   F   | FAQs, HOWTOs, and other miscellaneous documentation.                                                                                                                                               |
| GNOME | The GNOME desktop environment.                                                                                                                                                                     |
|   K   | The source code for the Linux kernel.                                                                                                                                                              |
|  KDE  | The K Desktop Environment. An X environment which shares a lot of look-and-feel features with the MacOS and Windows. The Qt widget library is also in this series, as KDE requires it to function. |
|  KDEI | Language support for the K Desktop Environment.                                                                                                                                                    |
|   L   | System libraries.                                                                                                                                                                                  |
|   N   | Networking programs. Daemons, mail programs, telnet, news readers, and so on.                                                                                                                      |
|   T   | teTeX document formatting system.                                                                                                                                                                  |
|  TCL  | The Tool Command Language, Tk, TclX, and TkDesk.                                                                                                                                                   |
|   X   | The base X Window System.                                                                                                                                                                          |
|  XAP  | X applications that are not part of a major desktop environment. For example Ghostscript and Netscape.                                                                                             |
|   Y   | Games (the BSD games collection, Sasteroids, Koules, and Lizards). 


    -   [Making Tags and Tagfiles](http://www.slackbook.org/html/package-management-making-tags-and-tagfiles.html)
    -   [Configuration Help](http://www.slackware.com/config/)

2.  Add user
3.  Edit profile (PATH)

```sh
emacs ~/.profile
export PATH=.:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin
. ~/.profile 
echo $PATH
.:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin
```

4.  System upgrade to Slackware current version

    ### Modify /etc/slackpkg/mirrors

```sh
cd /etc/slackpkg/
sudo emacs mirrors
sudo mv mirrors~ mirrors.orig
diff mirrors mirrors.orig
382c382
< http://ftp.twaren.net/Linux/Slackware/slackware64-current/
---
> # http://ftp.twaren.net/Linux/Slackware/slackware64-current/
```

    ### Upgrade system via slackpkg (slackware package management)

```sh
sudo slackpkg update gpg
sudo slackpkg update
sudo slackpkg install-new
sudo slackpkg upgrade-all
...

Your kernel image was updated.  We highly recommend you run: lilo
Do you want slackpkg to run lilo now? (Y/n)

Some packages had new configuration files installed.
You have four choices:

(K)eep the old files and consider .new files later

(O)verwrite all old files with the new ones. The
    old files will be stored with the suffix .orig

(R)emove all .new files

(P)rompt K, O, R selection for every single file
            
What do you want (K/O/R/P)?
O
# Modify mirrors file again
sudo slackpkg update gpg
sudo slackpkg update
sudo slackpkg clean-system
sudo init 6
```

5.  Change lilo timeout

```sh
cd /etc
sudo emacs lilo.conf
sudo mv lilo.conf~ lilo.conf.orig
diff lilo.conf lilo.conf.orig 
36c36
< timeout = 50
---
> timeout = 1200
sudo lilo
Warning: LBA32 addressing assumed
Added Linux  *
One warning was issued.
```

6.  Install and configure necessary softwares

    ### SSH Configration /etc/ssh/ssh\_config and /etc/ssh/sshd\_config

```sh
cd /etc/ssh
sudo emacs sshd_config
sudo mv sshd_config~ sshd_config.orig
diff sshd_config sshd_config.orig
13c13
< Port 22
---
> #Port 22
44c44
< PermitRootLogin no
---
> #PermitRootLogin yes
102c102
< X11Forwarding yes
---
> #X11Forwarding no
sudo /etc/rc.d/rc.sshd restart
sudo emacs ssh_config
sudo mv ssh_config~ ssh_config.orig
diff ssh_config ssh_config.orig
37c37
< Port 22
---
> #   Port 22
49d48
< ForwardX11Trusted yes
```

    ### [Profiles /etc/profile, \~/.profile, and \~/.bashrc](http://docs.slackware.com/howtos:cli_manual:shells)

```sh
diff profile profile.orig 
80,82d79
< if [ -f $HOME/.profile ]; then
<   . $HOME/.profile
< fi
more ~/.profile       
export PATH=.:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin
more .bashrc
source /etc/profile
```

    ### [KVM compliation](http://140.120.7.21/LinuxRef/Cluster/KVM-HOWTO.html)

```sh
wget http://wiki.qemu-project.org/download/qemu-2.2.0.tar.bz2
tar xjvf qemu-2.2.0.tar.bz2
rm qemu-*.tar.bz2
cd qemu-2.2.0
configure --help
configure --cpu=x86_64
make
echo $?
0
sudo make install
sudo modprobe kvm-amd
lsmod |grep kvm
kvm_amd                50703  0 
kvm                   395304  1 kvm_amd
qemu-system-x86_64
cd /usr/local/bin/
ls -l kvm
lrwxrwxrwx 1 root root 18 Dec 14 17:35 kvm -> qemu-system-x86_64*
ls -l /dev/kvm 
crw------- 1 root root 10, 232 Dec 16 12:55 /dev/kvm
sudo chmod 666 /dev/kvm 
ls -l /dev/kvm 
crw-rw-rw- 1 root root 10, 232 Dec 16 12:55 /dev/kvm
kvm -enable-kvm
sudo chmod 600 /dev/kvm 
```

    ### Open vSwitch

```sh
wget http://openvswitch.org/releases/openvswitch-2.3.1.tar.gz
tar zxvf openvswitch-2.3.1.tar.gz
rm openvswitch-*.tar.gz
cd openvswitch-2.3.1
./configure --with-linux=/lib/modules/`uname -r`/build
make
echo $?
0
sudo make install
sudo make modules_install
sudo modprobe openvswitch
lsmod |grep openvswitch
openvswitch            60055  0 
vxlan                  27671  1 openvswitch
gre                     3625  1 openvswitch
# Initialize the configuration database using ovsdb-tool
sudo mkdir -p /usr/local/etc/openvswitch
sudo ovsdb-tool create /usr/local/etc/openvswitch/conf.db vswitchd/vswitch.ovsschema
# Start configuration database, ovsdb-server.
sudo ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
                  --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
                  --private-key=db:Open_vSwitch,SSL,private_key \
                  --certificate=db:Open_vSwitch,SSL,certificate \
                  --bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \
                  --pidfile --detach
# Initialize the database using ovs-vsctl. 
sudo ovs-vsctl --no-wait init
# Start the main Open vSwitch daemon
sudo ovs-vswitchd --pidfile --detach
2014-12-13T15:28:09Z|00001|reconnect|INFO|unix:/usr/local/var/run/openvswitch/db.sock: connecting...
2014-12-13T15:28:09Z|00002|reconnect|INFO|unix:/usr/local/var/run/openvswitch/db.sock: connected
# Stop the Open vSwitch daemons
sudo kill `cd /usr/local/var/run/openvswitch && cat ovsdb-server.pid ovs-vswitchd.pid`
more INSTALL
```

    ### UML utilities {#uml-ut}

```sh
wget http://prdownloads.sourceforge.net/user-mode-linux/uml_utilities_20040406.tar.bz2
tar xjvf uml_utilities_20040406.tar.bz2
cd tools
make all
sudo make install
```

7.  [Configuring Graphical Logins](http://docs.slackware.com/zh-tw:slackware:install)

    You can first test that X auto-detects your video correctly by
    issuing the \`startx\` command. If X11 starts and you end up at a
    desktop, you're probably good to go.

```sh
startx
sudo emacs inittab
sudo mv inittab~ inittab.orig
diff inittab inittab.orig
25c25
< id:4:initdefault:
---
> id:3:initdefault:
# logout
sudo init 4
# or reboot
```

    To select or switch between available desktop environments run
    xwmconfig as root.

8.  Network configuration

```sh
sudo cat rc.inet1.conf
# /etc/rc.d/rc.inet1.conf
#
# This file contains the configuration settings for network interfaces.
# If USE_DHCP[interface] is set to "yes", this overrides any other settings.
# If you don't have an interface, leave the settings null ("").

# You can configure network interfaces other than eth0,eth1... by setting
# IFNAME[interface] to the interface's name. If IFNAME[interface] is unset
# or empty, it is assumed you're configuring eth<interface>.

# Several other parameters are available, the end of this file contains a
# comprehensive set of examples.

# =============================================================================

# Config information for eth0:
IPADDR[0]="192.168.180.1"
NETMASK[0]="255.255.255.0"
USE_DHCP[0]=""
DHCP_HOSTNAME[0]=""

# Config information for eth1:
IPADDR[1]=""
NETMASK[1]=""
USE_DHCP[1]=""
DHCP_HOSTNAME[1]=""

# Config information for eth2:
IPADDR[2]=""
NETMASK[2]=""
USE_DHCP[2]=""
DHCP_HOSTNAME[2]=""

# Config information for eth3:
IPADDR[3]=""
NETMASK[3]=""
USE_DHCP[3]=""
DHCP_HOSTNAME[3]=""

# Default gateway IP address:
GATEWAY="192.168.180.254"

# Change this to "yes" for debugging output to stdout.  Unfortunately,
# /sbin/hotplug seems to disable stdout so you'll only see debugging output
# when rc.inet1 is called directly.
DEBUG_ETH_UP="no"

# Example of how to configure a bridge:
# Note the added "BRNICS" variable which contains a space-separated list
# of the physical network interfaces you want to add to the bridge.
#IFNAME[0]="br0"
#BRNICS[0]="eth0"
#IPADDR[0]="192.168.0.1"
#NETMASK[0]="255.255.255.0"
#USE_DHCP[0]=""
#DHCP_HOSTNAME[0]=""

## Example config information for wlan0.  Uncomment the lines you need and fill
## in your info.  (You may not need all of these for your wireless network)
#IFNAME[4]="wlan0"
#IPADDR[4]=""
#NETMASK[4]=""
#USE_DHCP[4]="yes"
#DHCP_HOSTNAME[4]="icculus-wireless"
#DHCP_KEEPRESOLV[4]="yes"
#DHCP_KEEPNTP[4]="yes"
#DHCP_KEEPGW[4]="yes"
#DHCP_IPADDR[4]=""
#WLAN_ESSID[4]=BARRIER05
#WLAN_MODE[4]=Managed
##WLAN_RATE[4]="54M auto"
##WLAN_CHANNEL[4]="auto"
##WLAN_KEY[4]="D5AD1F04ACF048EC2D0B1C80C7"
##WLAN_IWPRIV[4]="set AuthMode=WPAPSK | set EncrypType=TKIP | set WPAPSK=96389dc66eaf7e6efd5b5523ae43c7925ff4df2f8b7099495192d44a774fda16"
#WLAN_WPA[4]="wpa_supplicant"
#WLAN_WPADRIVER[4]="ndiswrapper"

## Some examples of additional network parameters that you can use.
## Config information for wlan0:
#IFNAME[4]="wlan0"              # Use a different interface name instead of
                                # the default 'eth4'
#HWADDR[4]="00:01:23:45:67:89"  # Overrule the card's hardware MAC address
#MTU[4]=""                      # The default MTU is 1500, but you might need
                                # 1360 when you use NAT'ed IPSec traffic.
#DHCP_KEEPRESOLV[4]="yes"       # If you don't want /etc/resolv.conf overwritten
#DHCP_KEEPNTP[4]="yes"          # If you don't want ntp.conf overwritten
#DHCP_KEEPGW[4]="yes"           # If you don't want the DHCP server to change
                                # your default gateway
#DHCP_IPADDR[4]=""              # Request a specific IP address from the DHCP
                                # server
#WLAN_ESSID[4]=DARKSTAR         # Here, you can override _any_ parameter
                                # defined in rc.wireless.conf, by prepending
                                # 'WLAN_' to the parameter's name. Useful for
                                # those with multiple wireless interfaces.
#WLAN_IWPRIV[4]="set AuthMode=WPAPSK | set EncrypType=TKIP | set WPAPSK=thekey"
                                # Some drivers require a private ioctl to be
                                # set through the iwpriv command. If more than
                                # one is required, you can place them in the
                                # IWPRIV parameter (separated with the pipe (|)
                                # character, see the example).

sudo /etc/rc.d/rc.inet1 restart
```

9.  Install our own source and Customize our own system

```sh
scp as:/home/hsu/inet/ShellScripts/mkBpSubD /tmp
diff mkBpSubD /tmp/mkBpSubD.orig
7c7
<     chown hsu:users /backup/${sD}
---
>     chown hsu:hsu /backup/${sD}
13c13
< chown hsu:users /home/CHinese /src1/CHDB /src1/emacs-21 /src1/LinuxRef /src2/DSSSL /src2/guile-2 /src2/OpenJade
---
> chown hsu:hsu /home/CHinese /src1/CHDB /src1/emacs-21 /src1/LinuxRef /src2/DSSSL /src2/guile-2 /src2/OpenJade
sudo /tmp/mkBpSubD
scp as:/home/hsu/inet/ShellScripts/getSource /tmp
/tmp/getSource
```

10. Porting
    1.  [CHinese](http://140.120.7.21/LectureNotes/Diaries/Topic-OS-1-2014.html#ChChin)
    2.  [emacs](http://140.120.7.21/LectureNotes/Diaries/Topic-OS-1-2014.html#EmacsPort)

```sh
find / -name crt1.o -print 2>/dev/null
/usr/lib64/crt1.o
cd /usr/lib 
sudo ln -s /usr/lib64/crt1.o
sudo ln -s /usr/lib64/crti.o
sudo ln -s /usr/lib64/crtn.o
ls -l crt*
lrwxrwxrwx 1 root root 17 Dec 14 17:09 crt1.o -> /usr/lib64/crt1.o
lrwxrwxrwx 1 root root 17 Dec 14 17:09 crti.o -> /usr/lib64/crti.o
lrwxrwxrwx 1 root root 17 Dec 14 17:09 crtn.o -> /usr/lib64/crtn.o
# You don't have the next file in system!
more /usr/bin/mail
#! /bin/bash
echo "Sorry, We don't do mail in this host!"
sudo emacs /usr/bin/mail
# copy the above two lines to /usr/bin/mail and save file.
sudo chmod 755 /usr/bin/mail
cd /src1/emacs-21/objects
myconf 
/src1/emacs-21/src/xterm.c: In function 'x_set_toolkit_scroll_bar_thumb':
/src1/emacs-21/src/xterm.c:8889:30: error: 'ScrollbarPart' has no member named 'scroll_mode'
   scroll_mode = sb->scrollbar.scroll_mode;
                              ^
/src1/emacs-21/src/xterm.c:8891:18: error: 'ScrollbarPart' has no member named 'scroll_mode'
     sb->scrollbar.scroll_mode = 0;
                  ^
/src1/emacs-21/src/xterm.c:8902:21: error: 'ScrollbarPart' has no member named 'scroll_mode'
        sb->scrollbar.scroll_mode = scroll_mode;
                     ^
make[3]: *** [xterm.o] Error 1
make[3]: Leaving directory `/src1/emacs-21/objects/src'
make[2]: *** [bootstrap-temacs] Error 2
make[2]: Leaving directory `/src1/emacs-21/objects/src'
make[1]: *** [bootstrap-src] Error 2
make[1]: Leaving directory `/src1/emacs-21/objects'
make: *** [maybe_bootstrap] Error 2
diff /src1/emacs-21/src/xterm.c /src1/emacs-21/src/xterm.c.orig
8880c8880
< #if defined(HAVE_XAW3D) && defined(XAW_ARROW_SCROLLBARS)
---
> #ifdef HAVE_XAW3D
8900c8900
< #if defined(HAVE_XAW3D) && defined(XAW_ARROW_SCROLLBARS)
---
> #ifdef HAVE_XAW3D
myconf
echo $?
0
./src/emacs&   
sudo make install
cleanDir
```

    3.  [psgml](http://140.120.7.21/LectureNotes/Diaries/Topic-OS-1-2014.html#EmacsPort)
    4.  [LaTeX](http://140.120.7.21/LectureNotes/Diaries/Topic-OS-1-2014.html#ChLaTeX)

```sh
cd /src1/CHDB/ChLaTeX
gv ChLaTeX.ps&
ReleaseChLaTeX
cd $HOME/ChLaTeX
to # Open README.html via firefox 
~/inet/ShellScripts/ChLaTeXInstall.sh
you need to install packages: jadetex, latex2html, texlive-base,
  texlive-base-bin, texlive-fonts-recommended, texlive-latex-base,
  texlive-latex-recommended, first.
```

    5.  [OpenJade](http://140.120.7.21/LectureNotes/Diaries/Topic-OS-1-2014.html#SchemeEnv)
    6.  guile-2
    7.  esch
    8.  slib, jacal, ilisp

```sh
texi2html docs/ilisp.texi
/bin/bash: texi2html: command not found
make: *** [install] Error 127
```

11. User Mode Linux
    1.  [VDE switch](http://vde.sourceforge.net/)

        ### Build VDE2

```sh
svn co https://vde.svn.sourceforge.net/svnroot/vde/trunk/vde-2 vde_svn
cd vde_svn
autoreconf -fi
./configure --enable-experimental
Configure results:

 + VDE CryptCab............ enabled
 + VDE Router.............. enabled
 + VDE VXLAN............... enabled
 + Python Libraries........ enabled
 + TAP support............. enabled
 + pcap support............ enabled
 + Experimental features... enabled
 - Profiling options....... disabled
 - Kernel switch........... disabled

make
echo $?
0
sudo make install
```

    2.  [UML utilities](#uml-ut)
    3.  Build from linux source

```sh
cd /src3/kernel/
cp -r /usr/src/linux-3.14.24 .
wget http://anonscm.debian.org/cgit/pkg-uml/user-mode-linux.git/plain/config.amd64
cd linux-3.14.24/
cp ../config.amd64 .config
make ARCH=um menuconfig
make ARCH=um linux 
ld -r -dp -o arch/um/drivers/vde.o arch/um/drivers/vde_kern.o arch/um/drivers/vde_user.o  -m elf_x86_64 -r libvdeplug.a
ld: cannot find libvdeplug.a: No such file or directory
find / -name libvdeplug.a 2>/dev/null
/usr/local/lib/libvdeplug.a
ln -s /usr/local/lib/libvdeplug.a
make ARCH=um linux
arch/um/drivers/built-in.o: In function `nl80211_init':
pcap-linux.c:(.text+0x1b604): undefined reference to `nl_socket_alloc'
pcap-linux.c:(.text+0x1b618): undefined reference to `genl_connect'
pcap-linux.c:(.text+0x1b62a): undefined reference to `genl_ctrl_alloc_cache'
pcap-linux.c:(.text+0x1b63e): undefined reference to `genl_ctrl_search_by_name'
pcap-linux.c:(.text+0x1b686): undefined reference to `nl_socket_free'
pcap-linux.c:(.text+0x1b695): undefined reference to `nl_geterror'
pcap-linux.c:(.text+0x1b700): undefined reference to `nl_cache_free'
arch/um/drivers/built-in.o: In function `nl80211_cleanup':
pcap-linux.c:(.text+0x1b719): undefined reference to `genl_family_put'
pcap-linux.c:(.text+0x1b722): undefined reference to `nl_cache_free'
arch/um/drivers/built-in.o: In function `del_mon_if.isra.13':
pcap-linux.c:(.text+0x1d26e): undefined reference to `nlmsg_alloc'
pcap-linux.c:(.text+0x1d282): undefined reference to `genl_family_get_id'
pcap-linux.c:(.text+0x1d2d0): undefined reference to `nl_send_auto_complete'
pcap-linux.c:(.text+0x1d2dd): undefined reference to `nl_wait_for_ack'
pcap-linux.c:(.text+0x1d2ed): undefined reference to `nlmsg_free'
pcap-linux.c:(.text+0x1d32a): undefined reference to `nlmsg_free'
pcap-linux.c:(.text+0x1d33d): undefined reference to `nl_geterror'
pcap-linux.c:(.text+0x1d366): undefined reference to `nlmsg_free'
pcap-linux.c:(.text+0x1d37d): undefined reference to `nl_geterror'
arch/um/drivers/built-in.o: In function `enter_rfmon_mode_mac80211':
pcap-linux.c:(.text+0x1d93f): undefined reference to `nlmsg_alloc'
pcap-linux.c:(.text+0x1d955): undefined reference to `genl_family_get_id'
pcap-linux.c:(.text+0x1da1f): undefined reference to `nl_send_auto_complete'
pcap-linux.c:(.text+0x1da2d): undefined reference to `nl_wait_for_ack'
pcap-linux.c:(.text+0x1da4a): undefined reference to `nlmsg_free'
pcap-linux.c:(.text+0x1da92): undefined reference to `nl_geterror'
pcap-linux.c:(.text+0x1dabb): undefined reference to `nlmsg_free'
pcap-linux.c:(.text+0x1dae2): undefined reference to `nlmsg_free'
pcap-linux.c:(.text+0x1dbe2): undefined reference to `nlmsg_free'
pcap-linux.c:(.text+0x1dc05): undefined reference to `nl_geterror'
arch/um/drivers/built-in.o: In function `nl80211_cleanup':
pcap-linux.c:(.text+0x1b72b): undefined reference to `nl_socket_free'
arch/um/drivers/built-in.o: In function `canusb_close':
pcap-canusb-linux.c:(.text+0x22f3a): undefined reference to `pthread_join'
pcap-canusb-linux.c:(.text+0x22f4b): undefined reference to `libusb_close'
arch/um/drivers/built-in.o: In function `canusb_opendevice':
pcap-canusb-linux.c:(.text+0x230a8): undefined reference to `libusb_get_device_list'
pcap-canusb-linux.c:(.text+0x230f1): undefined reference to `libusb_get_device_descriptor'
pcap-canusb-linux.c:(.text+0x2311c): undefined reference to `libusb_open'
pcap-canusb-linux.c:(.text+0x23137): undefined reference to `libusb_get_string_descriptor_ascii'
pcap-canusb-linux.c:(.text+0x2315e): undefined reference to `libusb_kernel_driver_active'
pcap-canusb-linux.c:(.text+0x2316e): undefined reference to `libusb_detach_kernel_driver'
pcap-canusb-linux.c:(.text+0x23181): undefined reference to `libusb_set_configuration'
pcap-canusb-linux.c:(.text+0x23191): undefined reference to `libusb_claim_interface'
pcap-canusb-linux.c:(.text+0x231a3): undefined reference to `libusb_close'
pcap-canusb-linux.c:(.text+0x231ba): undefined reference to `libusb_free_device_list'
pcap-canusb-linux.c:(.text+0x231dc): undefined reference to `libusb_free_device_list'
arch/um/drivers/built-in.o: In function `canusb_capture_thread':
pcap-canusb-linux.c:(.text+0x23214): undefined reference to `libusb_init'
pcap-canusb-linux.c:(.text+0x2326a): undefined reference to `libusb_interrupt_transfer'
pcap-canusb-linux.c:(.text+0x23288): undefined reference to `libusb_interrupt_transfer'
pcap-canusb-linux.c:(.text+0x232b2): undefined reference to `libusb_bulk_transfer'
pcap-canusb-linux.c:(.text+0x232e4): undefined reference to `libusb_close'
pcap-canusb-linux.c:(.text+0x232ee): undefined reference to `libusb_exit'
arch/um/drivers/built-in.o: In function `canusb_activate':
pcap-canusb-linux.c:(.text+0x234e6): undefined reference to `libusb_interrupt_transfer'
pcap-canusb-linux.c:(.text+0x2353d): undefined reference to `pthread_create'
arch/um/drivers/built-in.o: In function `canusb_findalldevs':
(.text+0x235bd): undefined reference to `libusb_init'
arch/um/drivers/built-in.o: In function `canusb_findalldevs':
(.text+0x235eb): undefined reference to `libusb_get_device_list'
arch/um/drivers/built-in.o: In function `canusb_findalldevs':
(.text+0x2365a): undefined reference to `libusb_get_device_descriptor'
arch/um/drivers/built-in.o: In function `canusb_findalldevs':
(.text+0x23687): undefined reference to `libusb_open'
arch/um/drivers/built-in.o: In function `canusb_findalldevs':
(.text+0x236a4): undefined reference to `libusb_get_string_descriptor_ascii'
arch/um/drivers/built-in.o: In function `canusb_findalldevs':
(.text+0x236f2): undefined reference to `libusb_close'
arch/um/drivers/built-in.o: In function `canusb_findalldevs':
(.text+0x23726): undefined reference to `libusb_free_device_list'
arch/um/drivers/built-in.o: In function `canusb_findalldevs':
(.text+0x2373b): undefined reference to `libusb_free_device_list'
arch/um/drivers/built-in.o: In function `canusb_findalldevs':
(.text+0x23745): undefined reference to `libusb_exit'
arch/um/drivers/built-in.o: In function `canusb_create':
(.text+0x2376b): undefined reference to `libusb_init'
collect2: error: ld returned 1 exit status
make: *** [vmlinux] Error 1
sudo slackpkg search libpcap

Looking for libpcap in package list. Please wait... DONE

The list below shows all packages with name matching "libpcap".

[ installed ] - libpcap-1.4.0-x86_64-1

sudo slackpkg search libnl
The list below shows all packages with name matching "libnl".

[ installed ] - libnl-1.1.4-x86_64-1
[ installed ] - libnl3-3.2.21-x86_64-1

sudo slackpkg search libusb 
The list below shows all packages with name matching "libusb".

[ installed ] - libusb-1.0.9-x86_64-1
[ installed ] - libusb-compat-0.1.4-x86_64-1

diff .config .config.orig 
1391c1391
< CONFIG_UML_NET_PCAP=n
---
> CONFIG_UML_NET_PCAP=y
make ARCH=um linux 
echo $?
0
ls -lia linux vmlinux
2628904 -rwxr-xr-x 2 jssu users 8596188 Dec 15 17:06 linux*
2628904 -rwxr-xr-x 2 jssu users 8596188 Dec 15 17:06 vmlinux*
linux --version
3.14.24
make ARCH=um modules 
echo $?
0
if [ ! -d  /usr/local/lib/uml ]
then sudo mkdir /usr/local/lib/uml 
 sudo chown jssu:users /usr/local/lib/uml 
  else rm -rf /usr/local/lib/uml/*
  fi
make ARCH=um modules_install INSTALL_MOD_PATH=/usr/local/lib/uml 
make ARCH=um modules_install INSTALL_MOD_PATH=/usr/local/lib/uml make headers_install ARCH=um INSTALL_HDR_PATH=/usr/local/lib/uml
make: *** No rule to make target `make'.  Stop.
echo $?
2
cp linux /usr/local/lib/uml/linux.uml 
#$ make clean
cd /usr/local/bin/
sudo ln -s /usr/local/lib/uml/linux.uml
```

### [UML root file system (slackware)](http://www.inreto.de/mkuml/resoa-uml.html), [uml root filesystem creation](http://amdm/uml-rfs/README.html#MLNTemplate), [Create Our own DebianNet Template for MLN](http://140.120.7.21/LinuxRef/Cluster/MLN-Notes.html#UML-Root-Fs)

```sh
dd if=/dev/zero of=root_fs_slackware.img bs=1M count=4096
sudo mkfs.ext4 root_fs_slackware.img 
sudo mount -o loop root_fs_slackware.img /mnt/tmp/
sudo mount /video/ISOs/slackware64-14.1-install-dvd.iso /mnt/dvd/
cd /mnt/dvd/slackware64
```

#### [Software Sets](http://www.slackware.com/install/softwaresets.php)

```sh
    # The -ask and -menu options provide an easy way to select only desired packages.
    # Note, that you do not need to (and should not) install any of the kernels.
sudo installpkg -root /mnt/tmp -ask -menu a/*.t[gx]z ap/*.t[gx]z d/*.t[gx]z e/*.t[gx]z n/*.t[gx]z
ls -l /mnt/tmp/
total 168
drwxr-xr-x  2 root root  4096 May 23  2009 bin/
drwxr-xr-x  2 root root  4096 Dec 15 18:14 boot/
drwxr-xr-x 17 root root 69632 Dec 15 18:12 dev/
drwxr-xr-x 65 root root  4096 Dec 15 18:24 etc/
drwxr-xr-x  2 root root  4096 Oct  6  1997 home/
drwxr-xr-x  5 root root  4096 May  3  2010 lib/
drwxr-xr-x  2 root root  4096 Dec 15 18:23 lib64/
drwx------  2 root root 16384 Dec 15 18:11 lost+found/
drwxr-xr-x 16 root root  4096 Dec 15 18:12 media/
drwxr-xr-x 10 root root  4096 Sep 26  2006 mnt/
drwxr-xr-x  2 root root  4096 Jun 10  2007 opt/
drwxr-xr-x  2 root root  4096 Oct  6  1997 proc/
drwx--x---  2 root root  4096 Oct  6  1997 root/
drwxr-xr-x  2 root root  4096 Apr 19  2013 run/
drwxr-xr-x  2 root root 12288 Oct 31  2008 sbin/
drwxr-xr-x  2 root root  4096 Dec 15 18:21 srv/
drwxr-xr-x  2 root root  4096 May 12  2004 sys/
drwxrwxrwt  4 root root  4096 Aug 29  2003 tmp/
drwxr-xr-x 16 root root  4096 Oct 18  2013 usr/
drwxr-xr-x 16 root root  4096 Oct 18  2013 var/
cd /usr/local/lib/uml/lib/modules
sudo mkdir /mnt/tmp/lib/modules
find . -print | sudo cpio -pdm /mnt/tmp/lib/modules/ 
45534 blocks
sync; sync 
sudo chroot /mnt/tmp/
# cd /dev 
# for i in 0 1 2 3 4 5 6 7; do mknod ubd$i b 98 $[ $i * 16 ]; done 
# ldconfig
# passwd root
# exit
sync; sync
sudo umount /mnt/tmp 
```

12. diff Debian Slackware

```
    Debian
    Slackware
    Package management tools
    apt, aptitude, dpkg
    slackpkg, pkgtool, [install|remove]pkg
    init scripts
    /etc/init.d/daemon, /etc/rc.local
    /etc/rc.d/rc.daemon, /etc/rc.d/rc.local
    Network configration files
    /etc/network/interfaces, /etc/resolv.conf
    /etc/rc.d/rc.inet1.conf, /etc/resolv.conf
```

* * * * *

[Chi-Sheng Su](mailto:lagnua@gmail.com)
