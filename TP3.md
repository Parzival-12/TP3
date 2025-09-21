sur la machine kvm2:

```
[root@localhost etc]# ping frontend.one
PING frontend.one (10.3.1.10) 56(84) bytes of data.
64 bytes from frontend.one (10.3.1.10): icmp_seq=1 ttl=64 time=3.80 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=2 ttl=64 time=1.43 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=3 ttl=64 time=1.53 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=4 ttl=64 time=0.622 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=5 ttl=64 time=0.483 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=6 ttl=64 time=0.545 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=7 ttl=64 time=0.461 ms
^C
--- frontend.one ping statistics ---
7 packets transmitted, 7 received, 0% packet loss, time 6124ms
rtt min/avg/max/mdev = 0.461/1.267/3.797/1.114 ms
```

sur la machine kvm1:

``` 
[root@kvm1 ~]# ping -c 5 frontend.one
PING frontend.one (166.117.110.61) 56(84) bytes of data.
64 bytes from 166.117.110.61 (166.117.110.61): icmp_seq=1 ttl=128 time=173 ms
64 bytes from 166.117.110.61 (166.117.110.61): icmp_seq=2 ttl=128 time=50.4 ms
64 bytes from 166.117.110.61 (166.117.110.61): icmp_seq=3 ttl=128 time=65.3 ms
64 bytes from 166.117.110.61 (166.117.110.61): icmp_seq=4 ttl=128 time=102 ms
64 bytes from 166.117.110.61 (166.117.110.61): icmp_seq=5 ttl=128 time=62.8 ms

--- frontend.one ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4008ms
rtt min/avg/max/mdev = 50.385/90.581/172.627/44.474 ms
```

A. Database¶
🌞 Installer un serveur MySQL

on va installer une version spécifique de MySQL, demandée par OpenNebula
ajouter un dépôt :

télécharger le RPM disponible à l'URL suivante : 
```
[root@localhost ~]# wget https://dev.mysql.com/get/mysql80-community-release-el9-5.noarch.rpm
--2025-09-16 16:22:45--  https://dev.mysql.com/get/mysql80-community-release-el9-5.noarch.rpm
Resolving dev.mysql.com (dev.mysql.com)... 23.54.143.15, 2a02:26f0:680:48c::2e31, 2a02:26f0:680:48b::2e31
Connecting to dev.mysql.com (dev.mysql.com)|23.54.143.15|:443... connected.
HTTP request sent, awaiting response... 302 Moved Temporarily
Location: https://repo.mysql.com//mysql80-community-release-el9-5.noarch.rpm [following]
--2025-09-16 16:22:53--  https://repo.mysql.com//mysql80-community-release-el9-5.noarch.rpm
Resolving repo.mysql.com (repo.mysql.com)... 2.18.132.71, 2a02:26f0:9100:184::1d68, 2a02:26f0:9100:196::1d68
Connecting to repo.mysql.com (repo.mysql.com)|2.18.132.71|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13319 (13K) [application/x-redhat-package-manager]
Saving to: ‘mysql80-community-release-el9-5.noarch.rpm’

mysql80-community-release-el9 100%[=================================================>]  13.01K  --.-KB/s    in 0.001s

2025-09-16 16:22:54 (11.8 MB/s) - ‘mysql80-community-release-el9-5.noarch.rpm’ saved [13319/13319]
```

puis installez-le localement avec une commande rpm

``` 
[root@localhost ~]# sudo rpm -ivh mysql80-community-release-el9-5.noarch.rpm
warning: mysql80-community-release-el9-5.noarch.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql80-community-release-el9-5  ################################# [100%]
```

installer le paquet qui contient le serveur MySQL depuis ce nouveau dépôt :

faites un dnf search mysql pour voir les paquets dispos
```
[root@localhost ~]# sudo dnf search mysql
MySQL 8.0 Community Server                                                              774 kB/s | 2.7 MB     00:03
MySQL Connectors Community                                                               87 kB/s |  90 kB     00:01
MySQL Tools Community                                                                   183 kB/s | 1.2 MB     00:06
============================================ Name & Summary Matched: mysql =============================================mysql.x86_64 : MySQL client programs and shared libraries
apr-util-mysql.x86_64 : APR utility library MySQL DBD driver
dovecot-mysql.x86_64 : MySQL back end for dovecot
mysql-common.x86_64 : The shared files required for MySQL server and client
mysql-community-client.x86_64 : MySQL database client applications and tools
mysql-community-client-plugins.x86_64 : Shared plugins for MySQL client applications
mysql-community-common.x86_64 : MySQL database common files for server and client libs
mysql-community-debugsource.x86_64 : Debug sources for package mysql-community
mysql-community-devel.x86_64 : Development header files and libraries for MySQL database client applications
mysql-community-icu-data-files.x86_64 : MySQL packaging of ICU data files
mysql-community-libs.x86_64 : Shared libraries for MySQL database client applications
mysql-community-server-debug.x86_64 : The debug version of MySQL server
mysql-community-test.x86_64 : Test suite for the MySQL database server
mysql-connector-c++.x86_64 : MySQL database connector for C++
mysql-connector-c++-compat.x86_64 : MySQL Connector/C++ -- backward compatibility libraries
mysql-connector-c++-debugsource.x86_64 : Debug sources for package mysql-connector-c++
mysql-connector-c++-devel.x86_64 : Development header files and libraries for MySQL C++ client applications
mysql-connector-c++-jdbc.x86_64 : MySQL Driver for C++ which mimics the JDBC 4.0 API
mysql-connector-j.noarch : Standardized MySQL database driver for Java
mysql-connector-odbc.x86_64 : An ODBC 9.4 driver for MySQL - driver package
mysql-connector-odbc-debugsource.x86_64 : Debug sources for package mysql-connector-odbc
mysql-connector-odbc-setup.x86_64 : An ODBC 9.4 driver for MySQL - setup library
mysql-connector-python3.x86_64 : Standardized MySQL database driver for Python 3
mysql-errmsg.x86_64 : The error messages files required by MySQL server
mysql-ref-manual-8.0-en-html-chapter.noarch : The MySQL Reference Manual (HTML, English)
mysql-ref-manual-8.0-en-pdf.noarch : The MySQL Reference Manual (PDF, English)
mysql-router-community.x86_64 : MySQL Router
mysql-selinux.noarch : SELinux policy modules for MySQL and MariaDB packages
mysql-server.x86_64 : The MySQL server and related files
mysql-shell.x86_64 : Command line shell and scripting environment for MySQL
mysql-shell-debugsource.x86_64 : Debug sources for package mysql-shell
mysql-workbench-community.x86_64 : A MySQL visual database modeling, administration, development and migration tool
mysql80-community-release.noarch : MySQL repository configuration for yum
mysqlx-connector-python3.x86_64 : Standardized MySQL database driver for Python 3
pcp-pmda-mysql.x86_64 : Performance Co-Pilot (PCP) metrics for MySQL
perl-DBD-MySQL.x86_64 : A MySQL interface for Perl
php-mysqlnd.x86_64 : A module for PHP applications that use MySQL databases
postfix-mysql.x86_64 : Postfix MySQL map support
python3-PyMySQL.noarch : Pure-Python MySQL client library
python3.11-PyMySQL.noarch : Pure-Python MySQL client library
python3.11-PyMySQL+rsa.noarch : Metapackage for python3.11-PyMySQL: rsa extras
python3.12-PyMySQL.noarch : Pure-Python MySQL client library
python3.12-PyMySQL+rsa.noarch : Metapackage for python3.12-PyMySQL: rsa extras
qt5-qtbase-mysql.x86_64 : MySQL driver for Qt5's SQL classes
qt5-qtbase-mysql.i686 : MySQL driver for Qt5's SQL classes
rsyslog-mysql.x86_64 : MySQL support for rsyslog
rubygem-mysql2.x86_64 : A simple, fast Mysql library for Ruby, binding to libmysql
================================================= Name Matched: mysql ==================================================mysql-community-server.x86_64 : A very fast and reliable SQL database server
================================================ Summary Matched: mysql ================================================mariadb-java-client.noarch : Connects applications developed in Java to MariaDB and MySQL databases
mariadb-server-utils.x86_64 : Non-essential server utilities for MariaDB/MySQL applications
perl-DBD-MariaDB.x86_64 : MariaDB and MySQL driver for the Perl5 Database Interface (DBI)
```

vous devriez voir un paquet mysql-community-server dispo
installez-le
```
[root@localhost ~]# sudo dnf install mysql-community-server
Last metadata expiration check: 0:01:43 ago on Tue Sep 16 17:27:43 2025.
Dependencies resolved.
=======================================================================================================================
 Package                                  Architecture     Version                   Repository                   Size
=======================================================================================================================
Installing:
 mysql-community-server                   x86_64           8.0.43-1.el9              mysql80-community            50 M
Installing dependencies:
 mysql-community-client                   x86_64           8.0.43-1.el9              mysql80-community           3.3 M
 mysql-community-client-plugins           x86_64           8.0.43-1.el9              mysql80-community           1.4 M
 mysql-community-common                   x86_64           8.0.43-1.el9              mysql80-community           556 k
 mysql-community-icu-data-files           x86_64           8.0.43-1.el9              mysql80-community           2.3 M
 mysql-community-libs                     x86_64           8.0.43-1.el9              mysql80-community           1.5 M

Transaction Summary
=======================================================================================================================
Install  6 Packages

Total download size: 59 M
Installed size: 337 M
Is this ok [y/N]: y
Downloading Packages:
(1/6): mysql-community-common-8.0.43-1.el9.x86_64.rpm                                  104 kB/s | 556 kB     00:05
(2/6): mysql-community-client-plugins-8.0.43-1.el9.x86_64.rpm                           71 kB/s | 1.4 MB     00:19
(3/6): mysql-community-icu-data-files-8.0.43-1.el9.x86_64.rpm                           75 kB/s | 2.3 MB     00:31
(4/6): mysql-community-client-8.0.43-1.el9.x86_64.rpm                                   91 kB/s | 3.3 MB     00:37
(5/6): mysql-community-libs-8.0.43-1.el9.x86_64.rpm                                     78 kB/s | 1.5 MB     00:19
(6/6): mysql-community-server-8.0.43-1.el9.x86_64.rpm                                  933 kB/s |  50 MB     00:54
-----------------------------------------------------------------------------------------------------------------------
Total                                                                                  658 kB/s |  59 MB     01:31
MySQL 8.0 Community Server                                                             1.8 MB/s | 3.1 kB     00:00
Importing GPG key 0xA8D3785C:
 Userid     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 Fingerprint: BCA4 3417 C3B4 85DD 128E C6D4 B7B3 B788 A8D3 785C
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2023
Is this ok [y/N]: y
Key imported successfully
MySQL 8.0 Community Server                                                             3.0 MB/s | 3.1 kB     00:00
Importing GPG key 0x3A79BD29:
 Userid     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 Fingerprint: 859B E8D7 C586 F538 430B 19C2 467B 942D 3A79 BD29
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022
Is this ok [y/N]: y
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                               1/1
  Installing       : mysql-community-common-8.0.43-1.el9.x86_64                                                    1/6
  Installing       : mysql-community-client-plugins-8.0.43-1.el9.x86_64                                            2/6
  Installing       : mysql-community-libs-8.0.43-1.el9.x86_64                                                      3/6
  Running scriptlet: mysql-community-libs-8.0.43-1.el9.x86_64                                                      3/6
  Installing       : mysql-community-client-8.0.43-1.el9.x86_64                                                    4/6
  Installing       : mysql-community-icu-data-files-8.0.43-1.el9.x86_64                                            5/6
  Running scriptlet: mysql-community-server-8.0.43-1.el9.x86_64                                                    6/6
  Installing       : mysql-community-server-8.0.43-1.el9.x86_64                                                    6/6
  Running scriptlet: mysql-community-server-8.0.43-1.el9.x86_64                                                    6/6
  Verifying        : mysql-community-client-8.0.43-1.el9.x86_64                                                    1/6
  Verifying        : mysql-community-client-plugins-8.0.43-1.el9.x86_64                                            2/6
  Verifying        : mysql-community-common-8.0.43-1.el9.x86_64                                                    3/6
  Verifying        : mysql-community-icu-data-files-8.0.43-1.el9.x86_64                                            4/6
  Verifying        : mysql-community-libs-8.0.43-1.el9.x86_64                                                      5/6
  Verifying        : mysql-community-server-8.0.43-1.el9.x86_64                                                    6/6

Installed:
  mysql-community-client-8.0.43-1.el9.x86_64             mysql-community-client-plugins-8.0.43-1.el9.x86_64
  mysql-community-common-8.0.43-1.el9.x86_64             mysql-community-icu-data-files-8.0.43-1.el9.x86_64
  mysql-community-libs-8.0.43-1.el9.x86_64               mysql-community-server-8.0.43-1.el9.x86_64

Complete!
[root@localhost ~]#
```


🌞 Démarrer le serveur MySQL

démarrer le service mysqld
```
[root@localhost ~]# sudo systemctl start mysqld
[root@localhost ~]# sudo systemctl status mysqld
● mysqld.service - MySQL Server
     Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; preset: disabled)
     Active: active (running) since Tue 2025-09-16 17:41:39 CEST; 31s ago
       Docs: man:mysqld(8)
             http://dev.mysql.com/doc/refman/en/using-systemd.html
    Process: 2883 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
   Main PID: 2954 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 10850)
     Memory: 480.5M
        CPU: 13.358s
     CGroup: /system.slice/mysqld.service
             └─2954 /usr/sbin/mysqld

Sep 16 17:41:23 frontend.one systemd[1]: Starting MySQL Server...
Sep 16 17:41:39 frontend.one systemd[1]: Started MySQL Server.
[root@localhost ~]#
```
activer ce service au démarrage
```
[root@localhost ~]# sudo systemctl enable mysqld
[root@localhost ~]# systemctl is-enabled mysqld
enabled

```
🌞 Setup MySQL
un mot de passe temporaire pour l'utilisateur root de la base de données a été généré,
il est visible dans /var/log/mysqld.log
```
[root@frontend ~]# sudo grep 'temporary password' /var/log/mysqld.log
2025-09-16T15:41:32.598089Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: ?hY89L4,tgnS
[root@frontend ~]#
```

🌞 Installer OpenNebula

