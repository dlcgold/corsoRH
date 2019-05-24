# LEZIONE 4

### INTRODUZIONE
Si hanno due mantra:
* in Linux tutto è un file e si hanno file di sistema, se "è reinstallabile", e file di produzione, anche i processi sono file, anche l'hw viene visto come un file
* Linux è un sistema operativo a macro kernel, ovvero il kernel, oltre a fare le 
  solite funzioni base (interfacciare l'hardware etc), è composto anche da moduli (i driver) che vengono gestiti direttamente dal kernel

Si ha il kickstart con un file che passa i parametri all'installer di una distro,
contenendo anche le informazoni per il tuning e un sistema di provisioning come ansible per il post installazione.
*$PS1* è la variabile del prompt.
### YUM

Il gestore di pacchetti è *rpm*:
```shell
[osboxes@osboxes ~]$ rpm
RPM version 4.11.3
Copyright (C) 1998-2002 - Red Hat, Inc.
This program may be freely redistributed under the terms of the GNU GPL

Usage: rpm [-aKfgpqVcdLilsiv?] [-a|--all] [-f|--file] [-g|--group] [-p|--package]
        [--pkgid] [--hdrid] [--triggeredby] [--whatrequires] [--whatprovides]
        [--nomanifest] [-c|--configfiles] [-d|--docfiles] [-L|--licensefiles]
        [--dump] [-l|--list] [--queryformat=QUERYFORMAT] [-s|--state]
        [--nofiledigest] [--nofiles] [--nodeps] [--noscript] [--allfiles]
        [--allmatches] [--badreloc] [-e|--erase <package>+] [--excludedocs]
        [--excludepath=<path>] [--force] [-F|--freshen <packagefile>+]
        [-h|--hash] [--ignorearch] [--ignoreos] [--ignoresize] [-i|--install]
        [--justdb] [--nodeps] [--nofiledigest] [--nocontexts] [--noorder]
        [--noscripts] [--notriggers] [--nocollections] [--oldpackage]
        [--percent] [--prefix=<dir>] [--relocate=<old>=<new>] [--replacefiles]
        [--replacepkgs] [--test] [-U|--upgrade <packagefile>+]
        [--reinstall=<packagefile>+] [-D|--define 'MACRO EXPR']
        [--undefine=MACRO] [-E|--eval 'EXPR'] [--macros=<FILE:...>]
        [--noplugins] [--nodigest] [--nosignature] [--rcfile=<FILE:...>]
        [-r|--root ROOT] [--dbpath=DIRECTORY] [--querytags] [--showrc]
        [--quiet] [-v|--verbose] [--version] [-?|--help] [--usage] [--scripts]
        [--setperms] [--setugids] [--conflicts] [--obsoletes] [--provides]
        [--requires] [--info] [--changelog] [--xml] [--triggers] [--last]
        [--dupes] [--filesbypkg] [--fileclass] [--filecolor] [--fscontext]
        [--fileprovide] [--filerequire] [--filecaps]
		
[osboxes@osboxes ~]$ rpm -qi zsh
Name        : zsh
Version     : 5.0.2
Release     : 31.el7
Architecture: x86_64
Install Date: Fri 17 May 2019 01:15:28 PM EDT
Group       : System Environment/Shells
Size        : 5854390
License     : MIT
Signature   : RSA/SHA256, Mon 12 Nov 2018 09:49:55 AM EST, Key ID 24c6a8a7f4a80eb5
Source RPM  : zsh-5.0.2-31.el7.src.rpm
Build Date  : Tue 30 Oct 2018 12:48:17 PM EDT
Build Host  : x86-01.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://zsh.sourceforge.net/
Summary     : Powerful interactive shell
Description :
The zsh shell is a command interpreter usable as an interactive login
shell and as a shell script command processor.  Zsh resembles the ksh
shell (the Korn shell), but includes many enhancements.  Zsh supports
command line editing, built-in spelling correction, programmable
command completion, shell functions (with autoloading), a history
mechanism, and more.

[osboxes@osboxes ~]$ rpm -q --scripts zsh
postinstall scriptlet (using /bin/sh):
if [ ! -f /etc/shells ] ; then
    echo "/bin/zsh" > /etc/shells
else
    grep -q "^/bin/zsh$" /etc/shells || echo "/bin/zsh" >> /etc/shells
fi

if [ -f /usr/share/info/zsh.info.gz ]; then
# This is needed so that --excludedocs works.
/sbin/install-info /usr/share/info/zsh.info.gz /usr/share/info/dir \
  --entry="* zsh: (zsh).			An enhanced bourne shell."
fi

:
preuninstall scriptlet (using /bin/sh):
if [ "$1" = 0 ] ; then
    if [ -f /usr/share/info/zsh.info.gz ]; then
    # This is needed so that --excludedocs works.
    /sbin/install-info --delete /usr/share/info/zsh.info.gz /usr/share/info/dir \
      --entry="* zsh: (zsh).			An enhanced bourne shell."
    fi
fi
:
postuninstall scriptlet (using /bin/sh):
if [ "$1" = 0 ] ; then
    if [ -f /etc/shells ] ; then
        TmpFile=`/bin/mktemp /tmp/.zshrpmXXXXXX`
        grep -v '^/bin/zsh$' /etc/shells > $TmpFile
        cp -f $TmpFile /etc/shells
        rm -f $TmpFile
    fi
fi

[osboxes@osboxes ~]$ rpm -q --scripts kernel
postinstall scriptlet (using /bin/sh):


/usr/sbin/new-kernel-pkg --package kernel --install 3.10.0-957.el7.x86_64 || exit $?
preuninstall scriptlet (using /bin/sh):
/usr/sbin/new-kernel-pkg --rminitrd --rmmoddep --remove 3.10.0-957.el7.x86_64 || exit $?
if [ -x /usr/sbin/weak-modules ]
then
    /usr/sbin/weak-modules --remove-kernel 3.10.0-957.el7.x86_64 || exit $?
fi
posttrans scriptlet (using /bin/sh):
if [ -x /usr/sbin/weak-modules ]
then
    /usr/sbin/weak-modules --add-kernel 3.10.0-957.el7.x86_64 || exit $?
fi
/usr/sbin/new-kernel-pkg --package kernel --mkinitrd --dracut --depmod --update 3.10.0-957.el7.x86_64 
rc=$?
if [ $rc != 0 ]; then
    /usr/sbin/new-kernel-pkg --remove 3.10.0-957.el7.x86_64
    ERROR_MSG="ERROR: installing kernel-3.10.0-957.el7.x86_64: no space left for creating initramfs. Clean up /boot partition and re-run '/usr/sbin/new-kernel-pkg --package kernel --mkinitrd --dracut --depmod --install 3.10.0-957.el7.x86_64'"
    if [ -e /usr/bin/logger ]; then
        /usr/bin/logger -p syslog.warn "$ERROR_MSG"
    elif [ -e /usr/bin/cat ]; then
        /usr/bin/cat "$ERROR_MSG" > /dev/kmsg
    fi
    echo "$ERROR_MSG"
    exit $rc
fi
/usr/sbin/new-kernel-pkg --package kernel --rpmposttrans 3.10.0-957.el7.x86_64 || exit $?
postinstall scriptlet (using /bin/sh):


/usr/sbin/new-kernel-pkg --package kernel --install 3.10.0-957.12.2.el7.x86_64 || exit $?
preuninstall scriptlet (using /bin/sh):
/usr/sbin/new-kernel-pkg --rminitrd --rmmoddep --remove 3.10.0-957.12.2.el7.x86_64 || exit $?
if [ -x /usr/sbin/weak-modules ]
then
    /usr/sbin/weak-modules --remove-kernel 3.10.0-957.12.2.el7.x86_64 || exit $?
fi
posttrans scriptlet (using /bin/sh):
if [ -x /usr/sbin/weak-modules ]
then
    /usr/sbin/weak-modules --add-kernel 3.10.0-957.12.2.el7.x86_64 || exit $?
fi
/usr/sbin/new-kernel-pkg --package kernel --mkinitrd --dracut --depmod --update 3.10.0-957.12.2.el7.x86_64 
rc=$?
if [ $rc != 0 ]; then
    /usr/sbin/new-kernel-pkg --remove 3.10.0-957.12.2.el7.x86_64
    ERROR_MSG="ERROR: installing kernel-3.10.0-957.12.2.el7.x86_64: no space left for creating initramfs. Clean up /boot partition and re-run '/usr/sbin/new-kernel-pkg --package kernel --mkinitrd --dracut --depmod --install 3.10.0-957.12.2.el7.x86_64'"
    if [ -e /usr/bin/logger ]; then
        /usr/bin/logger -p syslog.warn "$ERROR_MSG"
    elif [ -e /usr/bin/cat ]; then
        /usr/bin/cat "$ERROR_MSG" > /dev/kmsg
    fi
    echo "$ERROR_MSG"
    exit $rc
fi
/usr/sbin/new-kernel-pkg --package kernel --rpmposttrans 3.10.0-957.12.2.el7.x86_64 || exit $?

[osboxes@osboxes ~]$ rpm -qa kernel
kernel-3.10.0-957.el7.x86_64
kernel-3.10.0-957.12.2.el7.x86_64

[osboxes@osboxes ~]$ sudo grubby --default-kernel
[sudo] password for osboxes: 
/boot/vmlinuz-3.10.0-957.12.2.el7.x86_64

```
Poi il file *spec* è un file contenente il codice da eseguire in file di installazione o disinstallazione del rpm.
Se aggiorni il kernel non aggiorni effettivamente ma fai un'installazione seriale
con l'eventuale cancellazione del vecchio o uno più vecchio ancora.
Se un pacchetto vuole delle dipendenze rpm non è in grado di soddifarle, installa
infatti un solo pacchetto. Per questo si usa un package manager che sfrutta il gestore di pacchetti
che sfrutta i repository. Nei sistemi Red Hat è *yum*:
```shell
[osboxes@osboxes ~]$ yum
Loaded plugins: fastestmirror, langpacks
You need to give some command
Usage: yum [options] COMMAND

List of Commands:

check          Check for problems in the rpmdb
check-update   Check for available package updates
clean          Remove cached data
deplist        List a package's dependencies
distribution-synchronization Synchronize installed packages to the latest available versions
downgrade      downgrade a package
erase          Remove a package or packages from your system
fs             Acts on the filesystem data of the host, mainly for removing docs/lanuages for minimal hosts.
fssnapshot     Creates filesystem snapshots, or lists/deletes current snapshots.
groups         Display, or use, the groups information
help           Display a helpful usage message
history        Display, or use, the transaction history
info           Display details about a package or group of packages
install        Install a package or packages on your system
langavailable  Check available languages
langinfo       List languages information
langinstall    Install appropriate language packs for a language
langlist       List installed languages
langremove     Remove installed language packs for a language
list           List a package or groups of packages
load-transaction load a saved transaction from filename
makecache      Generate the metadata cache
provides       Find what package provides the given value
reinstall      reinstall a package
repo-pkgs      Treat a repo. as a group of packages, so we can install/remove all of them
repolist       Display the configured software repositories
search         Search package details for the given string
shell          Run an interactive yum shell
swap           Simple way to swap packages, instead of using shell
update         Update a package or packages on your system
update-minimal Works like upgrade, but goes to the 'newest' package match which fixes a problem that affects your system
updateinfo     Acts on repository update information
upgrade        Update packages taking obsoletes into account
version        Display a version for the machine and/or available repos.


Options:
  -h, --help            show this help message and exit
  -t, --tolerant        be tolerant of errors
  -C, --cacheonly       run entirely from system cache, don't update cache
  -c [config file], --config=[config file]
                        config file location
  -R [minutes], --randomwait=[minutes]
                        maximum command wait time
  -d [debug level], --debuglevel=[debug level]
                        debugging output level
  --showduplicates      show duplicates, in repos, in list/search commands
  -e [error level], --errorlevel=[error level]
                        error output level
  --rpmverbosity=[debug level name]
                        debugging output level for rpm
  -q, --quiet           quiet operation
  -v, --verbose         verbose operation
  -y, --assumeyes       answer yes for all questions
  --assumeno            answer no for all questions
  --version             show Yum version and exit
  --installroot=[path]  set install root
  --enablerepo=[repo]   enable one or more repositories (wildcards allowed)
  --disablerepo=[repo]  disable one or more repositories (wildcards allowed)
  -x [package], --exclude=[package]
                        exclude package(s) by name or glob
  --disableexcludes=[repo]
                        disable exclude from main, for a repo or for
                        everything
  --disableincludes=[repo]
                        disable includepkgs for a repo or for everything
  --obsoletes           enable obsoletes processing during updates
  --noplugins           disable Yum plugins
  --nogpgcheck          disable gpg signature checking
  --disableplugin=[plugin]
                        disable plugins by name
  --enableplugin=[plugin]
                        enable plugins by name
  --skip-broken         skip packages with depsolving problems
  --color=COLOR         control whether color is used
  --releasever=RELEASEVER
                        set value of $releasever in yum config and repo files
  --downloadonly        don't update, just download
  --downloaddir=DLDIR   specifies an alternate directory to store packages
  --setopt=SETOPTS      set arbitrary config and repo options
  --bugfix              Include bugfix relevant packages, in updates
  --security            Include security relevant packages, in updates
  --advisory=ADVS, --advisories=ADVS
                        Include packages needed to fix the given advisory, in
                        updates
  --bzs=BZS             Include packages needed to fix the given BZ, in
                        updates
  --cves=CVES           Include packages needed to fix the given CVE, in
                        updates
  --sec-severity=SEVS, --secseverity=SEVS
                        Include security relevant packages matching the
                        severity, in updates

  Plugin Options:
```
Per installare un pacchetto uso *install* e con *remove* disinstallo il pacchetto
con ciò che dipende da quel pacchetto ma non pulisce ciò che è stato installato come dipendenza.
Si ha però *yum history* che ci permette di fare il revert di qualsiasi operazione.
Vediamo degli esempi:
```shell
[osboxes@osboxes ~]$ sudo yum install zsh
[sudo] password for osboxes: 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: linuxsoft.cern.ch
 * extras: linuxsoft.cern.ch
 * updates: linuxsoft.cern.ch
Resolving Dependencies
--> Running transaction check
---> Package zsh.x86_64 0:5.0.2-31.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================
 Package        Arch              Version                    Repository       Size
===================================================================================
Installing:
 zsh            x86_64            5.0.2-31.el7               base            2.4 M

Transaction Summary
===================================================================================
Install  1 Package

Total download size: 2.4 M
Installed size: 5.6 M
Is this ok [y/d/N]: y
Downloading packages:
zsh-5.0.2-31.el7.x86_64.rpm                                 | 2.4 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : zsh-5.0.2-31.el7.x86_64                                         1/1 
  Verifying  : zsh-5.0.2-31.el7.x86_64                                         1/1 

Installed:
  zsh.x86_64 0:5.0.2-31.el7                                                        

Complete!

[osboxes@osboxes ~]$ sudo yum list \*httpd\*
[sudo] password for osboxes: 
ùLoaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: linuxsoft.cern.ch
 * extras: linuxsoft.cern.ch
 * updates: linuxsoft.cern.ch
Available Packages
httpd.x86_64                                      2.4.6-89.el7.centos       updates
httpd-devel.x86_64                                2.4.6-89.el7.centos       updates
httpd-manual.noarch                               2.4.6-89.el7.centos       updates
httpd-tools.x86_64                                2.4.6-89.el7.centos       updates
keycloak-httpd-client-install.noarch              0.6-3.el7                 base   
libmicrohttpd.i686                                0.9.33-2.el7              base   
libmicrohttpd.x86_64                              0.9.33-2.el7              base   
libmicrohttpd-devel.i686                          0.9.33-2.el7              base   
libmicrohttpd-devel.x86_64                        0.9.33-2.el7              base   
libmicrohttpd-doc.noarch                          0.9.33-2.el7              base   
python2-keycloak-httpd-client-install.noarch      0.6-3.el7                 base 

[osboxes@osboxes ~]$ sudo yum remove httpd
Loaded plugins: fastestmirror, langpacks
Resolving Dependencies
--> Running transaction check
---> Package httpd.x86_64 0:2.4.6-89.el7.centos will be erased
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================
 Package       Arch           Version                       Repository        Size
===================================================================================
Removing:
 httpd         x86_64         2.4.6-89.el7.centos           @updates         9.4 M

Transaction Summary
===================================================================================
Remove  1 Package

Installed size: 9.4 M
Is this ok [y/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : httpd-2.4.6-89.el7.centos.x86_64                                1/1 
  Verifying  : httpd-2.4.6-89.el7.centos.x86_64                                1/1 

Removed:
  httpd.x86_64 0:2.4.6-89.el7.centos                                               

Complete!

[osboxes@osboxes ~]$ sudo yum history list
Loaded plugins: fastestmirror, langpacks
ID     | Login user               | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
     5 | osboxes.org <osboxes>    | 2019-05-17 13:39 | Erase          |    1   
     4 | osboxes.org <osboxes>    | 2019-05-17 13:39 | Install        |    5   
     3 | osboxes.org <osboxes>    | 2019-05-17 13:15 | Install        |    1   
     2 | osboxes.org <osboxes>    | 2019-05-17 12:53 | I, U           |  180 EE
     1 | System <unset>           | 2019-02-11 13:11 | Install        | 1390   
history list

[osboxes@osboxes ~]$ sudo yum history info 3
Loaded plugins: fastestmirror, langpacks
Transaction ID : 3
Begin time     : Fri May 17 13:15:27 2019
Begin rpmdb    : 1391:f54c667b7d7f46e2c08560cce9ac5d1ddb2dd0f0
End time       :            13:15:28 2019 (1 seconds)
End rpmdb      : 1392:87f1bf1620bf2852239e963e71de074f11aa5695
User           : osboxes.org <osboxes>
Return-Code    : Success
Command Line   : install zsh
Transaction performed with:
    Installed     rpm-4.11.3-35.el7.x86_64                      @anaconda
    Installed     yum-3.4.3-161.el7.centos.noarch               @anaconda
    Installed     yum-plugin-fastestmirror-1.1.31-50.el7.noarch @anaconda
Packages Altered:
    Install zsh-5.0.2-31.el7.x86_64 @base
history info

[osboxes@osboxes ~]$ sudo yum history undo 3
Loaded plugins: fastestmirror, langpacks
Undoing transaction 3, from Fri May 17 13:15:27 2019
    Install zsh-5.0.2-31.el7.x86_64 @base
Resolving Dependencies
--> Running transaction check
---> Package zsh.x86_64 0:5.0.2-31.el7 will be erased
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================
 Package        Arch              Version                   Repository        Size
===================================================================================
Removing:
 zsh            x86_64            5.0.2-31.el7              @base            5.6 M

Transaction Summary
===================================================================================
Remove  1 Package

Installed size: 5.6 M
Is this ok [y/N]: 
...

osboxes@osboxes ~]$ sudo yum repolist
[sudo] password for osboxes: 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: linuxsoft.cern.ch
 * extras: linuxsoft.cern.ch
 * updates: linuxsoft.cern.ch
repo id                              repo name                               status
base/7/x86_64                        CentOS-7 - Base                         10,019
extras/7/x86_64                      CentOS-7 - Extras                          413
updates/7/x86_64                     CentOS-7 - Updates                       1,945
repolist: 12,377
```
Per aggiungere una repo aggiungo un *.repo* in */etc/yum.repo.d/*:
```shell
[aggiornamenti]
name=Aggiornamenti TEST
baseurl=http://conten.example.com/rhel7.0/x86_64/errata
enabled=1
gpgcheck=0
```
e a questo punto *yum update* aggiungerà la repo. 
*EPEL* è una repo per redhat con i software che mancano su RHEL ma presenti su fedora.

### PROCESSO DI BOOT
Al momento dell'accensione della macchina accadono varie cose:
1) si carica il BIOS, presente su una eprom della macchina, che innanzitutto definisce 
   il device da cui fare il boot. legge ed esegue i primi 512 byte, l'MBR, del volume scelto che contiene
   il bootloader e la partition table, che è indirizzata da 4 bit. MBR indirizza al massimo 2tb,
   o meglio, sfruttando i *binary*, 2 *tebibyte*. Ora si ha *GPT* che indirizza migliaia di volte la quantità di MBR.
   Il bootloader è un codice software che serve a sapere dove si trova il kernel e a farlo eseguire.
   In RHEL si ha *grub2* come bootloader. I config di grub sono dentro boot:
   ```shell
   [osboxes@osboxes ~]$ sudo ls /boot/grub2
   [sudo] password for osboxes: 
   device.map  fonts  grub.cfg  grubenv  i386-pc  locale
   ```
   Il kernel riceve come parametro la root directory, e vari pezzi di codice per i vari pezzi hardware.
   Per ovviare al problema di avere parti inutili caricate si hanno i moduli che vengono caricati dinamicamente che si trovano in */lib/modules*
   Anche i file system sono moduli del kernel (redhat us *xfs*).
   Poi il kernel monta la root e per farlo necessita il modulo del file system, che si trova in */lib/modules* che però devo ancora montare,
   quindi, per accedere a quei moduli, passo l'environment mediante *initramfs*
   che si crea all'installazione del kernel che contiene i moduli per caricare la root.
   
2) Si esegue, o meglio si eseguiva, il processo numero 1, l'*init*. L'init esegue le classi 
dei servizi attivi, le *runlevel*. A questo punto la macchina ha completato il boot. 
  I tempi sono cambiati, ora si ha *systemd*, che racchiude init e runlevel, come si vede qui:
  ```shell
  [osboxes@osboxes ~]$ pstree
  systemd─┬─ModemManager───2*[{ModemManager}]
        ├─NetworkManager─┬─2*[dhclient]
        │                └─2*[{NetworkManager}]
        ├─2*[abrt-watch-log]
        ├─abrtd
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─alsactl
        ├─at-spi-bus-laun─┬─dbus-daemon───{dbus-daemon}
        │                 └─3*[{at-spi-bus-laun}]
        ├─at-spi2-registr───2*[{at-spi2-registr}]
        ├─atd
        ├─auditd─┬─audispd─┬─sedispatch
        │        │         └─{audispd}
        │        └─{auditd}
        ├─avahi-daemon───avahi-daemon
        ├─boltd───2*[{boltd}]
        ├─chronyd
        ├─colord───2*[{colord}]
        ├─crond
        ├─cupsd
        ├─2*[dbus-daemon───{dbus-daemon}]
        ├─dbus-launch
        ├─dconf-service───2*[{dconf-service}]
        ├─dnsmasq───dnsmasq
        ├─evolution-addre─┬─evolution-addre───5*[{evolution-addre}]
        │                 └─4*[{evolution-addre}]
        ├─evolution-calen─┬─evolution-calen───8*[{evolution-calen}]
        │                 └─4*[{evolution-calen}]
        ├─evolution-sourc───3*[{evolution-sourc}]
        ├─firewalld───{firewalld}
        ├─fwupd───4*[{fwupd}]
        ├─gdm─┬─X───4*[{X}]
        │     ├─gdm-session-wor─┬─gnome-session-b─┬─abrt-applet───2*[{abrt-applet}+
        │     │                 │                 ├─gnome-shell─┬─ibus-daemon─┬─ib+
        │     │                 │                 │             │             ├─ib+
        │     │                 │                 │             │             └─2*+
        │     │                 │                 │             └─17*[{gnome-shell+
        │     │                 │                 ├─gnome-software───3*[{gnome-sof+
        │     │                 │                 ├─gsd-a11y-settin───3*[{gsd-a11y+
        │     │                 │                 ├─gsd-account───3*[{gsd-account}+
        │     │                 │                 ├─gsd-clipboard───2*[{gsd-clipbo+
        │     │                 │                 ├─gsd-color───3*[{gsd-color}]
        │     │                 │                 ├─gsd-datetime───3*[{gsd-datetim+
        │     │                 │                 ├─gsd-disk-utilit───2*[{gsd-disk+
        │     │                 │                 ├─gsd-housekeepin───3*[{gsd-hous+
        │     │                 │                 ├─gsd-keyboard───3*[{gsd-keyboar+
        │     │                 │                 ├─gsd-media-keys───3*[{gsd-media+
        │     │                 │                 ├─gsd-mouse───3*[{gsd-mouse}]
        │     │                 │                 ├─gsd-power───4*[{gsd-power}]
        │     │                 │                 ├─gsd-print-notif───2*[{gsd-prin+
        │     │                 │                 ├─gsd-rfkill───2*[{gsd-rfkill}]
        │     │                 │                 ├─gsd-screensaver───2*[{gsd-scre+
        │     │                 │                 ├─gsd-sharing───3*[{gsd-sharing}+
        │     │                 │                 ├─gsd-smartcard───4*[{gsd-smartc+
        │     │                 │                 ├─gsd-sound───3*[{gsd-sound}]
        │     │                 │                 ├─gsd-wacom───2*[{gsd-wacom}]
        │     │                 │                 ├─gsd-xsettings───3*[{gsd-xsetti+
        │     │                 │                 ├─nautilus-deskto───3*[{nautilus+
        │     │                 │                 ├─seapplet
        │     │                 │                 ├─ssh-agent
        │     │                 │                 ├─tracker-extract───13*[{tracker+
        │     │                 │                 ├─tracker-miner-a───3*[{tracker-+
        │     │                 │                 ├─tracker-miner-f───3*[{tracker-+
        │     │                 │                 ├─tracker-miner-u───3*[{tracker-+
        │     │                 │                 └─3*[{gnome-session-b}]
        │     │                 └─2*[{gdm-session-wor}]
        │     └─3*[{gdm}]
        ├─gnome-keyring-d───3*[{gnome-keyring-d}]
        ├─gnome-shell-cal───5*[{gnome-shell-cal}]
        ├─goa-daemon───4*[{goa-daemon}]
        ├─goa-identity-se───3*[{goa-identity-se}]
        ├─gsd-printer───2*[{gsd-printer}]
        ├─gssproxy───5*[{gssproxy}]
        ├─gvfs-afc-volume───3*[{gvfs-afc-volume}]
        ├─gvfs-goa-volume───2*[{gvfs-goa-volume}]
        ├─gvfs-gphoto2-vo───2*[{gvfs-gphoto2-vo}]
        ├─gvfs-mtp-volume───2*[{gvfs-mtp-volume}]
        ├─gvfs-udisks2-vo───2*[{gvfs-udisks2-vo}]
        ├─gvfsd─┬─gvfsd-trash───2*[{gvfsd-trash}]
        │       └─2*[{gvfsd}]
        ├─gvfsd-fuse───5*[{gvfsd-fuse}]
        ├─gvfsd-metadata───2*[{gvfsd-metadata}]
        ├─ibus-daemon─┬─ibus-dconf───3*[{ibus-dconf}]
        │             └─2*[{ibus-daemon}]
        ├─ibus-portal───2*[{ibus-portal}]
        ├─2*[ibus-x11───2*[{ibus-x11}]]
        ├─irqbalance
        ├─ksmtuned───sleep
        ├─libvirtd───16*[{libvirtd}]
        ├─lsmd
        ├─lvmetad
        ├─master─┬─pickup
        │        └─qmgr
        ├─mission-control───3*[{mission-control}]
        ├─packagekitd───2*[{packagekitd}]
        ├─polkitd───6*[{polkitd}]
        ├─pulseaudio───2*[{pulseaudio}]
        ├─rngd
        ├─rpcbind
        ├─rsyslogd───2*[{rsyslogd}]
        ├─rtkit-daemon───2*[{rtkit-daemon}]
        ├─smartd
        ├─sshd───sshd───sshd───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        ├─tracker-store───7*[{tracker-store}]
        ├─tuned───4*[{tuned}]
        ├─udisksd───4*[{udisksd}]
        ├─upowerd───2*[{upowerd}]
        ├─wpa_supplicant
        └─xdg-permission-───2*[{xdg-permission-}]
  ```
  systemd viene controllato da *systemctl*. Se una macchina non completa il boot si ferma in uno di questi passaggi.
  Vediamo le unit in uso di systemd, filtrando i servizi:
  ```shell
  [osboxes@osboxes ~]$ systemctl list-units --type=service
UNIT                           LOAD   ACTIVE SUB     DESCRIPTION
abrt-ccpp.service              loaded active exited  Install ABRT coredump hook
abrt-oops.service              loaded active running ABRT kernel log watcher
abrt-xorg.service              loaded active running ABRT Xorg log watcher
abrtd.service                  loaded active running ABRT Automated Bug Reporting T
accounts-daemon.service        loaded active running Accounts Service
alsa-state.service             loaded active running Manage Sound Card State (resto
atd.service                    loaded active running Job spooling tools
auditd.service                 loaded active running Security Auditing Service
avahi-daemon.service           loaded active running Avahi mDNS/DNS-SD Stack
blk-availability.service       loaded active exited  Availability of block devices
bolt.service                   loaded active running Thunderbolt system service
chronyd.service                loaded active running NTP client/server
colord.service                 loaded active running Manage, Install and Generate C
crond.service                  loaded active running Command Scheduler
cups.service                   loaded active running CUPS Printing Service
dbus.service                   loaded active running D-Bus System Message Bus
firewalld.service              loaded active running firewalld - dynamic firewall d
fwupd.service                  loaded active running Firmware update daemon
gdm.service                    loaded active running GNOME Display Manager
gssproxy.service               loaded active running GSSAPI Proxy Daemon
irqbalance.service             loaded active running irqbalance daemon
iscsi-shutdown.service         loaded active exited  Logout off all iSCSI sessions 
kdump.service                  loaded active exited  Crash recovery kernel arming
kmod-static-nodes.service      loaded active exited  Create list of required static
ksm.service                    loaded active exited  Kernel Samepage Merging
ksmtuned.service               loaded active running Kernel Samepage Merging (KSM) 
libstoragemgmt.service         loaded active running libstoragemgmt plug-in server 
libvirtd.service               loaded active running Virtualization daemon
lvm2-lvmetad.service           loaded active running LVM2 metadata daemon
lvm2-monitor.service           loaded active exited  Monitoring of LVM2 mirrors, sn
ModemManager.service           loaded active running Modem Manager
network.service                loaded active exited  LSB: Bring up/down networking
NetworkManager-wait-online.service loaded active exited  Network Manager Wait Onlin
NetworkManager.service         loaded active running Network Manager
packagekit.service             loaded active running PackageKit Daemon
polkit.service                 loaded active running Authorization Manager
...
  ```