installez les paquets opennebula, opennebula-sunstone, opennebula-fireedge
```
[root@frontend ~]# sudo dnf install opennebula opennebula-sunstone opennebula-fireedge
Last metadata expiration check: 0:00:24 ago on Tue Sep 16 18:51:41 2025.
Dependencies resolved.
=========================================================================================================================================================================
 Package                                       Architecture               Version                                                   Repository                      Size
=========================================================================================================================================================================
Installing:
 opennebula                                    x86_64                     6.10.0.1-1.el9                                            opennebula                      10 M
 opennebula-fireedge                           x86_64                     6.10.0.1-1.el9                                            opennebula                      46 M
 opennebula-sunstone                           noarch                     6.10.0.1-1.el9                                            opennebula                      29 M
Installing dependencies:
 alsa-lib                                      x86_64                     1.2.13-2.el9                                              appstream                      506 k
 augeas-libs                                   x86_64                     1.14.1-2.el9                                              appstream                      373 k
 cups-libs                                     x86_64                     1:2.3.3op2-33.el9                                         baseos                         261 k
 flac-libs                                     x86_64                     1.3.3-10.el9_2.1                                          appstream                      217 k
 flexiblas                                     x86_64                     3.0.4-8.el9.0.1                                           appstream                       30 k
 flexiblas-netlib                              x86_64                     3.0.4-8.el9.0.1                                           appstream                      3.0 M
 flexiblas-openblas-openmp                     x86_64                     3.0.4-8.el9.0.1                                           appstream                       15 k
 freerdp-libs                                  x86_64                     2:2.11.7-1.el9                                            appstream                      900 k
 gd                                            x86_64                     2.3.2-3.el9                                               appstream                      131 k
 genisoimage                                   x86_64                     1.1.11-48.el9                                             epel                           324 k
 git                                           x86_64                     2.47.3-1.el9_6                                            appstream                       50 k
 git-core                                      x86_64                     2.47.3-1.el9_6                                            appstream                      4.6 M
 git-core-doc                                  noarch                     2.47.3-1.el9_6                                            appstream                      2.8 M
 glx-utils                                     x86_64                     8.4.0-12.20210504git0f9e7d9.el9.0.1                       appstream                       40 k
 gnuplot-common                                x86_64                     5.4.3-2.el9                                               epel                           776 k
 gsm                                           x86_64                     1.0.19-6.el9                                              appstream                       33 k
 jbigkit-libs                                  x86_64                     2.1-23.el9                                                appstream                       52 k
 keyutils-libs-devel                           x86_64                     1.6.3-1.el9                                               appstream                       54 k
 krb5-devel                                    x86_64                     1.21.1-8.el9_6                                            appstream                      132 k
 lame-libs                                     x86_64                     3.100-12.el9                                              appstream                      332 k
 libICE                                        x86_64                     1.0.10-8.el9                                              appstream                       70 k
 libSM                                         x86_64                     1.2.3-10.el9                                              appstream                       41 k
 libX11-xcb                                    x86_64                     1.7.0-11.el9                                              appstream                       10 k
 libXfixes                                     x86_64                     5.0.3-16.el9                                              appstream                       19 k
 libXpm                                        x86_64                     3.5.13-10.el9                                             appstream                       58 k
 libXxf86vm                                    x86_64                     1.1.4-18.el9                                              appstream                       18 k
 libasyncns                                    x86_64                     0.8-22.el9                                                appstream                       29 k
 libcerf                                       x86_64                     1.17-2.el9                                                epel                            38 k
 libcom_err-devel                              x86_64                     1.46.5-7.el9                                              appstream                       15 k
 libdrm                                        x86_64                     2.4.123-2.el9                                             appstream                      158 k
 libevdev                                      x86_64                     1.11.0-3.el9                                              appstream                       45 k
 libgfortran                                   x86_64                     11.5.0-5.el9_5                                            baseos                         797 k
 libglvnd                                      x86_64                     1:1.3.4-1.el9                                             appstream                      133 k
 libglvnd-egl                                  x86_64                     1:1.3.4-1.el9                                             appstream                       36 k
 libglvnd-glx                                  x86_64                     1:1.3.4-1.el9                                             appstream                      140 k
 libinput                                      x86_64                     1.19.3-5.el9_6                                            appstream                      193 k
 libkadm5                                      x86_64                     1.21.1-8.el9_6                                            baseos                          75 k
 libogg                                        x86_64                     2:1.3.4-6.el9                                             appstream                       32 k
 libpciaccess                                  x86_64                     0.16-7.el9                                                baseos                          26 k
 libquadmath                                   x86_64                     11.5.0-5.el9_5                                            baseos                         187 k
 libselinux-devel                              x86_64                     3.6-3.el9                                                 appstream                      113 k
 libsepol-devel                                x86_64                     3.6-2.el9                                                 appstream                       39 k
 libsndfile                                    x86_64                     1.0.31-9.el9                                              appstream                      205 k
 libsodium                                     x86_64                     1.0.18-8.el9                                              epel                           161 k
 libsodium-devel                               x86_64                     1.0.18-8.el9                                              epel                           1.0 M
 libssh2                                       x86_64                     1.11.0-1.el9                                              epel                           132 k
 libtiff                                       x86_64                     4.4.0-13.el9                                              appstream                      197 k
 libunwind                                     x86_64                     1.6.2-1.el9                                               epel                            67 k
 libunwind-devel                               x86_64                     1.6.2-1.el9                                               epel                            80 k
 liburing                                      x86_64                     2.5-1.el9                                                 appstream                       38 k
 libusal                                       x86_64                     1.1.11-48.el9                                             epel                           137 k
 libverto-devel                                x86_64                     0.3.2-3.el9                                               appstream                       14 k
 libvncserver                                  x86_64                     0.9.13-11.el9                                             epel                           296 k
 libvorbis                                     x86_64                     1:1.3.7-5.el9                                             appstream                      192 k
 libwacom                                      x86_64                     1.12.1-3.el9_4                                            appstream                       43 k
 libwacom-data                                 noarch                     1.12.1-3.el9_4                                            appstream                      105 k
 libwayland-client                             x86_64                     1.21.0-1.el9                                              appstream                       33 k
 libwayland-cursor                             x86_64                     1.21.0-1.el9                                              appstream                       18 k
 libwayland-server                             x86_64                     1.21.0-1.el9                                              appstream                       41 k
 libwebp                                       x86_64                     1.2.0-8.el9                                               appstream                      276 k
 libwinpr                                      x86_64                     2:2.11.7-1.el9                                            appstream                      356 k
 libxkbcommon                                  x86_64                     1.0.3-4.el9                                               appstream                      132 k
 libxkbcommon-x11                              x86_64                     1.0.3-4.el9                                               appstream                       21 k
 libxkbfile                                    x86_64                     1.1.0-8.el9                                               appstream                       88 k
 libxshmfence                                  x86_64                     1.3-10.el9                                                appstream                       12 k
 libxslt                                       x86_64                     1.1.34-13.el9_6                                           appstream                      239 k
 mesa-dri-drivers                              x86_64                     24.2.8-2.el9_6                                            appstream                      9.4 M
 mesa-filesystem                               x86_64                     24.2.8-2.el9_6                                            appstream                       11 k
 mesa-libEGL                                   x86_64                     24.2.8-2.el9_6                                            appstream                      141 k
 mesa-libGL                                    x86_64                     24.2.8-2.el9_6                                            appstream                      169 k
 mesa-libgbm                                   x86_64                     24.2.8-2.el9_6                                            appstream                       36 k
 mesa-libglapi                                 x86_64                     24.2.8-2.el9_6                                            appstream                       44 k
 mtdev                                         x86_64                     1.1.5-22.el9                                              appstream                       21 k
 nodejs                                        x86_64                     1:16.20.2-8.el9_4                                         appstream                      111 k
 nodejs-libs                                   x86_64                     1:16.20.2-8.el9_4                                         appstream                       15 M
 openblas                                      x86_64                     0.3.26-2.el9                                              appstream                       37 k
 openblas-openmp                               x86_64                     0.3.26-2.el9                                              appstream                      4.9 M
 opennebula-common                             noarch                     6.10.0.1-1.el9                                            opennebula                      23 k
 opennebula-common-onecfg                      noarch                     6.10.0.1-1.el9                                            opennebula                     9.1 k
 opennebula-libs                               noarch                     6.10.0.1-1.el9                                            opennebula                     202 k
 opennebula-provision-data                     noarch                     6.10.0.1-1.el9                                            opennebula                      52 k
 opennebula-rubygems                           x86_64                     6.10.0.1-1.el9                                            opennebula                      16 M
 opennebula-tools                              noarch                     6.10.0.1-1.el9                                            opennebula                     191 k
 openpgm                                       x86_64                     5.2.122-28.el9                                            epel                           176 k
 openpgm-devel                                 x86_64                     5.2.122-28.el9                                            epel                            58 k
 opus                                          x86_64                     1.3.1-10.el9                                              appstream                      199 k
 patch                                         x86_64                     2.7.6-16.el9                                              appstream                      127 k
 pcre2-devel                                   x86_64                     10.40-6.el9                                               appstream                      471 k
 pcre2-utf16                                   x86_64                     10.40-6.el9                                               appstream                      213 k
 pcre2-utf32                                   x86_64                     10.40-6.el9                                               appstream                      202 k
 perl-DynaLoader                               x86_64                     1.47-481.1.el9_6                                          appstream                       24 k
 perl-Error                                    noarch                     1:0.17029-7.el9                                           appstream                       41 k
 perl-File-Find                                noarch                     1.37-481.1.el9_6                                          appstream                       24 k
 perl-Git                                      noarch                     2.47.3-1.el9_6                                            appstream                       37 k
 perl-TermReadKey                              x86_64                     2.38-11.el9                                               appstream                       36 k
 perl-lib                                      x86_64                     0.65-481.1.el9_6                                          appstream                       13 k
 pulseaudio-libs                               x86_64                     15.0-3.el9                                                appstream                      663 k
 python3-numpy                                 x86_64                     1:1.23.5-1.el9                                            appstream                      5.8 M
 qemu-img                                      x86_64                     17:9.1.0-15.el9_6.7                                       appstream                      2.5 M
 qt5-qtbase                                    x86_64                     5.15.9-11.el9_6                                           appstream                      3.5 M
 qt5-qtbase-common                             noarch                     5.15.9-11.el9_6                                           appstream                      7.6 k
 qt5-qtbase-gui                                x86_64                     5.15.9-11.el9_6                                           appstream                      6.3 M
 qt5-qtsvg                                     x86_64                     5.15.9-2.el9                                              appstream                      184 k
 ruby                                          x86_64                     3.0.7-165.el9_5                                           appstream                       38 k
 ruby-libs                                     x86_64                     3.0.7-165.el9_5                                           appstream                      3.2 M
 rubygem-bigdecimal                            x86_64                     3.0.0-165.el9_5                                           appstream                       52 k
 rubygem-io-console                            x86_64                     0.5.7-165.el9_5                                           appstream                       23 k
 rubygem-json                                  x86_64                     2.5.1-165.el9_5                                           appstream                       51 k
 rubygem-psych                                 x86_64                     3.3.2-165.el9_5                                           appstream                       48 k
 rubygem-rexml                                 noarch                     3.2.5-165.el9_5                                           appstream                       92 k
 rubygems                                      noarch                     3.2.33-165.el9_5                                          appstream                      253 k
 sqlite                                        x86_64                     3.34.1-8.el9_6                                            appstream                      746 k
 uuid                                          x86_64                     1.6.2-55.el9                                              appstream                       56 k
 xcb-util                                      x86_64                     0.4.0-19.el9                                              appstream                       18 k
 xcb-util-image                                x86_64                     0.4.0-19.el9.0.1                                          appstream                       18 k
 xcb-util-keysyms                              x86_64                     0.4.0-17.el9                                              appstream                       14 k
 xcb-util-renderutil                           x86_64                     0.3.9-20.el9.0.1                                          appstream                       16 k
 xcb-util-wm                                   x86_64                     0.4.1-22.el9                                              appstream                       31 k
 xkeyboard-config                              noarch                     2.33-2.el9                                                appstream                      779 k
 xmlrpc-c                                      x86_64                     1.51.0-16.el9                                             appstream                      140 k
 zeromq                                        x86_64                     4.3.4-2.el9                                               epel                           431 k
 zeromq-devel                                  x86_64                     4.3.4-2.el9                                               epel                            16 k
Installing weak dependencies:
 gnuplot                                       x86_64                     5.4.3-2.el9                                               epel                           820 k
 nodejs-docs                                   noarch                     1:16.20.2-8.el9_4                                         appstream                      7.1 M
 nodejs-full-i18n                              x86_64                     1:16.20.2-8.el9_4                                         appstream                      8.2 M
 npm                                           x86_64                     1:8.19.4-1.16.20.2.8.el9_4                                appstream                      1.7 M
 opennebula-guacd                              x86_64                     6.10.0.1-1.2.0+1.el9                                      opennebula                     220 k
 ruby-default-gems                             noarch                     3.0.7-165.el9_5                                           appstream                       30 k
 rubygem-bundler                               noarch                     2.2.33-165.el9_5                                          appstream                      370 k
 rubygem-rdoc                                  noarch                     6.3.4.1-165.el9_5                                         appstream                      398 k

Transaction Summary
=========================================================================================================================================================================
Install  133 Packages

Total download size: 196 M
Installed size: 1.1 G


[root@frontend ~]# sudo dnf install opennebula opennebula-sunstone opennebula-fireedge
Last metadata expiration check: 0:17:03 ago on Tue Sep 16 19:06:25 2025.
Package opennebula-6.10.0.1-1.el9.x86_64 is already installed.
Package opennebula-sunstone-6.10.0.1-1.el9.noarch is already installed.
Package opennebula-fireedge-6.10.0.1-1.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

🌞 Configuration OpenNebula

dans le fichier /etc/one/oned.conf, définissez correctement les paramètres de connexion à la base de données, c'est la clause DB = que vous devez remplacer par
```
[root@frontend ~]# nano /etc/one/oned.conf

# Sample configuration for MySQL
 DB = [ BACKEND = "mysql",
        SERVER  = "localhost",
        PORT    = 0,
        USER    = "oneadmin",
        PASSWD  = "",
        DB_NAME = "opennebula",
        CONNECTIONS = 25,
        COMPARE_BINARY = "no" ]
```

🌞 Créer un user pour se log sur la WebUI OpenNebula

pour ça, il faut se log en tant que l'utilisateur oneadmin sur le serveur
une fois connecté en tant que oneadmin, inscrivez le user oneadmin et le password de votre choix dans le fichier /var/lib/one/.one/one_auth sous la forme user:password
```
[chims@frontend ~]$ su oneadmin
Password:
[oneadmin@frontend chims]$

[oneadmin@frontend chims]$ nano /var/lib/one/.one/one_auth
```



🌞 Démarrer les services OpenNebula

démarrez les services opennebula, opennebula-sunstone
activez-les aussi au démarrage de la machine
```
[chims@frontend ~]$ sudo systemctl start opennebula opennebula-sunstone
[sudo] password for chims:
[chims@frontend ~]$ sudo systemctl start opennebula opennebula-sunstone
[chims@frontend ~]$ sudo systemctl enable opennebula opennebula-sunstone
Created symlink /etc/systemd/system/multi-user.target.wants/opennebula.service → /usr/lib/systemd/system/opennebula.service.
Created symlink /etc/systemd/system/multi-user.target.wants/opennebula-sunstone.service → /usr/lib/systemd/system/opennebula-sunstone.service.
[chims@frontend ~]$ sudo systemctl status opennebula opennebula-sunstone
● opennebula.service - OpenNebula Cloud Controller Daemon
     Loaded: loaded (/usr/lib/systemd/system/opennebula.service; enabled; preset: disabled)
     Active: active (running) since Tue 2025-09-16 20:12:24 CEST; 33min ago
   Main PID: 9300 (oned)
      Tasks: 74 (limit: 10850)
     Memory: 335.9M
        CPU: 1min 52.147s
     CGroup: /system.slice/opennebula.service
             ├─9300 /usr/bin/oned -f
             ├─9301 /usr/bin/ruby /usr/lib/one/mads/one_hm.rb -p 2101 -l 2102 -b 127.0.0.1
             ├─9325 /usr/bin/ruby /usr/lib/one/mads/one_vmm_exec.rb -t 15 -r 0 kvm -p
             ├─9353 /usr/bin/ruby /usr/lib/one/mads/one_vmm_exec.rb -t 15 -r 0 lxc
             ├─9375 /usr/bin/ruby /usr/lib/one/mads/one_vmm_exec.rb -t 15 -r 0 kvm
             ├─9394 /usr/bin/ruby /usr/lib/one/mads/one_tm.rb -t 15 -d dummy,lvm,shared,fs_lvm,fs_lvm_ssh,qcow2,ssh,ceph,dev,vcenter,iscsi_libvirt
             ├─9417 /usr/bin/ruby /usr/lib/one/mads/one_auth_mad.rb --authn ssh,x509,ldap,server_cipher,server_x509
             ├─9432 /usr/bin/ruby /usr/lib/one/mads/one_datastore.rb -t 15 -d dummy,fs,lvm,ceph,dev,iscsi_libvirt,vcenter,restic,rsync -s shared,ssh,ceph,fs_lvm,fs_lvm_>
             ├─9449 /usr/bin/ruby /usr/lib/one/mads/one_market.rb -t 15 -m http,s3,one,linuxcontainers
             ├─9466 /usr/bin/ruby /usr/lib/one/mads/one_ipam.rb -t 1 -i dummy,aws,equinix,vultr
             ├─9481 /usr/lib/one/mads/onemonitord "-c monitord.conf"
             ├─9482 /usr/bin/ruby /usr/lib/one/mads/one_im_exec.rb -r 3 -t 15 -w 90 kvm
             ├─9495 /usr/bin/ruby /usr/lib/one/mads/one_im_exec.rb -r 3 -t 15 -w 90 lxc
             └─9508 /usr/bin/ruby /usr/lib/one/mads/one_im_exec.rb -r 3 -t 15 -w 90 qemu

Sep 16 20:11:51 frontend.one systemd[1]: Starting OpenNebula Cloud Controller Daemon...
Sep 16 20:11:52 frontend.one opennebula[9298]: gzip: /var/log/one/vcenter_monitor*.log: No such file or directory
Sep 16 20:12:24 frontend.one systemd[1]: Started OpenNebula Cloud Controller Daemon.

● opennebula-sunstone.service - OpenNebula Web UI Server
     Loaded: loaded (/usr/lib/systemd/system/opennebula-sunstone.service; enabled; preset: disabled)
     Active: active (running) since Tue 2025-09-16 20:10:15 CEST; 35min ago
   Main PID: 8803 (ruby)
      Tasks: 21 (limit: 10850)
     Memory: 139.5M
```

C. Conf système¶
🌞 Ouverture firewall

ouvrez les ports suivants, avec des commandes firewall-cmd :
```
[chims@frontend ~]$ sudo firewall-cmd --add-port=22/tcp --permanent
[sudo] password for chims:
success
[chims@frontend ~]$ sudo firewall-cmd --add-port=2633/tcp --permanent
success
[chims@frontend ~]$ sudo firewall-cmd --add-port=4124/tcp --permanent
success
[chims@frontend ~]$ sudo firewall-cmd --add-port=4124/udp --permanent
success
[chims@frontend ~]$ sudo firewall-cmd --add-port=29876/tcp --permanent
success
[chims@frontend ~]$ sudo firewall-cmd --reload
success
[chims@frontend ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160 ens192
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 9869/tcp 22/tcp 2633/tcp 4124/tcp 4124/udp 29876/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[chims@frontend ~]$
```

D. Test
A ce stade, vous devriez déjà pouvoir visiter la WebUI de OpenNebula.

RDV sur http://10.3.1.10:9869 avec un navigateur!
```

```

A. KVM¶
🌞 Ajouter des dépôts supplémentaires

ajoutez les dépôts de OpenNebula, les mêmes que pour le Frontend !

juste les dépôts, n'installez pas les paquets OpenNebula
ajoutez aussi les dépôts du serveur MySQL communautaire

le RPM de la partie précédente (comme sur le frontend)
ajoutez aussi les dépôts EPEL en exécutant :

```
[root@kvm1 chims]# dnf install -y epel-release
Last metadata expiration check: 0:00:21 ago on Wed Sep 17 14:08:29 2025.
Package epel-release-9-7.el9.noarch is already installed.
Dependencies resolved.
=================================================================================================================================================
 Package                                Architecture                     Version                            Repository                      Size
=================================================================================================================================================
Upgrading:
 epel-release                           noarch                           9-10.el9                           epel                            19 k

Transaction Summary
=================================================================================================================================================
Upgrade  1 Package

Total download size: 19 k
Downloading Packages:
epel-release-9-10.el9.noarch.rpm                                                                                  31 kB/s |  19 kB     00:00
-------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                             14 kB/s |  19 kB     00:01
Extra Packages for Enterprise Linux 9 - x86_64                                                                   1.2 MB/s | 1.6 kB     00:00
Importing GPG key 0x3228467C:
 Userid     : "Fedora (epel9) <epel@fedoraproject.org>"
 Fingerprint: FF8A D134 4597 106E CE81 3B91 8A38 72BF 3228 467C
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-9
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                         1/1
  Upgrading        : epel-release-9-10.el9.noarch                                                                                            1/2
  Running scriptlet: epel-release-9-10.el9.noarch                                                                                            1/2
  Cleanup          : epel-release-9-7.el9.noarch                                                                                             2/2
  Running scriptlet: epel-release-9-7.el9.noarch                                                                                             2/2
  Verifying        : epel-release-9-10.el9.noarch                                                                                            1/2
  Verifying        : epel-release-9-7.el9.noarch                                                                                             2/2

Upgraded:
  epel-release-9-10.el9.noarch

Complete!
```

🌞 Installer les libs MySQL
```
[root@kvm1 chims]# dnf install -y mysql-community-server
Last metadata expiration check: 0:00:51 ago on Wed Sep 17 14:08:29 2025.
Dependencies resolved.
=================================================================================================================================================
 Package                                         Architecture            Version                        Repository                          Size
=================================================================================================================================================
Installing:
 mysql-community-server                          x86_64                  8.0.43-1.el9                   mysql80-community                   50 M
Installing dependencies:
 mysql-community-client                          x86_64                  8.0.43-1.el9                   mysql80-community                  3.3 M
 mysql-community-client-plugins                  x86_64                  8.0.43-1.el9                   mysql80-community                  1.4 M
 mysql-community-common                          x86_64                  8.0.43-1.el9                   mysql80-community                  556 k
 mysql-community-icu-data-files                  x86_64                  8.0.43-1.el9                   mysql80-community                  2.3 M
 mysql-community-libs                            x86_64                  8.0.43-1.el9                   mysql80-community                  1.5 M

Transaction Summary
=================================================================================================================================================
Install  6 Packages

Total download size: 59 M
Installed size: 337 M
Downloading Packages:
(1/6): mysql-community-common-8.0.43-1.el9.x86_64.rpm                                                            185 kB/s | 556 kB     00:03
(2/6): mysql-community-client-plugins-8.0.43-1.el9.x86_64.rpm                                                    248 kB/s | 1.4 MB     00:05
(3/6): mysql-community-libs-8.0.43-1.el9.x86_64.rpm                                                              297 kB/s | 1.5 MB     00:05
(4/6): mysql-community-icu-data-files-8.0.43-1.el9.x86_64.rpm                                                    230 kB/s | 2.3 MB     00:10
(5/6): mysql-community-client-8.0.43-1.el9.x86_64.rpm                                                            215 kB/s | 3.3 MB     00:15
(6/6): mysql-community-server-8.0.43-1.el9.x86_64.rpm                                                            356 kB/s |  50 MB     02:22
-------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                            391 kB/s |  59 MB     02:33
MySQL 8.0 Community Server                                                                                       3.0 MB/s | 3.1 kB     00:00
Importing GPG key 0xA8D3785C:
 Userid     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 Fingerprint: BCA4 3417 C3B4 85DD 128E C6D4 B7B3 B788 A8D3 785C
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2023
Key imported successfully
MySQL 8.0 Community Server                                                                                       3.0 MB/s | 3.1 kB     00:00
Importing GPG key 0x3A79BD29:
 Userid     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 Fingerprint: 859B E8D7 C586 F538 430B 19C2 467B 942D 3A79 BD29
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                         1/1
  Installing       : mysql-community-common-8.0.43-1.el9.x86_64                                                                              1/6
  Installing       : mysql-community-client-plugins-8.0.43-1.el9.x86_64                                                                      2/6
  Installing       : mysql-community-libs-8.0.43-1.el9.x86_64                                                                                3/6
  Running scriptlet: mysql-community-libs-8.0.43-1.el9.x86_64                                                                                3/6
  Installing       : mysql-community-client-8.0.43-1.el9.x86_64                                                                              4/6
  Installing       : mysql-community-icu-data-files-8.0.43-1.el9.x86_64                                                                      5/6
  Running scriptlet: mysql-community-server-8.0.43-1.el9.x86_64                                                                              6/6
  Installing       : mysql-community-server-8.0.43-1.el9.x86_64                                                                              6/6
  Running scriptlet: mysql-community-server-8.0.43-1.el9.x86_64                                                                              6/6
  Verifying        : mysql-community-client-8.0.43-1.el9.x86_64                                                                              1/6
  Verifying        : mysql-community-client-plugins-8.0.43-1.el9.x86_64                                                                      2/6
  Verifying        : mysql-community-common-8.0.43-1.el9.x86_64                                                                              3/6
  Verifying        : mysql-community-icu-data-files-8.0.43-1.el9.x86_64                                                                      4/6
  Verifying        : mysql-community-libs-8.0.43-1.el9.x86_64                                                                                5/6
  Verifying        : mysql-community-server-8.0.43-1.el9.x86_64                                                                              6/6

Installed:
  mysql-community-client-8.0.43-1.el9.x86_64                          mysql-community-client-plugins-8.0.43-1.el9.x86_64
  mysql-community-common-8.0.43-1.el9.x86_64                          mysql-community-icu-data-files-8.0.43-1.el9.x86_64
  mysql-community-libs-8.0.43-1.el9.x86_64                            mysql-community-server-8.0.43-1.el9.x86_64

Complete!

```

🌞 Installer KVM

un paquet spécifique qui vient des dépôts OpenNebula : opennebula-node-kvm
```
[chims@kvm1 ~]$ sudo dnf install opennebula-node-kvm
[sudo] password for chims:
Last metadata expiration check: 0:02:30 ago on Wed Sep 17 14:12:16 2025.
Dependencies resolved.
=================================================================================================================================================
 Package                                           Architecture      Version                                         Repository             Size
=================================================================================================================================================
Installing:
 opennebula-node-kvm                               noarch            6.10.0.1-1.el9                                  opennebula             13 k
Installing dependencies:
 augeas                                            x86_64            1.14.1-2.el9                                    appstream              63 k
 augeas-libs                                       x86_64            1.14.1-2.el9                                    appstream             373 k
 boost-iostreams                                   x86_64            1.75.0-10.el9                                   appstream              36 k
 boost-system                                      x86_64            1.75.0-10.el9                                   appstream              11 k
 boost-thread                                      x86_64            1.75.0-10.el9                                   appstream              53 k
 capstone                                          x86_64            4.0.2-10.el9                                    appstream             766 k
 daxctl-libs                                       x86_64            78-2.el9                                        baseos                 41 k
 device-mapper-multipath-libs                      x86_64            0.8.7-35.el9_6.1                                baseos                267 k
 dnsmasq                                           x86_64            2.85-16.el9_4                                   appstream             326 k
 edk2-ovmf                                         noarch            20241117-2.el9                                  appstream             6.1 M
 flac-libs                                         x86_64            1.3.3-10.el9_2.1                                appstream             217 k
 gnutls-dane                                       x86_64            3.8.3-6.el9                                     appstream              18 k
 gnutls-utils                                      x86_64            3.8.3-6.el9                                     appstream             283 k
 gsm                                               x86_64            1.0.19-6.el9                                    appstream              33 k
 gssproxy                                          x86_64            0.8.4-7.el9                                     baseos                108 k
 ipxe-roms-qemu                                    noarch            20200823-9.git4bd064de.el9                      appstream             673 k
 iscsi-initiator-utils                             x86_64            6.2.1.9-1.gita65a472.el9                        baseos                383 k
 iscsi-initiator-utils-iscsiuio                    x86_64            6.2.1.9-1.gita65a472.el9                        baseos                 80 k
 isns-utils-libs                                   x86_64            0.101-4.el9                                     baseos                100 k
 libX11-xcb                                        x86_64            1.7.0-11.el9                                    appstream              10 k
 libXfixes                                         x86_64            5.0.3-16.el9                                    appstream              19 k
 libXxf86vm                                        x86_64            1.1.4-18.el9                                    appstream              18 k
 libasyncns                                        x86_64            0.8-22.el9                                      appstream              29 k
 libblkio                                          x86_64            1.5.0-1.el9_4                                   appstream             1.0 M
 libdrm                                            x86_64            2.4.123-2.el9                                   appstream             158 k
 libepoxy                                          x86_64            1.5.5-4.el9                                     appstream             244 k
 libev                                             x86_64            4.33-6.el9                                      baseos                 51 k
 libfdt                                            x86_64            1.6.0-7.el9                                     appstream              33 k
 libglvnd                                          x86_64            1:1.3.4-1.el9                                   appstream             133 k
 libglvnd-egl                                      x86_64            1:1.3.4-1.el9                                   appstream              36 k
 libglvnd-glx                                      x86_64            1:1.3.4-1.el9                                   appstream             140 k
 libnbd                                            x86_64            1.20.3-1.el9                                    appstream             168 k
 libnfsidmap                                       x86_64            1:2.5.4-34.el9                                  baseos                 60 k
 libogg                                            x86_64            2:1.3.4-6.el9                                   appstream              32 k
 libpciaccess                                      x86_64            0.16-7.el9                                      baseos                 26 k
 libpmem                                           x86_64            1.12.1-1.el9                                    appstream             111 k
 librados2                                         x86_64            2:16.2.4-5.el9                                  appstream             3.4 M
 librbd1                                           x86_64            2:16.2.4-5.el9                                  appstream             3.0 M
 librdmacm                                         x86_64            54.0-1.el9                                      baseos                 70 k
 libslirp                                          x86_64            4.4.0-8.el9                                     appstream              67 k
 libsndfile                                        x86_64            1.0.31-9.el9                                    appstream             205 k
 libtpms                                           x86_64            0.9.1-5.20211126git1ff6fe1f43.el9_6             appstream             182 k
 liburing                                          x86_64            2.5-1.el9                                       appstream              38 k
 libverto-libev                                    x86_64            0.3.2-3.el9                                     baseos                 13 k
 libvirt                                           x86_64            10.10.0-7.6.el9_6                               appstream              29 k
 libvirt-client                                    x86_64            10.10.0-7.6.el9_6                               appstream             449 k
 libvirt-client-qemu                               x86_64            10.10.0-7.6.el9_6                               appstream              49 k
 libvirt-daemon                                    x86_64            10.10.0-7.6.el9_6                               appstream             215 k
 libvirt-daemon-common                             x86_64            10.10.0-7.6.el9_6                               appstream             137 k
 libvirt-daemon-config-network                     x86_64            10.10.0-7.6.el9_6                               appstream              31 k
 libvirt-daemon-config-nwfilter                    x86_64            10.10.0-7.6.el9_6                               appstream              37 k
 libvirt-daemon-driver-interface                   x86_64            10.10.0-7.6.el9_6                               appstream             221 k
 libvirt-daemon-driver-network                     x86_64            10.10.0-7.6.el9_6                               appstream             270 k
 libvirt-daemon-driver-nodedev                     x86_64            10.10.0-7.6.el9_6                               appstream             245 k
 libvirt-daemon-driver-nwfilter                    x86_64            10.10.0-7.6.el9_6                               appstream             257 k
 libvirt-daemon-driver-qemu                        x86_64            10.10.0-7.6.el9_6                               appstream             991 k
 libvirt-daemon-driver-secret                      x86_64            10.10.0-7.6.el9_6                               appstream             218 k
 libvirt-daemon-driver-storage                     x86_64            10.10.0-7.6.el9_6                               appstream              28 k
 libvirt-daemon-driver-storage-core                x86_64            10.10.0-7.6.el9_6                               appstream             273 k
 libvirt-daemon-driver-storage-disk                x86_64            10.10.0-7.6.el9_6                               appstream              39 k
 libvirt-daemon-driver-storage-iscsi               x86_64            10.10.0-7.6.el9_6                               appstream              37 k
 libvirt-daemon-driver-storage-logical             x86_64            10.10.0-7.6.el9_6                               appstream              41 k
 libvirt-daemon-driver-storage-mpath               x86_64            10.10.0-7.6.el9_6                               appstream              34 k
 libvirt-daemon-driver-storage-rbd                 x86_64            10.10.0-7.6.el9_6                               appstream              45 k
 libvirt-daemon-driver-storage-scsi                x86_64            10.10.0-7.6.el9_6                               appstream              36 k
 libvirt-daemon-lock                               x86_64            10.10.0-7.6.el9_6                               appstream              66 k
 libvirt-daemon-log                                x86_64            10.10.0-7.6.el9_6                               appstream              70 k
 libvirt-daemon-plugin-lockd                       x86_64            10.10.0-7.6.el9_6                               appstream              40 k
 libvirt-daemon-proxy                              x86_64            10.10.0-7.6.el9_6                               appstream             214 k
 libvirt-libs                                      x86_64            10.10.0-7.6.el9_6                               appstream             5.0 M
 libvorbis                                         x86_64            1:1.3.7-5.el9                                   appstream             192 k
 libwayland-client                                 x86_64            1.21.0-1.el9                                    appstream              33 k
 libwayland-server                                 x86_64            1.21.0-1.el9                                    appstream              41 k
 libxkbcommon                                      x86_64            1.0.3-4.el9                                     appstream             132 k
 libxshmfence                                      x86_64            1.3-10.el9                                      appstream              12 k
 libxslt                                           x86_64            1.1.34-13.el9_6                                 appstream             239 k
 lzop                                              x86_64            1.04-8.el9                                      baseos                 55 k
 mdevctl                                           x86_64            1.1.0-4.el9                                     appstream             758 k
 mesa-dri-drivers                                  x86_64            24.2.8-2.el9_6                                  appstream             9.4 M
 mesa-filesystem                                   x86_64            24.2.8-2.el9_6                                  appstream              11 k
 mesa-libEGL                                       x86_64            24.2.8-2.el9_6                                  appstream             141 k
 mesa-libGL                                        x86_64            24.2.8-2.el9_6                                  appstream             169 k
 mesa-libgbm                                       x86_64            24.2.8-2.el9_6                                  appstream              36 k
 mesa-libglapi                                     x86_64            24.2.8-2.el9_6                                  appstream              44 k
 nbdkit-basic-filters                              x86_64            1.38.5-5.el9_6                                  appstream             308 k
 nbdkit-basic-plugins                              x86_64            1.38.5-5.el9_6                                  appstream             207 k
 nbdkit-selinux                                    noarch            1.38.5-5.el9_6                                  appstream              23 k
 nbdkit-server                                     x86_64            1.38.5-5.el9_6                                  appstream             128 k
 ndctl-libs                                        x86_64            78-2.el9                                        baseos                 89 k
 nfs-utils                                         x86_64            1:2.5.4-34.el9                                  baseos                430 k
 numad                                             x86_64            0.5-37.20150602git.el9                          baseos                 36 k
 opennebula-common                                 noarch            6.10.0.1-1.el9                                  opennebula             23 k
 opennebula-common-onecfg                          noarch            6.10.0.1-1.el9                                  opennebula            9.1 k
 opennebula-rubygems                               x86_64            6.10.0.1-1.el9                                  opennebula             16 M
 opus                                              x86_64            1.3.1-10.el9                                    appstream             199 k
 passt-selinux                                     noarch            0^20250217.ga1e48a0-10.el9_6                    appstream              27 k
 pulseaudio-libs                                   x86_64            15.0-3.el9                                      appstream             663 k
 python3-cffi                                      x86_64            1.14.5-5.el9                                    baseos                241 k
 python3-cryptography                              x86_64            36.0.1-4.el9                                    baseos                1.2 M
 python3-libvirt                                   x86_64            10.10.0-1.el9                                   appstream             333 k
 python3-lxml                                      x86_64            4.6.5-3.el9                                     appstream             1.2 M
 python3-ply                                       noarch            3.11-14.el9.0.1                                 baseos                103 k
 python3-pycparser                                 noarch            2.20-6.el9                                      baseos                124 k
 qemu-img                                          x86_64            17:9.1.0-15.el9_6.7                             appstream             2.5 M
 qemu-kvm                                          x86_64            17:9.1.0-15.el9_6.7                             appstream              64 k
 qemu-kvm-audio-pa                                 x86_64            17:9.1.0-15.el9_6.7                             appstream              73 k
 qemu-kvm-block-blkio                              x86_64            17:9.1.0-15.el9_6.7                             appstream              76 k
 qemu-kvm-block-rbd                                x86_64            17:9.1.0-15.el9_6.7                             appstream              78 k
 qemu-kvm-common                                   x86_64            17:9.1.0-15.el9_6.7                             appstream             668 k
 qemu-kvm-core                                     x86_64            17:9.1.0-15.el9_6.7                             appstream             4.9 M
 qemu-kvm-device-display-virtio-gpu                x86_64            17:9.1.0-15.el9_6.7                             appstream              85 k
 qemu-kvm-device-display-virtio-gpu-pci            x86_64            17:9.1.0-15.el9_6.7                             appstream              68 k
 qemu-kvm-device-display-virtio-vga                x86_64            17:9.1.0-15.el9_6.7                             appstream              69 k
 qemu-kvm-device-usb-host                          x86_64            17:9.1.0-15.el9_6.7                             appstream              82 k
 qemu-kvm-device-usb-redirect                      x86_64            17:9.1.0-15.el9_6.7                             appstream              87 k
 qemu-kvm-docs                                     x86_64            17:9.1.0-15.el9_6.7                             appstream             1.2 M
 qemu-kvm-tools                                    x86_64            17:9.1.0-15.el9_6.7                             appstream             571 k
 qemu-kvm-ui-egl-headless                          x86_64            17:9.1.0-15.el9_6.7                             appstream              69 k
 qemu-kvm-ui-opengl                                x86_64            17:9.1.0-15.el9_6.7                             appstream              76 k
 qemu-pr-helper                                    x86_64            17:9.1.0-15.el9_6.7                             appstream             492 k
 rpcbind                                           x86_64            1.2.6-7.el9                                     baseos                 56 k
 ruby                                              x86_64            3.0.7-165.el9_5                                 appstream              38 k
 ruby-libs                                         x86_64            3.0.7-165.el9_5                                 appstream             3.2 M
 rubygem-io-console                                x86_64            0.5.7-165.el9_5                                 appstream              23 k
 rubygem-json                                      x86_64            2.5.1-165.el9_5                                 appstream              51 k
 rubygem-psych                                     x86_64            3.3.2-165.el9_5                                 appstream              48 k
 rubygem-rexml                                     noarch            3.2.5-165.el9_5                                 appstream              92 k
 rubygem-sqlite3                                   x86_64            1.4.2-8.el9                                     epel                   44 k
 rubygems                                          noarch            3.2.33-165.el9_5                                appstream             253 k
 scrub                                             x86_64            2.6.1-4.el9                                     appstream              44 k
 seabios-bin                                       noarch            1.16.3-4.el9                                    appstream             100 k
 seavgabios-bin                                    noarch            1.16.3-4.el9                                    appstream              34 k
 sssd-nfs-idmap                                    x86_64            2.9.6-4.el9_6.2                                 baseos                 39 k
 swtpm                                             x86_64            0.8.0-2.el9_4                                   appstream              42 k
 swtpm-libs                                        x86_64            0.8.0-2.el9_4                                   appstream              48 k
 swtpm-tools                                       x86_64            0.8.0-2.el9_4                                   appstream             116 k
 systemd-container                                 x86_64            252-51.el9_6.1                                  baseos                577 k
 unbound-libs                                      x86_64            1.16.2-19.el9_6.1                               appstream             550 k
 usbredir                                          x86_64            0.13.0-2.el9                                    appstream              50 k
 virtiofsd                                         x86_64            1.13.2-1.el9_6                                  appstream             990 k
 xkeyboard-config                                  noarch            2.33-2.el9                                      appstream             779 k
 zstd                                              x86_64            1.5.5-1.el9                                     baseos                461 k
Installing weak dependencies:
 nbdkit                                            x86_64            1.38.5-5.el9_6                                  appstream             8.5 k
 nbdkit-curl-plugin                                x86_64            1.38.5-5.el9_6                                  appstream              39 k
 nbdkit-ssh-plugin                                 x86_64            1.38.5-5.el9_6                                  appstream              28 k
 passt                                             x86_64            0^20250217.ga1e48a0-10.el9_6                    appstream             259 k
 ruby-default-gems                                 noarch            3.0.7-165.el9_5                                 appstream              30 k
 rubygem-bigdecimal                                x86_64            3.0.0-165.el9_5                                 appstream              52 k
 rubygem-bundler                                   noarch            2.2.33-165.el9_5                                appstream             370 k
 rubygem-rdoc                                      noarch            6.3.4.1-165.el9_5                               appstream             398 k

Transaction Summary
=================================================================================================================================================
Install  151 Packages

Total download size: 80 M
Installed size: 482 M
Is this ok [y/N]: y
Downloading Packages:
(1/151): rubygem-sqlite3-1.4.2-8.el9.x86_64.rpm                                                                  143 kB/s |  44 kB     00:00
(2/151): opennebula-common-onecfg-6.10.0.1-1.el9.noarch.rpm                                                       20 kB/s | 9.1 kB     00:00
(3/151): opennebula-common-6.10.0.1-1.el9.noarch.rpm                                                              45 kB/s |  23 kB     00:00
(4/151): opennebula-node-kvm-6.10.0.1-1.el9.noarch.rpm                                                            30 kB/s |  13 kB     00:00
(5/151): python3-pycparser-2.20-6.el9.noarch.rpm                                                                 130 kB/s | 124 kB     00:00
(6/151): sssd-nfs-idmap-2.9.6-4.el9_6.2.x86_64.rpm                                                                96 kB/s |  39 kB     00:00
(7/151): numad-0.5-37.20150602git.el9.x86_64.rpm                                                                 138 kB/s |  36 kB     00:00
(8/151): isns-utils-libs-0.101-4.el9.x86_64.rpm                                                                   59 kB/s | 100 kB     00:01
(9/151): iscsi-initiator-utils-6.2.1.9-1.gita65a472.el9.x86_64.rpm                                               187 kB/s | 383 kB     00:02
(10/151): lzop-1.04-8.el9.x86_64.rpm                                                                             148 kB/s |  55 kB     00:00
(11/151): librdmacm-54.0-1.el9.x86_64.rpm                                                                        147 kB/s |  70 kB     00:00
(12/151): device-mapper-multipath-libs-0.8.7-35.el9_6.1.x86_64.rpm                                               113 kB/s | 267 kB     00:02
(13/151): python3-ply-3.11-14.el9.0.1.noarch.rpm                                                                  79 kB/s | 103 kB     00:01
(14/151): iscsi-initiator-utils-iscsiuio-6.2.1.9-1.gita65a472.el9.x86_64.rpm                                      92 kB/s |  80 kB     00:00
(15/151): libpciaccess-0.16-7.el9.x86_64.rpm                                                                      87 kB/s |  26 kB     00:00
(16/151): gssproxy-0.8.4-7.el9.x86_64.rpm                                                                        101 kB/s | 108 kB     00:01
(17/151): ndctl-libs-78-2.el9.x86_64.rpm                                                                          61 kB/s |  89 kB     00:01
(18/151): libev-4.33-6.el9.x86_64.rpm                                                                             59 kB/s |  51 kB     00:00
(19/151): libverto-libev-0.3.2-3.el9.x86_64.rpm                                                                   66 kB/s |  13 kB     00:00
(20/151): python3-cryptography-36.0.1-4.el9.x86_64.rpm                                                            99 kB/s | 1.2 MB     00:11
(21/151): python3-cffi-1.14.5-5.el9.x86_64.rpm                                                                    72 kB/s | 241 kB     00:03
(22/151): nfs-utils-2.5.4-34.el9.x86_64.rpm                                                                       94 kB/s | 430 kB     00:04
(23/151): rpcbind-1.2.6-7.el9.x86_64.rpm                                                                          66 kB/s |  56 kB     00:00
(24/151): systemd-container-252-51.el9_6.1.x86_64.rpm                                                             85 kB/s | 577 kB     00:06
(25/151): libnfsidmap-2.5.4-34.el9.x86_64.rpm                                                                     88 kB/s |  60 kB     00:00
(26/151): daxctl-libs-78-2.el9.x86_64.rpm                                                                        134 kB/s |  41 kB     00:00
(27/151): libslirp-4.4.0-8.el9.x86_64.rpm                                                                         69 kB/s |  67 kB     00:00
(28/151): zstd-1.5.5-1.el9.x86_64.rpm                                                                             78 kB/s | 461 kB     00:05
(29/151): libvirt-daemon-plugin-lockd-10.10.0-7.6.el9_6.x86_64.rpm                                               149 kB/s |  40 kB     00:00
(30/151): dnsmasq-2.85-16.el9_4.x86_64.rpm                                                                       174 kB/s | 326 kB     00:01
(31/151): flac-libs-1.3.3-10.el9_2.1.x86_64.rpm                                                                   82 kB/s | 217 kB     00:02
(32/151): rubygem-json-2.5.1-165.el9_5.x86_64.rpm                                                                 40 kB/s |  51 kB     00:01
(33/151): libvirt-daemon-proxy-10.10.0-7.6.el9_6.x86_64.rpm                                                      161 kB/s | 214 kB     00:01
(34/151): libXxf86vm-1.1.4-18.el9.x86_64.rpm                                                                      16 kB/s |  18 kB     00:01
(35/151): passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch.rpm                                                   65 kB/s |  27 kB     00:00
(36/151): libglvnd-1.3.4-1.el9.x86_64.rpm                                                                        108 kB/s | 133 kB     00:01
(37/151): libwayland-client-1.21.0-1.el9.x86_64.rpm                                                               11 kB/s |  33 kB     00:02
(38/151): virtiofsd-1.13.2-1.el9_6.x86_64.rpm                                                                    115 kB/s | 990 kB     00:08
(39/151): libvirt-daemon-10.10.0-7.6.el9_6.x86_64.rpm                                                             39 kB/s | 215 kB     00:05
(40/151): libvirt-daemon-lock-10.10.0-7.6.el9_6.x86_64.rpm                                                        56 kB/s |  66 kB     00:01
(41/151): rubygem-bundler-2.2.33-165.el9_5.noarch.rpm                                                             64 kB/s | 370 kB     00:05
(42/151): swtpm-0.8.0-2.el9_4.x86_64.rpm                                                                          15 kB/s |  42 kB     00:02
(43/151): swtpm-tools-0.8.0-2.el9_4.x86_64.rpm                                                                    38 kB/s | 116 kB     00:03
(44/151): nbdkit-selinux-1.38.5-5.el9_6.noarch.rpm                                                               9.3 kB/s |  23 kB     00:02
(45/151): libvirt-daemon-driver-nodedev-10.10.0-7.6.el9_6.x86_64.rpm                                              51 kB/s | 245 kB     00:04
(46/151): passt-0^20250217.ga1e48a0-10.el9_6.x86_64.rpm                                                           53 kB/s | 259 kB     00:04
(47/151): qemu-kvm-block-rbd-9.1.0-15.el9_6.7.x86_64.rpm                                                          29 kB/s |  78 kB     00:02
(48/151): libvirt-daemon-log-10.10.0-7.6.el9_6.x86_64.rpm                                                         20 kB/s |  70 kB     00:03
(49/151): libvirt-libs-10.10.0-7.6.el9_6.x86_64.rpm                                                              165 kB/s | 5.0 MB     00:31
(50/151): mesa-libGL-24.2.8-2.el9_6.x86_64.rpm                                                                   109 kB/s | 169 kB     00:01
(51/151): augeas-libs-1.14.1-2.el9.x86_64.rpm                                                                     99 kB/s | 373 kB     00:03
(52/151): mesa-libglapi-24.2.8-2.el9_6.x86_64.rpm                                                                 14 kB/s |  44 kB     00:03
(53/151): libnbd-1.20.3-1.el9.x86_64.rpm                                                                         136 kB/s | 168 kB     00:01
(54/151): ruby-3.0.7-165.el9_5.x86_64.rpm                                                                         30 kB/s |  38 kB     00:01
(55/151): boost-thread-1.75.0-10.el9.x86_64.rpm                                                                   72 kB/s |  53 kB     00:00
(56/151): libtpms-0.9.1-5.20211126git1ff6fe1f43.el9_6.x86_64.rpm                                                 106 kB/s | 182 kB     00:01
(57/151): seavgabios-bin-1.16.3-4.el9.noarch.rpm                                                                  20 kB/s |  34 kB     00:01
(58/151): liburing-2.5-1.el9.x86_64.rpm                                                                           47 kB/s |  38 kB     00:00
(59/151): nbdkit-server-1.38.5-5.el9_6.x86_64.rpm                                                                116 kB/s | 128 kB     00:01
(60/151): libvirt-daemon-config-network-10.10.0-7.6.el9_6.x86_64.rpm                                              24 kB/s |  31 kB     00:01
(61/151): scrub-2.6.1-4.el9.x86_64.rpm                                                                           4.1 kB/s |  44 kB     00:10
(62/151): libxshmfence-1.3-10.el9.x86_64.rpm                                                                     2.1 kB/s |  12 kB     00:05
(63/151): qemu-kvm-docs-9.1.0-15.el9_6.7.x86_64.rpm                                                               68 kB/s | 1.2 MB     00:18
(64/151): qemu-kvm-device-display-virtio-gpu-9.1.0-15.el9_6.7.x86_64.rpm                                          28 kB/s |  85 kB     00:02
(65/151): libvirt-daemon-driver-storage-mpath-10.10.0-7.6.el9_6.x86_64.rpm                                        23 kB/s |  34 kB     00:01
(66/151): rubygem-rdoc-6.3.4.1-165.el9_5.noarch.rpm                                                               94 kB/s | 398 kB     00:04
(67/151): libvirt-daemon-driver-storage-core-10.10.0-7.6.el9_6.x86_64.rpm                                         39 kB/s | 273 kB     00:06
(68/151): qemu-kvm-device-usb-redirect-9.1.0-15.el9_6.7.x86_64.rpm                                                22 kB/s |  87 kB     00:03
(69/151): libvirt-daemon-driver-storage-iscsi-10.10.0-7.6.el9_6.x86_64.rpm                                        30 kB/s |  37 kB     00:01
(70/151): libvirt-client-10.10.0-7.6.el9_6.x86_64.rpm                                                             86 kB/s | 449 kB     00:05
(71/151): libX11-xcb-1.7.0-11.el9.x86_64.rpm                                                                     1.7 kB/s |  10 kB     00:05
(72/151): ipxe-roms-qemu-20200823-9.git4bd064de.el9.noarch.rpm                                                    57 kB/s | 673 kB     00:11
(73/151): mesa-libgbm-24.2.8-2.el9_6.x86_64.rpm                                                                   28 kB/s |  36 kB     00:01
(74/151): libglvnd-glx-1.3.4-1.el9.x86_64.rpm                                                                    167 kB/s | 140 kB     00:00
(75/151): libvirt-daemon-driver-nwfilter-10.10.0-7.6.el9_6.x86_64.rpm                                            116 kB/s | 257 kB     00:02
(76/151): capstone-4.0.2-10.el9.x86_64.rpm                                                                       139 kB/s | 766 kB     00:05
(77/151): libepoxy-1.5.5-4.el9.x86_64.rpm                                                                         48 kB/s | 244 kB     00:05
(78/151): libvirt-daemon-driver-storage-logical-10.10.0-7.6.el9_6.x86_64.rpm                                      37 kB/s |  41 kB     00:01
(79/151): nbdkit-curl-plugin-1.38.5-5.el9_6.x86_64.rpm                                                            13 kB/s |  39 kB     00:02
(80/151): xkeyboard-config-2.33-2.el9.noarch.rpm                                                                 187 kB/s | 779 kB     00:04
(81/151): libglvnd-egl-1.3.4-1.el9.x86_64.rpm                                                                     26 kB/s |  36 kB     00:01
(82/151): qemu-kvm-ui-egl-headless-9.1.0-15.el9_6.7.x86_64.rpm                                                   139 kB/s |  69 kB     00:00
(83/151): rubygems-3.2.33-165.el9_5.noarch.rpm                                                                    38 kB/s | 253 kB     00:06
(84/151): qemu-kvm-9.1.0-15.el9_6.7.x86_64.rpm                                                                    19 kB/s |  64 kB     00:03
(85/151): opennebula-rubygems-6.10.0.1-1.el9.x86_64.rpm                                                          116 kB/s |  16 MB     02:21
(86/151): libpmem-1.12.1-1.el9.x86_64.rpm                                                                         48 kB/s | 111 kB     00:02
(87/151): gnutls-dane-3.8.3-6.el9.x86_64.rpm                                                                      16 kB/s |  18 kB     00:01
(88/151): qemu-pr-helper-9.1.0-15.el9_6.7.x86_64.rpm                                                             107 kB/s | 492 kB     00:04
(89/151): augeas-1.14.1-2.el9.x86_64.rpm                                                                          24 kB/s |  63 kB     00:02
(90/151): swtpm-libs-0.8.0-2.el9_4.x86_64.rpm                                                                     26 kB/s |  48 kB     00:01
(91/151): pulseaudio-libs-15.0-3.el9.x86_64.rpm                                                                  103 kB/s | 663 kB     00:06
(92/151): libvirt-daemon-common-10.10.0-7.6.el9_6.x86_64.rpm                                                      35 kB/s | 137 kB     00:03
(93/151): libvirt-daemon-driver-storage-disk-10.10.0-7.6.el9_6.x86_64.rpm                                         17 kB/s |  39 kB     00:02
(94/151): rubygem-bigdecimal-3.0.0-165.el9_5.x86_64.rpm                                                           19 kB/s |  52 kB     00:02
(95/151): qemu-kvm-core-9.1.0-15.el9_6.7.x86_64.rpm                                                              169 kB/s | 4.9 MB     00:29
(96/151): nbdkit-ssh-plugin-1.38.5-5.el9_6.x86_64.rpm                                                             11 kB/s |  28 kB     00:02
(97/151): unbound-libs-1.16.2-19.el9_6.1.x86_64.rpm                                                               97 kB/s | 550 kB     00:05
(98/151): libdrm-2.4.123-2.el9.x86_64.rpm                                                                         39 kB/s | 158 kB     00:04
(99/151): qemu-kvm-device-display-virtio-vga-9.1.0-15.el9_6.7.x86_64.rpm                                          23 kB/s |  69 kB     00:02
(100/151): qemu-img-9.1.0-15.el9_6.7.x86_64.rpm                                                                  138 kB/s | 2.5 MB     00:18
(101/151): boost-iostreams-1.75.0-10.el9.x86_64.rpm                                                               13 kB/s |  36 kB     00:02
(102/151): gsm-1.0.19-6.el9.x86_64.rpm                                                                            14 kB/s |  33 kB     00:02
(103/151): ruby-default-gems-3.0.7-165.el9_5.noarch.rpm                                                          104 kB/s |  30 kB     00:00
(104/151): rubygem-io-console-0.5.7-165.el9_5.x86_64.rpm                                                         108 kB/s |  23 kB     00:00
(105/151): opus-1.3.1-10.el9.x86_64.rpm                                                                          229 kB/s | 199 kB     00:00
(106/151): libvirt-daemon-driver-interface-10.10.0-7.6.el9_6.x86_64.rpm                                          164 kB/s | 221 kB     00:01
(107/151): usbredir-0.13.0-2.el9.x86_64.rpm                                                                       35 kB/s |  50 kB     00:01
(108/151): rubygem-rexml-3.2.5-165.el9_5.noarch.rpm                                                               96 kB/s |  92 kB     00:00
(109/151): libogg-1.3.4-6.el9.x86_64.rpm                                                                          61 kB/s |  32 kB     00:00
(110/151): boost-system-1.75.0-10.el9.x86_64.rpm                                                                  25 kB/s |  11 kB     00:00
(111/151): rubygem-psych-3.3.2-165.el9_5.x86_64.rpm                                                              165 kB/s |  48 kB     00:00
(112/151): libXfixes-5.0.3-16.el9.x86_64.rpm                                                                      90 kB/s |  19 kB     00:00
(113/151): qemu-kvm-device-usb-host-9.1.0-15.el9_6.7.x86_64.rpm                                                  167 kB/s |  82 kB     00:00
(114/151): qemu-kvm-device-display-virtio-gpu-pci-9.1.0-15.el9_6.7.x86_64.rpm                                    105 kB/s |  68 kB     00:00
(115/151): libasyncns-0.8-22.el9.x86_64.rpm                                                                       39 kB/s |  29 kB     00:00
(116/151): libvorbis-1.3.7-5.el9.x86_64.rpm                                                                      145 kB/s | 192 kB     00:01
(117/151): seabios-bin-1.16.3-4.el9.noarch.rpm                                                                    63 kB/s | 100 kB     00:01
(118/151): nbdkit-basic-plugins-1.38.5-5.el9_6.x86_64.rpm                                                         86 kB/s | 207 kB     00:02
(119/151): gnutls-utils-3.8.3-6.el9.x86_64.rpm                                                                    99 kB/s | 283 kB     00:02
(120/151): libvirt-daemon-config-nwfilter-10.10.0-7.6.el9_6.x86_64.rpm                                            13 kB/s |  37 kB     00:02
(121/151): qemu-kvm-block-blkio-9.1.0-15.el9_6.7.x86_64.rpm                                                       18 kB/s |  76 kB     00:04
(122/151): libvirt-daemon-driver-secret-10.10.0-7.6.el9_6.x86_64.rpm                                              51 kB/s | 218 kB     00:04
(123/151): mdevctl-1.1.0-4.el9.x86_64.rpm                                                                         81 kB/s | 758 kB     00:09
(124/151): libblkio-1.5.0-1.el9_4.x86_64.rpm                                                                      97 kB/s | 1.0 MB     00:10
(125/151): libsndfile-1.0.31-9.el9.x86_64.rpm                                                                     68 kB/s | 205 kB     00:03
(126/151): python3-libvirt-10.10.0-1.el9.x86_64.rpm                                                               80 kB/s | 333 kB     00:04
(127/151): ruby-libs-3.0.7-165.el9_5.x86_64.rpm                                                                  159 kB/s | 3.2 MB     00:20
(128/151): mesa-libEGL-24.2.8-2.el9_6.x86_64.rpm                                                                  57 kB/s | 141 kB     00:02
(129/151): mesa-filesystem-24.2.8-2.el9_6.x86_64.rpm                                                             4.1 kB/s |  11 kB     00:02
(130/151): libvirt-daemon-driver-storage-scsi-10.10.0-7.6.el9_6.x86_64.rpm                                        12 kB/s |  36 kB     00:03
(131/151): libvirt-daemon-driver-qemu-10.10.0-7.6.el9_6.x86_64.rpm                                               208 kB/s | 991 kB     00:04
(132/151): qemu-kvm-tools-9.1.0-15.el9_6.7.x86_64.rpm                                                            133 kB/s | 571 kB     00:04
(133/151): libwayland-server-1.21.0-1.el9.x86_64.rpm                                                              15 kB/s |  41 kB     00:02
(134/151): nbdkit-1.38.5-5.el9_6.x86_64.rpm                                                                      2.9 kB/s | 8.5 kB     00:02
(135/151): libvirt-daemon-driver-storage-10.10.0-7.6.el9_6.x86_64.rpm                                            9.0 kB/s |  28 kB     00:03
(136/151): libxkbcommon-1.0.3-4.el9.x86_64.rpm                                                                    29 kB/s | 132 kB     00:04
(137/151): qemu-kvm-audio-pa-9.1.0-15.el9_6.7.x86_64.rpm                                                          25 kB/s |  73 kB     00:02
(138/151): libvirt-daemon-driver-storage-rbd-10.10.0-7.6.el9_6.x86_64.rpm                                         24 kB/s |  45 kB     00:01
(139/151): librados2-16.2.4-5.el9.x86_64.rpm                                                                      99 kB/s | 3.4 MB     00:35
(140/151): libvirt-daemon-driver-network-10.10.0-7.6.el9_6.x86_64.rpm                                             62 kB/s | 270 kB     00:04
(141/151): qemu-kvm-ui-opengl-9.1.0-15.el9_6.7.x86_64.rpm                                                         27 kB/s |  76 kB     00:02
(142/151): libvirt-client-qemu-10.10.0-7.6.el9_6.x86_64.rpm                                                       26 kB/s |  49 kB     00:01
(143/151): libfdt-1.6.0-7.el9.x86_64.rpm                                                                          15 kB/s |  33 kB     00:02
(144/151): librbd1-16.2.4-5.el9.x86_64.rpm                                                                        96 kB/s | 3.0 MB     00:31
(145/151): libvirt-10.10.0-7.6.el9_6.x86_64.rpm                                                                  9.6 kB/s |  29 kB     00:02
(146/151): edk2-ovmf-20241117-2.el9.noarch.rpm                                                                   115 kB/s | 6.1 MB     00:54
(147/151): nbdkit-basic-filters-1.38.5-5.el9_6.x86_64.rpm                                                         50 kB/s | 308 kB     00:06
(148/151): libxslt-1.1.34-13.el9_6.x86_64.rpm                                                                     42 kB/s | 239 kB     00:05
(149/151): python3-lxml-4.6.5-3.el9.x86_64.rpm                                                                    74 kB/s | 1.2 MB     00:16
(150/151): qemu-kvm-common-9.1.0-15.el9_6.7.x86_64.rpm                                                            73 kB/s | 668 kB     00:09
(151/151): mesa-dri-drivers-24.2.8-2.el9_6.x86_64.rpm                                                            191 kB/s | 9.4 MB     00:50
-------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                            272 kB/s |  80 MB     05:01
OpenNebula Community Edition                                                                                     9.2 kB/s | 3.1 kB     00:00
Importing GPG key 0x906DC27C:
 Userid     : "OpenNebula Repository <contact@opennebula.io>"
 Fingerprint: 0B2D 385C 7C93 04B1 1A03 67B9 05A0 5927 906D C27C
 From       : https://downloads.opennebula.io/repo/repo2.key
Is this ok [y/N]: y
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                         1/1
  Installing       : libvirt-libs-10.10.0-7.6.el9_6.x86_64                                                                                 1/151
  Installing       : ruby-libs-3.0.7-165.el9_5.x86_64                                                                                      2/151
  Installing       : nbdkit-server-1.38.5-5.el9_6.x86_64                                                                                   3/151
  Installing       : libogg-2:1.3.4-6.el9.x86_64                                                                                           4/151
  Installing       : libX11-xcb-1.7.0-11.el9.x86_64                                                                                        5/151
  Installing       : libxshmfence-1.3-10.el9.x86_64                                                                                        6/151
  Installing       : liburing-2.5-1.el9.x86_64                                                                                             7/151
  Installing       : qemu-img-17:9.1.0-15.el9_6.7.x86_64                                                                                   8/151
  Running scriptlet: libvirt-daemon-lock-10.10.0-7.6.el9_6.x86_64                                                                          9/151
  Installing       : libvirt-daemon-lock-10.10.0-7.6.el9_6.x86_64                                                                          9/151
  Running scriptlet: libvirt-daemon-log-10.10.0-7.6.el9_6.x86_64                                                                          10/151
  Installing       : libvirt-daemon-log-10.10.0-7.6.el9_6.x86_64                                                                          10/151
  Installing       : libvirt-client-10.10.0-7.6.el9_6.x86_64                                                                              11/151
  Running scriptlet: libvirt-daemon-common-10.10.0-7.6.el9_6.x86_64                                                                       12/151
  Installing       : libvirt-daemon-common-10.10.0-7.6.el9_6.x86_64                                                                       12/151
  Running scriptlet: libvirt-daemon-driver-nwfilter-10.10.0-7.6.el9_6.x86_64                                                              13/151
  Installing       : libvirt-daemon-driver-nwfilter-10.10.0-7.6.el9_6.x86_64                                                              13/151
  Installing       : passt-0^20250217.ga1e48a0-10.el9_6.x86_64                                                                            14/151
  Running scriptlet: passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                                                                    15/151
  Installing       : passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                                                                    15/151
  Running scriptlet: passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                                                                    15/151
  Installing       : daxctl-libs-78-2.el9.x86_64                                                                                          16/151
  Installing       : ndctl-libs-78-2.el9.x86_64                                                                                           17/151
  Installing       : libxslt-1.1.34-13.el9_6.x86_64                                                                                       18/151
  Installing       : libwayland-server-1.21.0-1.el9.x86_64                                                                                19/151
  Installing       : libepoxy-1.5.5-4.el9.x86_64                                                                                          20/151
  Installing       : libtpms-0.9.1-5.20211126git1ff6fe1f43.el9_6.x86_64                                                                   21/151
  Installing       : libnbd-1.20.3-1.el9.x86_64                                                                                           22/151
  Installing       : augeas-libs-1.14.1-2.el9.x86_64                                                                                      23/151
  Installing       : libglvnd-1:1.3.4-1.el9.x86_64                                                                                        24/151
  Installing       : libnfsidmap-1:2.5.4-34.el9.x86_64                                                                                    25/151
  Installing       : libpciaccess-0.16-7.el9.x86_64                                                                                       26/151
  Installing       : libdrm-2.4.123-2.el9.x86_64                                                                                          27/151
  Installing       : librdmacm-54.0-1.el9.x86_64                                                                                          28/151
  Installing       : augeas-1.14.1-2.el9.x86_64                                                                                           29/151
  Installing       : swtpm-libs-0.8.0-2.el9_4.x86_64                                                                                      30/151
  Installing       : swtpm-0.8.0-2.el9_4.x86_64                                                                                           31/151
  Running scriptlet: swtpm-0.8.0-2.el9_4.x86_64                                                                                           31/151
  Installing       : python3-lxml-4.6.5-3.el9.x86_64                                                                                      32/151
  Installing       : libpmem-1.12.1-1.el9.x86_64                                                                                          33/151
  Running scriptlet: libvirt-daemon-config-nwfilter-10.10.0-7.6.el9_6.x86_64                                                              34/151
  Installing       : libvirt-daemon-config-nwfilter-10.10.0-7.6.el9_6.x86_64                                                              34/151
  Running scriptlet: libvirt-daemon-config-nwfilter-10.10.0-7.6.el9_6.x86_64                                                              34/151
  Running scriptlet: libvirt-daemon-driver-interface-10.10.0-7.6.el9_6.x86_64                                                             35/151
  Installing       : libvirt-daemon-driver-interface-10.10.0-7.6.el9_6.x86_64                                                             35/151
  Running scriptlet: libvirt-daemon-driver-secret-10.10.0-7.6.el9_6.x86_64                                                                36/151
  Installing       : libvirt-daemon-driver-secret-10.10.0-7.6.el9_6.x86_64                                                                36/151
  Installing       : libvirt-daemon-plugin-lockd-10.10.0-7.6.el9_6.x86_64                                                                 37/151
  Installing       : flac-libs-1.3.3-10.el9_2.1.x86_64                                                                                    38/151
  Installing       : libvorbis-1:1.3.7-5.el9.x86_64                                                                                       39/151
  Installing       : nbdkit-curl-plugin-1.38.5-5.el9_6.x86_64                                                                             40/151
  Installing       : nbdkit-ssh-plugin-1.38.5-5.el9_6.x86_64                                                                              41/151
  Installing       : nbdkit-basic-plugins-1.38.5-5.el9_6.x86_64                                                                           42/151
  Installing       : nbdkit-basic-filters-1.38.5-5.el9_6.x86_64                                                                           43/151
  Installing       : ruby-3.0.7-165.el9_5.x86_64                                                                                          44/151
  Installing       : rubygem-bigdecimal-3.0.0-165.el9_5.x86_64                                                                            45/151
  Installing       : rubygem-bundler-2.2.33-165.el9_5.noarch                                                                              46/151
  Installing       : ruby-default-gems-3.0.7-165.el9_5.noarch                                                                             47/151
  Installing       : rubygem-io-console-0.5.7-165.el9_5.x86_64                                                                            48/151
  Installing       : rubygem-psych-3.3.2-165.el9_5.x86_64                                                                                 49/151
  Installing       : rubygems-3.2.33-165.el9_5.noarch                                                                                     50/151
  Installing       : rubygem-json-2.5.1-165.el9_5.x86_64                                                                                  51/151
  Installing       : rubygem-rdoc-6.3.4.1-165.el9_5.noarch                                                                                52/151
  Installing       : rubygem-sqlite3-1.4.2-8.el9.x86_64                                                                                   53/151
  Installing       : opennebula-rubygems-6.10.0.1-1.el9.x86_64                                                                            54/151
  Running scriptlet: opennebula-rubygems-6.10.0.1-1.el9.x86_64                                                                            54/151
  Installing       : rubygem-rexml-3.2.5-165.el9_5.noarch                                                                                 55/151
  Running scriptlet: libvirt-daemon-proxy-10.10.0-7.6.el9_6.x86_64                                                                        56/151
  Installing       : libvirt-daemon-proxy-10.10.0-7.6.el9_6.x86_64                                                                        56/151
  Running scriptlet: libvirt-daemon-10.10.0-7.6.el9_6.x86_64                                                                              57/151
  Installing       : libvirt-daemon-10.10.0-7.6.el9_6.x86_64                                                                              57/151
  Installing       : python3-libvirt-10.10.0-1.el9.x86_64                                                                                 58/151
  Installing       : libfdt-1.6.0-7.el9.x86_64                                                                                            59/151
  Installing       : edk2-ovmf-20241117-2.el9.noarch                                                                                      60/151
  Installing       : mesa-filesystem-24.2.8-2.el9_6.x86_64                                                                                61/151
  Installing       : mdevctl-1.1.0-4.el9.x86_64                                                                                           62/151
  Running scriptlet: libvirt-daemon-driver-nodedev-10.10.0-7.6.el9_6.x86_64                                                               63/151
  Installing       : libvirt-daemon-driver-nodedev-10.10.0-7.6.el9_6.x86_64                                                               63/151
  Installing       : libblkio-1.5.0-1.el9_4.x86_64                                                                                        64/151
  Installing       : seabios-bin-1.16.3-4.el9.noarch                                                                                      65/151
  Installing       : libasyncns-0.8-22.el9.x86_64                                                                                         66/151
  Installing       : libXfixes-5.0.3-16.el9.x86_64                                                                                        67/151
  Installing       : boost-system-1.75.0-10.el9.x86_64                                                                                    68/151
  Installing       : boost-thread-1.75.0-10.el9.x86_64                                                                                    69/151
  Installing       : usbredir-0.13.0-2.el9.x86_64                                                                                         70/151
  Installing       : opus-1.3.1-10.el9.x86_64                                                                                             71/151
  Installing       : gsm-1.0.19-6.el9.x86_64                                                                                              72/151
  Installing       : libsndfile-1.0.31-9.el9.x86_64                                                                                       73/151
  Installing       : pulseaudio-libs-15.0-3.el9.x86_64                                                                                    74/151
  Installing       : boost-iostreams-1.75.0-10.el9.x86_64                                                                                 75/151
  Installing       : librados2-2:16.2.4-5.el9.x86_64                                                                                      76/151
  Running scriptlet: librados2-2:16.2.4-5.el9.x86_64                                                                                      76/151
  Installing       : librbd1-2:16.2.4-5.el9.x86_64                                                                                        77/151
  Running scriptlet: librbd1-2:16.2.4-5.el9.x86_64                                                                                        77/151
  Running scriptlet: unbound-libs-1.16.2-19.el9_6.1.x86_64                                                                                78/151
  Installing       : unbound-libs-1.16.2-19.el9_6.1.x86_64                                                                                78/151
  Running scriptlet: unbound-libs-1.16.2-19.el9_6.1.x86_64                                                                                78/151
Created symlink /etc/systemd/system/timers.target.wants/unbound-anchor.timer → /usr/lib/systemd/system/unbound-anchor.timer.

  Installing       : gnutls-dane-3.8.3-6.el9.x86_64                                                                                       79/151
  Installing       : gnutls-utils-3.8.3-6.el9.x86_64                                                                                      80/151
  Installing       : swtpm-tools-0.8.0-2.el9_4.x86_64                                                                                     81/151
  Installing       : xkeyboard-config-2.33-2.el9.noarch                                                                                   82/151
  Installing       : libxkbcommon-1.0.3-4.el9.x86_64                                                                                      83/151
  Installing       : qemu-kvm-tools-17:9.1.0-15.el9_6.7.x86_64                                                                            84/151
  Installing       : capstone-4.0.2-10.el9.x86_64                                                                                         85/151
  Installing       : ipxe-roms-qemu-20200823-9.git4bd064de.el9.noarch                                                                     86/151
  Installing       : scrub-2.6.1-4.el9.x86_64                                                                                             87/151
  Installing       : qemu-kvm-docs-17:9.1.0-15.el9_6.7.x86_64                                                                             88/151
  Installing       : seavgabios-bin-1.16.3-4.el9.noarch                                                                                   89/151
  Installing       : qemu-kvm-common-17:9.1.0-15.el9_6.7.x86_64                                                                           90/151
  Running scriptlet: qemu-kvm-common-17:9.1.0-15.el9_6.7.x86_64                                                                           90/151
  Installing       : qemu-kvm-device-display-virtio-gpu-17:9.1.0-15.el9_6.7.x86_64                                                        91/151
  Installing       : qemu-kvm-device-display-virtio-gpu-pci-17:9.1.0-15.el9_6.7.x86_64                                                    92/151
  Installing       : qemu-kvm-block-rbd-17:9.1.0-15.el9_6.7.x86_64                                                                        93/151
  Installing       : qemu-kvm-device-usb-redirect-17:9.1.0-15.el9_6.7.x86_64                                                              94/151
  Installing       : qemu-kvm-device-display-virtio-vga-17:9.1.0-15.el9_6.7.x86_64                                                        95/151
  Installing       : qemu-kvm-device-usb-host-17:9.1.0-15.el9_6.7.x86_64                                                                  96/151
  Installing       : qemu-kvm-block-blkio-17:9.1.0-15.el9_6.7.x86_64                                                                      97/151
  Installing       : qemu-kvm-audio-pa-17:9.1.0-15.el9_6.7.x86_64                                                                         98/151
  Running scriptlet: nbdkit-selinux-1.38.5-5.el9_6.noarch                                                                                 99/151
  Installing       : nbdkit-selinux-1.38.5-5.el9_6.noarch                                                                                 99/151
  Running scriptlet: nbdkit-selinux-1.38.5-5.el9_6.noarch                                                                                 99/151
  Installing       : nbdkit-1.38.5-5.el9_6.x86_64                                                                                        100/151
  Installing       : libwayland-client-1.21.0-1.el9.x86_64                                                                               101/151
  Installing       : mesa-libglapi-24.2.8-2.el9_6.x86_64                                                                                 102/151
  Installing       : mesa-libgbm-24.2.8-2.el9_6.x86_64                                                                                   103/151
  Installing       : libglvnd-egl-1:1.3.4-1.el9.x86_64                                                                                   104/151
  Installing       : mesa-libEGL-24.2.8-2.el9_6.x86_64                                                                                   105/151
  Installing       : mesa-dri-drivers-24.2.8-2.el9_6.x86_64                                                                              106/151
  Installing       : virtiofsd-1.13.2-1.el9_6.x86_64                                                                                     107/151
  Installing       : libXxf86vm-1.1.4-18.el9.x86_64                                                                                      108/151
  Installing       : libglvnd-glx-1:1.3.4-1.el9.x86_64                                                                                   109/151
  Installing       : mesa-libGL-24.2.8-2.el9_6.x86_64                                                                                    110/151
  Installing       : qemu-kvm-ui-opengl-17:9.1.0-15.el9_6.7.x86_64                                                                       111/151
  Installing       : qemu-kvm-ui-egl-headless-17:9.1.0-15.el9_6.7.x86_64                                                                 112/151
  Running scriptlet: dnsmasq-2.85-16.el9_4.x86_64                                                                                        113/151
  Installing       : dnsmasq-2.85-16.el9_4.x86_64                                                                                        113/151
  Running scriptlet: dnsmasq-2.85-16.el9_4.x86_64                                                                                        113/151
  Running scriptlet: libvirt-daemon-driver-network-10.10.0-7.6.el9_6.x86_64                                                              114/151
  Installing       : libvirt-daemon-driver-network-10.10.0-7.6.el9_6.x86_64                                                              114/151
  Running scriptlet: libvirt-daemon-driver-network-10.10.0-7.6.el9_6.x86_64                                                              114/151
  Running scriptlet: libvirt-daemon-config-network-10.10.0-7.6.el9_6.x86_64                                                              115/151
  Installing       : libvirt-daemon-config-network-10.10.0-7.6.el9_6.x86_64                                                              115/151
  Running scriptlet: libvirt-daemon-config-network-10.10.0-7.6.el9_6.x86_64                                                              115/151
  Installing       : libslirp-4.4.0-8.el9.x86_64                                                                                         116/151
  Installing       : qemu-kvm-core-17:9.1.0-15.el9_6.7.x86_64                                                                            117/151
  Installing       : zstd-1.5.5-1.el9.x86_64                                                                                             118/151
  Running scriptlet: rpcbind-1.2.6-7.el9.x86_64                                                                                          119/151
  Installing       : rpcbind-1.2.6-7.el9.x86_64                                                                                          119/151
  Running scriptlet: rpcbind-1.2.6-7.el9.x86_64                                                                                          119/151
Created symlink /etc/systemd/system/multi-user.target.wants/rpcbind.service → /usr/lib/systemd/system/rpcbind.service.
Created symlink /etc/systemd/system/sockets.target.wants/rpcbind.socket → /usr/lib/systemd/system/rpcbind.socket.

  Installing       : systemd-container-252-51.el9_6.1.x86_64                                                                             120/151
  Installing       : libev-4.33-6.el9.x86_64                                                                                             121/151
  Installing       : libverto-libev-0.3.2-3.el9.x86_64                                                                                   122/151
  Installing       : gssproxy-0.8.4-7.el9.x86_64                                                                                         123/151
  Running scriptlet: gssproxy-0.8.4-7.el9.x86_64                                                                                         123/151
  Running scriptlet: nfs-utils-1:2.5.4-34.el9.x86_64                                                                                     124/151
  Installing       : nfs-utils-1:2.5.4-34.el9.x86_64                                                                                     124/151
  Running scriptlet: nfs-utils-1:2.5.4-34.el9.x86_64                                                                                     124/151
  Running scriptlet: libvirt-daemon-driver-storage-core-10.10.0-7.6.el9_6.x86_64                                                         125/151
  Installing       : libvirt-daemon-driver-storage-core-10.10.0-7.6.el9_6.x86_64                                                         125/151
  Installing       : libvirt-daemon-driver-storage-mpath-10.10.0-7.6.el9_6.x86_64                                                        126/151
  Installing       : libvirt-daemon-driver-storage-logical-10.10.0-7.6.el9_6.x86_64                                                      127/151
  Installing       : libvirt-daemon-driver-storage-disk-10.10.0-7.6.el9_6.x86_64                                                         128/151
  Installing       : libvirt-daemon-driver-storage-scsi-10.10.0-7.6.el9_6.x86_64                                                         129/151
  Installing       : libvirt-daemon-driver-storage-rbd-10.10.0-7.6.el9_6.x86_64                                                          130/151
  Installing       : python3-ply-3.11-14.el9.0.1.noarch                                                                                  131/151
  Installing       : python3-pycparser-2.20-6.el9.noarch                                                                                 132/151
  Installing       : python3-cffi-1.14.5-5.el9.x86_64                                                                                    133/151
  Installing       : python3-cryptography-36.0.1-4.el9.x86_64                                                                            134/151
  Installing       : libvirt-client-qemu-10.10.0-7.6.el9_6.x86_64                                                                        135/151
  Installing       : device-mapper-multipath-libs-0.8.7-35.el9_6.1.x86_64                                                                136/151
  Installing       : qemu-pr-helper-17:9.1.0-15.el9_6.7.x86_64                                                                           137/151
  Installing       : qemu-kvm-17:9.1.0-15.el9_6.7.x86_64                                                                                 138/151
  Installing       : lzop-1.04-8.el9.x86_64                                                                                              139/151
  Installing       : numad-0.5-37.20150602git.el9.x86_64                                                                                 140/151
  Running scriptlet: numad-0.5-37.20150602git.el9.x86_64                                                                                 140/151
  Running scriptlet: libvirt-daemon-driver-qemu-10.10.0-7.6.el9_6.x86_64                                                                 141/151
  Installing       : libvirt-daemon-driver-qemu-10.10.0-7.6.el9_6.x86_64                                                                 141/151
  Installing       : isns-utils-libs-0.101-4.el9.x86_64                                                                                  142/151
  Installing       : iscsi-initiator-utils-iscsiuio-6.2.1.9-1.gita65a472.el9.x86_64                                                      143/151
  Running scriptlet: iscsi-initiator-utils-iscsiuio-6.2.1.9-1.gita65a472.el9.x86_64                                                      143/151
Created symlink /etc/systemd/system/sockets.target.wants/iscsiuio.socket → /usr/lib/systemd/system/iscsiuio.socket.

  Installing       : iscsi-initiator-utils-6.2.1.9-1.gita65a472.el9.x86_64                                                               144/151
  Running scriptlet: iscsi-initiator-utils-6.2.1.9-1.gita65a472.el9.x86_64                                                               144/151
Created symlink /etc/systemd/system/sysinit.target.wants/iscsi-starter.service → /usr/lib/systemd/system/iscsi-starter.service.
Created symlink /etc/systemd/system/sockets.target.wants/iscsid.socket → /usr/lib/systemd/system/iscsid.socket.
Created symlink /etc/systemd/system/sysinit.target.wants/iscsi-onboot.service → /usr/lib/systemd/system/iscsi-onboot.service.

  Installing       : libvirt-daemon-driver-storage-iscsi-10.10.0-7.6.el9_6.x86_64                                                        145/151
  Installing       : libvirt-daemon-driver-storage-10.10.0-7.6.el9_6.x86_64                                                              146/151
  Installing       : libvirt-10.10.0-7.6.el9_6.x86_64                                                                                    147/151
  Running scriptlet: opennebula-common-onecfg-6.10.0.1-1.el9.noarch                                                                      148/151
  Installing       : opennebula-common-onecfg-6.10.0.1-1.el9.noarch                                                                      148/151
  Running scriptlet: opennebula-common-6.10.0.1-1.el9.noarch                                                                             149/151
  Installing       : opennebula-common-6.10.0.1-1.el9.noarch                                                                             149/151
  Running scriptlet: opennebula-common-6.10.0.1-1.el9.noarch                                                                             149/151
  Installing       : opennebula-node-kvm-6.10.0.1-1.el9.noarch                                                                           150/151
  Running scriptlet: opennebula-node-kvm-6.10.0.1-1.el9.noarch                                                                           150/151
  Installing       : sssd-nfs-idmap-2.9.6-4.el9_6.2.x86_64                                                                               151/151
  Running scriptlet: libvirt-daemon-lock-10.10.0-7.6.el9_6.x86_64                                                                        151/151
Created symlink /etc/systemd/system/sockets.target.wants/virtlockd.socket → /usr/lib/systemd/system/virtlockd.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtlockd-admin.socket → /usr/lib/systemd/system/virtlockd-admin.socket.

  Running scriptlet: libvirt-daemon-log-10.10.0-7.6.el9_6.x86_64                                                                         151/151
Created symlink /etc/systemd/system/sockets.target.wants/virtlogd.socket → /usr/lib/systemd/system/virtlogd.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtlogd-admin.socket → /usr/lib/systemd/system/virtlogd-admin.socket.

  Running scriptlet: libvirt-daemon-common-10.10.0-7.6.el9_6.x86_64                                                                      151/151
  Running scriptlet: libvirt-daemon-driver-nwfilter-10.10.0-7.6.el9_6.x86_64                                                             151/151
Created symlink /etc/systemd/system/sockets.target.wants/virtnwfilterd.socket → /usr/lib/systemd/system/virtnwfilterd.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtnwfilterd-ro.socket → /usr/lib/systemd/system/virtnwfilterd-ro.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtnwfilterd-admin.socket → /usr/lib/systemd/system/virtnwfilterd-admin.socket.

  Running scriptlet: passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                                                                   151/151
  Running scriptlet: swtpm-0.8.0-2.el9_4.x86_64                                                                                          151/151
  Running scriptlet: libvirt-daemon-config-nwfilter-10.10.0-7.6.el9_6.x86_64                                                             151/151
  Running scriptlet: libvirt-daemon-driver-interface-10.10.0-7.6.el9_6.x86_64                                                            151/151
Created symlink /etc/systemd/system/sockets.target.wants/virtinterfaced.socket → /usr/lib/systemd/system/virtinterfaced.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtinterfaced-ro.socket → /usr/lib/systemd/system/virtinterfaced-ro.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtinterfaced-admin.socket → /usr/lib/systemd/system/virtinterfaced-admin.socket.

  Running scriptlet: libvirt-daemon-driver-secret-10.10.0-7.6.el9_6.x86_64                                                               151/151
Created symlink /etc/systemd/system/sockets.target.wants/virtsecretd.socket → /usr/lib/systemd/system/virtsecretd.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtsecretd-ro.socket → /usr/lib/systemd/system/virtsecretd-ro.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtsecretd-admin.socket → /usr/lib/systemd/system/virtsecretd-admin.socket.

  Running scriptlet: libvirt-daemon-proxy-10.10.0-7.6.el9_6.x86_64                                                                       151/151
Created symlink /etc/systemd/system/sockets.target.wants/virtproxyd.socket → /usr/lib/systemd/system/virtproxyd.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtproxyd-ro.socket → /usr/lib/systemd/system/virtproxyd-ro.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtproxyd-admin.socket → /usr/lib/systemd/system/virtproxyd-admin.socket.

  Running scriptlet: libvirt-daemon-10.10.0-7.6.el9_6.x86_64                                                                             151/151
  Running scriptlet: libvirt-daemon-driver-nodedev-10.10.0-7.6.el9_6.x86_64                                                              151/151
Created symlink /etc/systemd/system/sockets.target.wants/virtnodedevd.socket → /usr/lib/systemd/system/virtnodedevd.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtnodedevd-ro.socket → /usr/lib/systemd/system/virtnodedevd-ro.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtnodedevd-admin.socket → /usr/lib/systemd/system/virtnodedevd-admin.socket.

  Running scriptlet: nbdkit-selinux-1.38.5-5.el9_6.noarch                                                                                151/151
  Running scriptlet: libvirt-daemon-driver-network-10.10.0-7.6.el9_6.x86_64                                                              151/151
Created symlink /etc/systemd/system/sockets.target.wants/virtnetworkd.socket → /usr/lib/systemd/system/virtnetworkd.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtnetworkd-ro.socket → /usr/lib/systemd/system/virtnetworkd-ro.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtnetworkd-admin.socket → /usr/lib/systemd/system/virtnetworkd-admin.socket.

  Running scriptlet: libvirt-daemon-config-network-10.10.0-7.6.el9_6.x86_64                                                              151/151
  Running scriptlet: libvirt-daemon-driver-storage-core-10.10.0-7.6.el9_6.x86_64                                                         151/151
Created symlink /etc/systemd/system/sockets.target.wants/virtstoraged.socket → /usr/lib/systemd/system/virtstoraged.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtstoraged-ro.socket → /usr/lib/systemd/system/virtstoraged-ro.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtstoraged-admin.socket → /usr/lib/systemd/system/virtstoraged-admin.socket.

  Running scriptlet: libvirt-daemon-driver-qemu-10.10.0-7.6.el9_6.x86_64                                                                 151/151
Created symlink /etc/systemd/system/sockets.target.wants/virtqemud.socket → /usr/lib/systemd/system/virtqemud.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtqemud-ro.socket → /usr/lib/systemd/system/virtqemud-ro.socket.
Created symlink /etc/systemd/system/sockets.target.wants/virtqemud-admin.socket → /usr/lib/systemd/system/virtqemud-admin.socket.
Created symlink /etc/systemd/system/multi-user.target.wants/virtqemud.service → /usr/lib/systemd/system/virtqemud.service.

  Running scriptlet: sssd-nfs-idmap-2.9.6-4.el9_6.2.x86_64                                                                               151/151
/usr/lib/sysusers.d/libvirt-qemu.conf:1: Conflict with earlier configuration for group 'kvm', ignoring line.

Couldn't write '1' to 'net/bridge/bridge-nf-call-arptables', ignoring: No such file or directory
Couldn't write '1' to 'net/bridge/bridge-nf-call-ip6tables', ignoring: No such file or directory
Couldn't write '1' to 'net/bridge/bridge-nf-call-iptables', ignoring: No such file or directory

  Verifying        : rubygem-sqlite3-1.4.2-8.el9.x86_64                                                                                    1/151
  Verifying        : opennebula-common-6.10.0.1-1.el9.noarch                                                                               2/151
  Verifying        : opennebula-common-onecfg-6.10.0.1-1.el9.noarch                                                                        3/151
  Verifying        : opennebula-node-kvm-6.10.0.1-1.el9.noarch                                                                             4/151
  Verifying        : opennebula-rubygems-6.10.0.1-1.el9.x86_64                                                                             5/151
  Verifying        : python3-pycparser-2.20-6.el9.noarch                                                                                   6/151
  Verifying        : isns-utils-libs-0.101-4.el9.x86_64                                                                                    7/151
  Verifying        : sssd-nfs-idmap-2.9.6-4.el9_6.2.x86_64                                                                                 8/151
  Verifying        : numad-0.5-37.20150602git.el9.x86_64                                                                                   9/151
  Verifying        : iscsi-initiator-utils-6.2.1.9-1.gita65a472.el9.x86_64                                                                10/151
  Verifying        : python3-cryptography-36.0.1-4.el9.x86_64                                                                             11/151
  Verifying        : lzop-1.04-8.el9.x86_64                                                                                               12/151
  Verifying        : librdmacm-54.0-1.el9.x86_64                                                                                          13/151
  Verifying        : device-mapper-multipath-libs-0.8.7-35.el9_6.1.x86_64                                                                 14/151
  Verifying        : python3-ply-3.11-14.el9.0.1.noarch                                                                                   15/151
  Verifying        : iscsi-initiator-utils-iscsiuio-6.2.1.9-1.gita65a472.el9.x86_64                                                       16/151
  Verifying        : libpciaccess-0.16-7.el9.x86_64                                                                                       17/151
  Verifying        : gssproxy-0.8.4-7.el9.x86_64                                                                                          18/151
  Verifying        : ndctl-libs-78-2.el9.x86_64                                                                                           19/151
  Verifying        : libev-4.33-6.el9.x86_64                                                                                              20/151
  Verifying        : libverto-libev-0.3.2-3.el9.x86_64                                                                                    21/151
  Verifying        : python3-cffi-1.14.5-5.el9.x86_64                                                                                     22/151
  Verifying        : nfs-utils-1:2.5.4-34.el9.x86_64                                                                                      23/151
  Verifying        : systemd-container-252-51.el9_6.1.x86_64                                                                              24/151
  Verifying        : rpcbind-1.2.6-7.el9.x86_64                                                                                           25/151
  Verifying        : zstd-1.5.5-1.el9.x86_64                                                                                              26/151
  Verifying        : libnfsidmap-1:2.5.4-34.el9.x86_64                                                                                    27/151
  Verifying        : daxctl-libs-78-2.el9.x86_64                                                                                          28/151
  Verifying        : libslirp-4.4.0-8.el9.x86_64                                                                                          29/151
  Verifying        : libvirt-daemon-plugin-lockd-10.10.0-7.6.el9_6.x86_64                                                                 30/151
  Verifying        : dnsmasq-2.85-16.el9_4.x86_64                                                                                         31/151
  Verifying        : flac-libs-1.3.3-10.el9_2.1.x86_64                                                                                    32/151
  Verifying        : rubygem-json-2.5.1-165.el9_5.x86_64                                                                                  33/151
  Verifying        : libvirt-daemon-proxy-10.10.0-7.6.el9_6.x86_64                                                                        34/151
  Verifying        : libXxf86vm-1.1.4-18.el9.x86_64                                                                                       35/151
  Verifying        : passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                                                                    36/151
  Verifying        : libglvnd-1:1.3.4-1.el9.x86_64                                                                                        37/151
  Verifying        : virtiofsd-1.13.2-1.el9_6.x86_64                                                                                      38/151
  Verifying        : libwayland-client-1.21.0-1.el9.x86_64                                                                                39/151
  Verifying        : libvirt-daemon-10.10.0-7.6.el9_6.x86_64                                                                              40/151
  Verifying        : libvirt-daemon-lock-10.10.0-7.6.el9_6.x86_64                                                                         41/151
  Verifying        : libvirt-libs-10.10.0-7.6.el9_6.x86_64                                                                                42/151
  Verifying        : rubygem-bundler-2.2.33-165.el9_5.noarch                                                                              43/151
  Verifying        : swtpm-0.8.0-2.el9_4.x86_64                                                                                           44/151
  Verifying        : swtpm-tools-0.8.0-2.el9_4.x86_64                                                                                     45/151
  Verifying        : nbdkit-selinux-1.38.5-5.el9_6.noarch                                                                                 46/151
  Verifying        : libvirt-daemon-driver-nodedev-10.10.0-7.6.el9_6.x86_64                                                               47/151
  Verifying        : passt-0^20250217.ga1e48a0-10.el9_6.x86_64                                                                            48/151
  Verifying        : qemu-kvm-block-rbd-17:9.1.0-15.el9_6.7.x86_64                                                                        49/151
  Verifying        : libvirt-daemon-log-10.10.0-7.6.el9_6.x86_64                                                                          50/151
  Verifying        : mesa-libGL-24.2.8-2.el9_6.x86_64                                                                                     51/151
  Verifying        : augeas-libs-1.14.1-2.el9.x86_64                                                                                      52/151
  Verifying        : mesa-libglapi-24.2.8-2.el9_6.x86_64                                                                                  53/151
  Verifying        : libnbd-1.20.3-1.el9.x86_64                                                                                           54/151
  Verifying        : ruby-3.0.7-165.el9_5.x86_64                                                                                          55/151
  Verifying        : boost-thread-1.75.0-10.el9.x86_64                                                                                    56/151
  Verifying        : libtpms-0.9.1-5.20211126git1ff6fe1f43.el9_6.x86_64                                                                   57/151
  Verifying        : seavgabios-bin-1.16.3-4.el9.noarch                                                                                   58/151
  Verifying        : liburing-2.5-1.el9.x86_64                                                                                            59/151
  Verifying        : nbdkit-server-1.38.5-5.el9_6.x86_64                                                                                  60/151
  Verifying        : libvirt-daemon-config-network-10.10.0-7.6.el9_6.x86_64                                                               61/151
  Verifying        : qemu-kvm-docs-17:9.1.0-15.el9_6.7.x86_64                                                                             62/151
  Verifying        : scrub-2.6.1-4.el9.x86_64                                                                                             63/151
  Verifying        : libxshmfence-1.3-10.el9.x86_64                                                                                       64/151
  Verifying        : qemu-kvm-device-display-virtio-gpu-17:9.1.0-15.el9_6.7.x86_64                                                        65/151
  Verifying        : libvirt-daemon-driver-storage-mpath-10.10.0-7.6.el9_6.x86_64                                                         66/151
  Verifying        : rubygem-rdoc-6.3.4.1-165.el9_5.noarch                                                                                67/151
  Verifying        : libvirt-daemon-driver-storage-core-10.10.0-7.6.el9_6.x86_64                                                          68/151
  Verifying        : qemu-kvm-device-usb-redirect-17:9.1.0-15.el9_6.7.x86_64                                                              69/151
  Verifying        : libvirt-daemon-driver-storage-iscsi-10.10.0-7.6.el9_6.x86_64                                                         70/151
  Verifying        : libvirt-client-10.10.0-7.6.el9_6.x86_64                                                                              71/151
  Verifying        : ipxe-roms-qemu-20200823-9.git4bd064de.el9.noarch                                                                     72/151
  Verifying        : libX11-xcb-1.7.0-11.el9.x86_64                                                                                       73/151
  Verifying        : mesa-libgbm-24.2.8-2.el9_6.x86_64                                                                                    74/151
  Verifying        : libglvnd-glx-1:1.3.4-1.el9.x86_64                                                                                    75/151
  Verifying        : libvirt-daemon-driver-nwfilter-10.10.0-7.6.el9_6.x86_64                                                              76/151
  Verifying        : capstone-4.0.2-10.el9.x86_64                                                                                         77/151
  Verifying        : libepoxy-1.5.5-4.el9.x86_64                                                                                          78/151
  Verifying        : libvirt-daemon-driver-storage-logical-10.10.0-7.6.el9_6.x86_64                                                       79/151
  Verifying        : xkeyboard-config-2.33-2.el9.noarch                                                                                   80/151
  Verifying        : nbdkit-curl-plugin-1.38.5-5.el9_6.x86_64                                                                             81/151
  Verifying        : libglvnd-egl-1:1.3.4-1.el9.x86_64                                                                                    82/151
  Verifying        : qemu-kvm-ui-egl-headless-17:9.1.0-15.el9_6.7.x86_64                                                                  83/151
  Verifying        : qemu-kvm-core-17:9.1.0-15.el9_6.7.x86_64                                                                             84/151
  Verifying        : rubygems-3.2.33-165.el9_5.noarch                                                                                     85/151
  Verifying        : qemu-kvm-17:9.1.0-15.el9_6.7.x86_64                                                                                  86/151
  Verifying        : qemu-pr-helper-17:9.1.0-15.el9_6.7.x86_64                                                                            87/151
  Verifying        : libpmem-1.12.1-1.el9.x86_64                                                                                          88/151
  Verifying        : gnutls-dane-3.8.3-6.el9.x86_64                                                                                       89/151
  Verifying        : pulseaudio-libs-15.0-3.el9.x86_64                                                                                    90/151
  Verifying        : augeas-1.14.1-2.el9.x86_64                                                                                           91/151
  Verifying        : swtpm-libs-0.8.0-2.el9_4.x86_64                                                                                      92/151
  Verifying        : libvirt-daemon-common-10.10.0-7.6.el9_6.x86_64                                                                       93/151
  Verifying        : qemu-img-17:9.1.0-15.el9_6.7.x86_64                                                                                  94/151
  Verifying        : libvirt-daemon-driver-storage-disk-10.10.0-7.6.el9_6.x86_64                                                          95/151
  Verifying        : rubygem-bigdecimal-3.0.0-165.el9_5.x86_64                                                                            96/151
  Verifying        : unbound-libs-1.16.2-19.el9_6.1.x86_64                                                                                97/151
  Verifying        : nbdkit-ssh-plugin-1.38.5-5.el9_6.x86_64                                                                              98/151
  Verifying        : libdrm-2.4.123-2.el9.x86_64                                                                                          99/151
  Verifying        : qemu-kvm-device-display-virtio-vga-17:9.1.0-15.el9_6.7.x86_64                                                       100/151
  Verifying        : boost-iostreams-1.75.0-10.el9.x86_64                                                                                101/151
  Verifying        : gsm-1.0.19-6.el9.x86_64                                                                                             102/151
  Verifying        : ruby-default-gems-3.0.7-165.el9_5.noarch                                                                            103/151
  Verifying        : rubygem-io-console-0.5.7-165.el9_5.x86_64                                                                           104/151
  Verifying        : opus-1.3.1-10.el9.x86_64                                                                                            105/151
  Verifying        : libvirt-daemon-driver-interface-10.10.0-7.6.el9_6.x86_64                                                            106/151
  Verifying        : usbredir-0.13.0-2.el9.x86_64                                                                                        107/151
  Verifying        : rubygem-rexml-3.2.5-165.el9_5.noarch                                                                                108/151
  Verifying        : libogg-2:1.3.4-6.el9.x86_64                                                                                         109/151
  Verifying        : boost-system-1.75.0-10.el9.x86_64                                                                                   110/151
  Verifying        : rubygem-psych-3.3.2-165.el9_5.x86_64                                                                                111/151
  Verifying        : libXfixes-5.0.3-16.el9.x86_64                                                                                       112/151
  Verifying        : qemu-kvm-device-usb-host-17:9.1.0-15.el9_6.7.x86_64                                                                 113/151
  Verifying        : qemu-kvm-device-display-virtio-gpu-pci-17:9.1.0-15.el9_6.7.x86_64                                                   114/151
  Verifying        : libasyncns-0.8-22.el9.x86_64                                                                                        115/151
  Verifying        : libvorbis-1:1.3.7-5.el9.x86_64                                                                                      116/151
  Verifying        : seabios-bin-1.16.3-4.el9.noarch                                                                                     117/151
  Verifying        : nbdkit-basic-plugins-1.38.5-5.el9_6.x86_64                                                                          118/151
  Verifying        : gnutls-utils-3.8.3-6.el9.x86_64                                                                                     119/151
  Verifying        : libvirt-daemon-config-nwfilter-10.10.0-7.6.el9_6.x86_64                                                             120/151
  Verifying        : ruby-libs-3.0.7-165.el9_5.x86_64                                                                                    121/151
  Verifying        : qemu-kvm-block-blkio-17:9.1.0-15.el9_6.7.x86_64                                                                     122/151
  Verifying        : libvirt-daemon-driver-secret-10.10.0-7.6.el9_6.x86_64                                                               123/151
  Verifying        : libblkio-1.5.0-1.el9_4.x86_64                                                                                       124/151
  Verifying        : mdevctl-1.1.0-4.el9.x86_64                                                                                          125/151
  Verifying        : libsndfile-1.0.31-9.el9.x86_64                                                                                      126/151
  Verifying        : python3-libvirt-10.10.0-1.el9.x86_64                                                                                127/151
  Verifying        : mesa-libEGL-24.2.8-2.el9_6.x86_64                                                                                   128/151
  Verifying        : libvirt-daemon-driver-qemu-10.10.0-7.6.el9_6.x86_64                                                                 129/151
  Verifying        : mesa-filesystem-24.2.8-2.el9_6.x86_64                                                                               130/151
  Verifying        : libvirt-daemon-driver-storage-scsi-10.10.0-7.6.el9_6.x86_64                                                         131/151
  Verifying        : qemu-kvm-tools-17:9.1.0-15.el9_6.7.x86_64                                                                           132/151
  Verifying        : edk2-ovmf-20241117-2.el9.noarch                                                                                     133/151
  Verifying        : libwayland-server-1.21.0-1.el9.x86_64                                                                               134/151
  Verifying        : nbdkit-1.38.5-5.el9_6.x86_64                                                                                        135/151
  Verifying        : librados2-2:16.2.4-5.el9.x86_64                                                                                     136/151
  Verifying        : libvirt-daemon-driver-storage-10.10.0-7.6.el9_6.x86_64                                                              137/151
  Verifying        : libxkbcommon-1.0.3-4.el9.x86_64                                                                                     138/151
  Verifying        : qemu-kvm-audio-pa-17:9.1.0-15.el9_6.7.x86_64                                                                        139/151
  Verifying        : libvirt-daemon-driver-storage-rbd-10.10.0-7.6.el9_6.x86_64                                                          140/151
  Verifying        : librbd1-2:16.2.4-5.el9.x86_64                                                                                       141/151
  Verifying        : libvirt-daemon-driver-network-10.10.0-7.6.el9_6.x86_64                                                              142/151
  Verifying        : qemu-kvm-ui-opengl-17:9.1.0-15.el9_6.7.x86_64                                                                       143/151
  Verifying        : libvirt-client-qemu-10.10.0-7.6.el9_6.x86_64                                                                        144/151
  Verifying        : libfdt-1.6.0-7.el9.x86_64                                                                                           145/151
  Verifying        : libvirt-10.10.0-7.6.el9_6.x86_64                                                                                    146/151
  Verifying        : mesa-dri-drivers-24.2.8-2.el9_6.x86_64                                                                              147/151
  Verifying        : nbdkit-basic-filters-1.38.5-5.el9_6.x86_64                                                                          148/151
  Verifying        : python3-lxml-4.6.5-3.el9.x86_64                                                                                     149/151
  Verifying        : libxslt-1.1.34-13.el9_6.x86_64                                                                                      150/151
  Verifying        : qemu-kvm-common-17:9.1.0-15.el9_6.7.x86_64                                                                          151/151

Installed:
  augeas-1.14.1-2.el9.x86_64                                               augeas-libs-1.14.1-2.el9.x86_64
  boost-iostreams-1.75.0-10.el9.x86_64                                     boost-system-1.75.0-10.el9.x86_64
  boost-thread-1.75.0-10.el9.x86_64                                        capstone-4.0.2-10.el9.x86_64
  daxctl-libs-78-2.el9.x86_64                                              device-mapper-multipath-libs-0.8.7-35.el9_6.1.x86_64
  dnsmasq-2.85-16.el9_4.x86_64                                             edk2-ovmf-20241117-2.el9.noarch
  flac-libs-1.3.3-10.el9_2.1.x86_64                                        gnutls-dane-3.8.3-6.el9.x86_64
  gnutls-utils-3.8.3-6.el9.x86_64                                          gsm-1.0.19-6.el9.x86_64
  gssproxy-0.8.4-7.el9.x86_64                                              ipxe-roms-qemu-20200823-9.git4bd064de.el9.noarch
  iscsi-initiator-utils-6.2.1.9-1.gita65a472.el9.x86_64                    iscsi-initiator-utils-iscsiuio-6.2.1.9-1.gita65a472.el9.x86_64
  isns-utils-libs-0.101-4.el9.x86_64                                       libX11-xcb-1.7.0-11.el9.x86_64
  libXfixes-5.0.3-16.el9.x86_64                                            libXxf86vm-1.1.4-18.el9.x86_64
  libasyncns-0.8-22.el9.x86_64                                             libblkio-1.5.0-1.el9_4.x86_64
  libdrm-2.4.123-2.el9.x86_64                                              libepoxy-1.5.5-4.el9.x86_64
  libev-4.33-6.el9.x86_64                                                  libfdt-1.6.0-7.el9.x86_64
  libglvnd-1:1.3.4-1.el9.x86_64                                            libglvnd-egl-1:1.3.4-1.el9.x86_64
  libglvnd-glx-1:1.3.4-1.el9.x86_64                                        libnbd-1.20.3-1.el9.x86_64
  libnfsidmap-1:2.5.4-34.el9.x86_64                                        libogg-2:1.3.4-6.el9.x86_64
  libpciaccess-0.16-7.el9.x86_64                                           libpmem-1.12.1-1.el9.x86_64
  librados2-2:16.2.4-5.el9.x86_64                                          librbd1-2:16.2.4-5.el9.x86_64
  librdmacm-54.0-1.el9.x86_64                                              libslirp-4.4.0-8.el9.x86_64
  libsndfile-1.0.31-9.el9.x86_64                                           libtpms-0.9.1-5.20211126git1ff6fe1f43.el9_6.x86_64
  liburing-2.5-1.el9.x86_64                                                libverto-libev-0.3.2-3.el9.x86_64
  libvirt-10.10.0-7.6.el9_6.x86_64                                         libvirt-client-10.10.0-7.6.el9_6.x86_64
  libvirt-client-qemu-10.10.0-7.6.el9_6.x86_64                             libvirt-daemon-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-common-10.10.0-7.6.el9_6.x86_64                           libvirt-daemon-config-network-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-config-nwfilter-10.10.0-7.6.el9_6.x86_64                  libvirt-daemon-driver-interface-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-driver-network-10.10.0-7.6.el9_6.x86_64                   libvirt-daemon-driver-nodedev-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-driver-nwfilter-10.10.0-7.6.el9_6.x86_64                  libvirt-daemon-driver-qemu-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-driver-secret-10.10.0-7.6.el9_6.x86_64                    libvirt-daemon-driver-storage-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-driver-storage-core-10.10.0-7.6.el9_6.x86_64              libvirt-daemon-driver-storage-disk-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-driver-storage-iscsi-10.10.0-7.6.el9_6.x86_64             libvirt-daemon-driver-storage-logical-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-driver-storage-mpath-10.10.0-7.6.el9_6.x86_64             libvirt-daemon-driver-storage-rbd-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-driver-storage-scsi-10.10.0-7.6.el9_6.x86_64              libvirt-daemon-lock-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-log-10.10.0-7.6.el9_6.x86_64                              libvirt-daemon-plugin-lockd-10.10.0-7.6.el9_6.x86_64
  libvirt-daemon-proxy-10.10.0-7.6.el9_6.x86_64                            libvirt-libs-10.10.0-7.6.el9_6.x86_64
  libvorbis-1:1.3.7-5.el9.x86_64                                           libwayland-client-1.21.0-1.el9.x86_64
  libwayland-server-1.21.0-1.el9.x86_64                                    libxkbcommon-1.0.3-4.el9.x86_64
  libxshmfence-1.3-10.el9.x86_64                                           libxslt-1.1.34-13.el9_6.x86_64
  lzop-1.04-8.el9.x86_64                                                   mdevctl-1.1.0-4.el9.x86_64
  mesa-dri-drivers-24.2.8-2.el9_6.x86_64                                   mesa-filesystem-24.2.8-2.el9_6.x86_64
  mesa-libEGL-24.2.8-2.el9_6.x86_64                                        mesa-libGL-24.2.8-2.el9_6.x86_64
  mesa-libgbm-24.2.8-2.el9_6.x86_64                                        mesa-libglapi-24.2.8-2.el9_6.x86_64
  nbdkit-1.38.5-5.el9_6.x86_64                                             nbdkit-basic-filters-1.38.5-5.el9_6.x86_64
  nbdkit-basic-plugins-1.38.5-5.el9_6.x86_64                               nbdkit-curl-plugin-1.38.5-5.el9_6.x86_64
  nbdkit-selinux-1.38.5-5.el9_6.noarch                                     nbdkit-server-1.38.5-5.el9_6.x86_64
  nbdkit-ssh-plugin-1.38.5-5.el9_6.x86_64                                  ndctl-libs-78-2.el9.x86_64
  nfs-utils-1:2.5.4-34.el9.x86_64                                          numad-0.5-37.20150602git.el9.x86_64
  opennebula-common-6.10.0.1-1.el9.noarch                                  opennebula-common-onecfg-6.10.0.1-1.el9.noarch
  opennebula-node-kvm-6.10.0.1-1.el9.noarch                                opennebula-rubygems-6.10.0.1-1.el9.x86_64
  opus-1.3.1-10.el9.x86_64                                                 passt-0^20250217.ga1e48a0-10.el9_6.x86_64
  passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                        pulseaudio-libs-15.0-3.el9.x86_64
  python3-cffi-1.14.5-5.el9.x86_64                                         python3-cryptography-36.0.1-4.el9.x86_64
  python3-libvirt-10.10.0-1.el9.x86_64                                     python3-lxml-4.6.5-3.el9.x86_64
  python3-ply-3.11-14.el9.0.1.noarch                                       python3-pycparser-2.20-6.el9.noarch
  qemu-img-17:9.1.0-15.el9_6.7.x86_64                                      qemu-kvm-17:9.1.0-15.el9_6.7.x86_64
  qemu-kvm-audio-pa-17:9.1.0-15.el9_6.7.x86_64                             qemu-kvm-block-blkio-17:9.1.0-15.el9_6.7.x86_64
  qemu-kvm-block-rbd-17:9.1.0-15.el9_6.7.x86_64                            qemu-kvm-common-17:9.1.0-15.el9_6.7.x86_64
  qemu-kvm-core-17:9.1.0-15.el9_6.7.x86_64                                 qemu-kvm-device-display-virtio-gpu-17:9.1.0-15.el9_6.7.x86_64
  qemu-kvm-device-display-virtio-gpu-pci-17:9.1.0-15.el9_6.7.x86_64        qemu-kvm-device-display-virtio-vga-17:9.1.0-15.el9_6.7.x86_64
  qemu-kvm-device-usb-host-17:9.1.0-15.el9_6.7.x86_64                      qemu-kvm-device-usb-redirect-17:9.1.0-15.el9_6.7.x86_64
  qemu-kvm-docs-17:9.1.0-15.el9_6.7.x86_64                                 qemu-kvm-tools-17:9.1.0-15.el9_6.7.x86_64
  qemu-kvm-ui-egl-headless-17:9.1.0-15.el9_6.7.x86_64                      qemu-kvm-ui-opengl-17:9.1.0-15.el9_6.7.x86_64
  qemu-pr-helper-17:9.1.0-15.el9_6.7.x86_64                                rpcbind-1.2.6-7.el9.x86_64
  ruby-3.0.7-165.el9_5.x86_64                                              ruby-default-gems-3.0.7-165.el9_5.noarch
  ruby-libs-3.0.7-165.el9_5.x86_64                                         rubygem-bigdecimal-3.0.0-165.el9_5.x86_64
  rubygem-bundler-2.2.33-165.el9_5.noarch                                  rubygem-io-console-0.5.7-165.el9_5.x86_64
  rubygem-json-2.5.1-165.el9_5.x86_64                                      rubygem-psych-3.3.2-165.el9_5.x86_64
  rubygem-rdoc-6.3.4.1-165.el9_5.noarch                                    rubygem-rexml-3.2.5-165.el9_5.noarch
  rubygem-sqlite3-1.4.2-8.el9.x86_64                                       rubygems-3.2.33-165.el9_5.noarch
  scrub-2.6.1-4.el9.x86_64                                                 seabios-bin-1.16.3-4.el9.noarch
  seavgabios-bin-1.16.3-4.el9.noarch                                       sssd-nfs-idmap-2.9.6-4.el9_6.2.x86_64
  swtpm-0.8.0-2.el9_4.x86_64                                               swtpm-libs-0.8.0-2.el9_4.x86_64
  swtpm-tools-0.8.0-2.el9_4.x86_64                                         systemd-container-252-51.el9_6.1.x86_64
  unbound-libs-1.16.2-19.el9_6.1.x86_64                                    usbredir-0.13.0-2.el9.x86_64
  virtiofsd-1.13.2-1.el9_6.x86_64                                          xkeyboard-config-2.33-2.el9.noarch
  zstd-1.5.5-1.el9.x86_64

Complete!
```

🌞 Dépendances additionnelles

```
[chims@kvm1 ~]$ sudo dnf install -y genisoimage
[sudo] password for chims:
Last metadata expiration check: 0:12:29 ago on Wed Sep 17 14:12:16 2025.
Dependencies resolved.
=================================================================================================================================================
 Package                              Architecture                    Version                                Repository                     Size
=================================================================================================================================================
Installing:
 genisoimage                          x86_64                          1.1.11-48.el9                          epel                          324 k
Installing dependencies:
 libusal                              x86_64                          1.1.11-48.el9                          epel                          137 k

Transaction Summary
=================================================================================================================================================
Install  2 Packages

Total download size: 461 k
Installed size: 1.6 M
Downloading Packages:
(1/2): libusal-1.1.11-48.el9.x86_64.rpm                                                                          142 kB/s | 137 kB     00:00
(2/2): genisoimage-1.1.11-48.el9.x86_64.rpm                                                                      272 kB/s | 324 kB     00:01
-------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                            180 kB/s | 461 kB     00:02
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                         1/1
  Installing       : libusal-1.1.11-48.el9.x86_64                                                                                            1/2
  Installing       : genisoimage-1.1.11-48.el9.x86_64                                                                                        2/2
  Running scriptlet: genisoimage-1.1.11-48.el9.x86_64                                                                                        2/2
  Verifying        : genisoimage-1.1.11-48.el9.x86_64                                                                                        1/2
  Verifying        : libusal-1.1.11-48.el9.x86_64                                                                                            2/2

Installed:
  genisoimage-1.1.11-48.el9.x86_64                                          libusal-1.1.11-48.el9.x86_64

Complete!
```

🌞 Démarrer le service libvirtd

```
[chims@kvm1 ~]$ sudo systemctl enable --now libvirtd
Created symlink /etc/systemd/system/multi-user.target.wants/libvirtd.service → /usr/lib/systemd/system/libvirtd.service.
Created symlink /etc/systemd/system/sockets.target.wants/libvirtd.socket → /usr/lib/systemd/system/libvirtd.socket.
Created symlink /etc/systemd/system/sockets.target.wants/libvirtd-ro.socket → /usr/lib/systemd/system/libvirtd-ro.socket.
Created symlink /etc/systemd/system/sockets.target.wants/libvirtd-admin.socket → /usr/lib/systemd/system/libvirtd-admin.socket.
[chims@kvm1 ~]$
```
B. Système¶
🌞 Ouverture firewall
```
[chims@kvm1 ~]$ sudo firewall --add-port=22/tcp --permanent
sudo: firewall: command not found
[chims@kvm1 ~]$ sudo firewall-cmd --add-port=22/tcp --permanent
success
[chims@kvm1 ~]$ sudo firewall-cmd --add-port=8472/udp --permanent
success
[chims@kvm1 ~]$  sudo firewall-cmd --reload
success
```

🌞 Handle SSH
```
[root@kvm1 ~]# ssh chims@frontend.one
chims@frontend.one's password:
Permission denied, please try again.
chims@frontend.one's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last failed login: Wed Sep 17 15:21:03 CEST 2025 from 10.3.1.11 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Wed Sep 17 15:20:42 2025 from 10.3.1.11
[chims@frontend ~]$
```

🌞 Handle SSH

uniquement pour ce point, repassez en SSH sur frontend.one
```
[root@kvm1 ~]# ssh chims@frontend.one
chims@frontend.one's password:
Permission denied, please try again.
chims@frontend.one's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last failed login: Wed Sep 17 15:25:27 CEST 2025 from 10.3.1.11 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Wed Sep 17 15:21:08 2025 from 10.3.1.11
[chims@frontend ~]$
```

OpenNebula reposant sur des connexions SSH, elles doivent toutes se passer sans interaction humaine (pas de demande d'acceptation d'empreintes, ni de passwords par exemple)
donc, en étant connecté en tant que oneadmin sur frontend.one
une paire de clés SSH a été générée sur l'utilisateur oneadmin (dans le dossier .ssh dans son homedir, comme d'hab)
il faudra déposer la clé publique sur les noeuds KVM

.la clé publique doit être déposée sur l'utilisateur oneadmin qui existe aussi sur les noeuds KVM
à la main ou ssh-copy-id

```
[oneadmin@frontend ~]$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/var/lib/one/.ssh/id_rsa):
/var/lib/one/.ssh/id_rsa already exists.
Overwrite (y/n)? Y
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/lib/one/.ssh/id_rsa
Your public key has been saved in /var/lib/one/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:KGlZcqLhq8C1aNWCS2xFv7rYfV2pTC/rRieu1PDt99k oneadmin@frontend.one
The key's randomart image is:
+---[RSA 4096]----+
|   .             |
|  . .            |
|  ..o.o          |
|..oo.B..         |
| =o+=o..S   .    |
|+ =o+.  ++.+     |
|.=.o   .*o*.     |
|o.o o .. B.. .  o|
|.. o ...+oo.. .oE|
+----[SHA256]-----+
[oneadmin@frontend ~]$ ssh-copy-id oneadmin@kvm1.one
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/var/lib/one/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Enter passphrase for key '/var/lib/one/.ssh/id_ed25519':
oneadmin@kvm1.one's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'oneadmin@kvm1.one'"
and check to make sure that only the key(s) you wanted were added.