Per far partire un servizio uso *systemctl start servizio.service* e ne vedo
lo stato con *systemctl status servizio.service*, per abilitare all'avio *systemctl enable servizio.service*,
per disabilitare *systemctl disable servizio.service* (questi ultimi agiscono mediante symlink), riavviarlo con *systemctl restart servizio.service* e il reload con *systemctl reload servizio.service*
dove, nel reload, il servizio non cade ma rilegge la configurazione (mentre il restart lo riavvia completamente)

I target sono i set di servizi che devono essere attivi in una particolare configurazione:
```shell
[osboxes@osboxes ~]$ systemctl list-units --type=target
UNIT                   LOAD   ACTIVE SUB    DESCRIPTION
basic.target           loaded active active Basic System
cryptsetup.target      loaded active active Local Encrypted Volumes
getty-pre.target       loaded active active Login Prompts (Pre)
getty.target           loaded active active Login Prompts
graphical.target       loaded active active Graphical Interface
local-fs-pre.target    loaded active active Local File Systems (Pre)
local-fs.target        loaded active active Local File Systems
multi-user.target      loaded active active Multi-User System
network-online.target  loaded active active Network is Online
network-pre.target     loaded active active Network (Pre)
network.target         loaded active active Network
nfs-client.target      loaded active active NFS client services
nss-user-lookup.target loaded active active User and Group Name Lookups
paths.target           loaded active active Paths
remote-fs-pre.target   loaded active active Remote File Systems (Pre)
remote-fs.target       loaded active active Remote File Systems
rpc_pipefs.target      loaded active active rpc_pipefs.target
rpcbind.target         loaded active active RPC Port Mapper
slices.target          loaded active active Slices
sockets.target         loaded active active Sockets
sound.target           loaded active active Sound Card
swap.target            loaded active active Swap
sysinit.target         loaded active active System Initialization
timers.target          loaded active active Timers

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

24 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```
e noi ci troviamo in quello gerarchicamente superiore.

Se al boot non trovo il device posso richiedere a grub di acricare l'unità emergeny per entrare in una sorta di shell d'emergenza.
**aggiungere parte di troubleshooting**