[oneadmin@frontend ~]$


```


```
[oneadmin@frontend ~]$ ssh oneadmin@kvm1.one
Last failed login: Wed Sep 17 16:07:38 CEST 2025 from 10.3.1.10 on ssh:notty
There were 2 failed login attempts since the last successful login.
Last login: Wed Sep 17 16:07:29 2025 from 10.3.1.10
[oneadmin@kvm1 ~]$
```

il faudra aussi trust les empreintes des autres serveurs
à la main ou ssh-keyscan

```
[oneadmin@frontend ~]$ ssh-keyscan kvm1.one >> ~/.ssh/known_hosts
# kvm1.one:22 SSH-2.0-OpenSSH_8.7
# kvm1.one:22 SSH-2.0-OpenSSH_8.7
# kvm1.one:22 SSH-2.0-OpenSSH_8.7
# kvm1.one:22 SSH-2.0-OpenSSH_8.7
# kvm1.one:22 SSH-2.0-OpenSSH_8.7
[oneadmin@frontend ~]$
```

```
[root@kvm1 ~]# cat /etc/systemd/system/vxlan.service
[root@kvm1 ~]# 
[root@kvm1 ~]# sudo systemctl start vxlan.service
[root@kvm1 ~]# sudo systemctl daemon-reload
[root@kvm1 ~]# sudo systemctl enable vxlan.service
Created symlink /etc/systemd/system/multi-user.target.wants/vxlan.service → /etc/systemd/system/vxlan.service.
[root@kvm1 ~]# sudo systemctl start vxlan.service
[root@kvm1 ~]#
```
1. Ajout d'un noeud¶
🌞 Setup de kvm2.one, à l'identique de kvm1.one excepté :

une autre IP statique bien sûr
idem, pour le bridge, donnez-lui l'IP 10.220.220.202/24 (celle qui est juste après l'IP du bridge de kvm1)
```
4: vxlan_bridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether b6:07:0c:95:27:ea brd ff:ff:ff:ff:ff:ff
    inet 10.220.220.202/24 scope global vxlan_bridge
       valid_lft forever preferred_lft forever
    inet6 fe80::b407:cff:fe95:27ea/64 scope link
       valid_lft forever preferred_lft forever
```
