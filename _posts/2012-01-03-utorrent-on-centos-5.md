---
layout: post
title: Utorrent on Centos 5 (GLIBC_2.11 not found Problem)
---

<div class="message">
    Note: This is a quick quide for those who are facing GLIBC or other library issues with utorrent. If you are not facing these problems or haven't tried yet, You should follow the complete descriptive guide HERE
</div>

If you are using a relatively old distribution (with glibc &lt;= 2.10) like centos 5 for Utorrent (utserver), you must be getting a GLIBC_2.11 not found error:

    ./utserver: /lib32/libc.so.6: version `GLIBC_2.11' not found (required by ./utserver) 

or /lib instead of /lib32 on some distros:

    ./utserver: /lib/libc.so.6: version `GLIBC_2.11' not found (required by ./utserver)

Although it is recommended to use a newer distribution that has glibc &gt;= 2.11, if you some how cannot afford to do that, here is a hack that should work.


The trick is that, if you can provide utserver binary with the required libraries in a custom directory, it should work no matter what libraries your current distribution is using. You don't need to tamper your existing installation and libraries, just download the required libraries in a custom directory and run utserver with those libraries.


Here are the steps:


### 1- Get all the libraries


I am using centos 6.2 (32 bit) rpms to get the libraries, these are highly and easily available. Feel free to gather these libraries from any convenient location you like.

Make sure you have the required tools:

    yum install cpio xz

Download the rpms in a temp location:

    mkdir /tmp/centos6rpms
    cd /tmp/centos6rpms/
    wget http://mirror.crazynetwork.it/centos/6.2/os/i386/Packages/glibc-2.12-1.47.el6.i686.rpm
    wget http://mirror.crazynetwork.it/centos/6.2/os/i386/Packages/libgcc-4.4.6-3.el6.i686.rpm
    wget http://mirror.crazynetwork.it/centos/6.2/os/i386/Packages/openssl-1.0.0-20.el6.i686.rpm
    wget http://mirror.crazynetwork.it/centos/6.2/os/i386/Packages/krb5-libs-1.9-22.el6.i686.rpm
    wget http://mirror.crazynetwork.it/centos/6.2/os/i386/Packages/libcom_err-1.41.12-11.el6.i686.rpm
    wget http://mirror.crazynetwork.it/centos/6.2/os/i386/Packages/zlib-1.2.3-27.el6.i686.rpm
    wget http://mirror.crazynetwork.it/centos/6.2/os/i386/Packages/keyutils-libs-1.4-3.el6.i686.rpm
    wget http://mirror.crazynetwork.it/centos/6.2/os/i386/Packages/libselinux-2.0.94-5.2.el6.i686.rpm

### 2- Extract and place the libraries


I am using cpio to extract files from the rpm. Be carefull, do not attempt to install these rpms, unless you know what you are doing. Installing these rpms will replace/change the current libraries of the running system and can crash your system.

    cd /tmp/centos6rpms/
    rpm2cpio glibc-2.12-1.47.el6.i686.rpm | xz -d | cpio -idmv
    rpm2cpio keyutils-libs-1.4-3.el6.i686.rpm | xz -d | cpio -idmv
    rpm2cpio krb5-libs-1.9-22.el6.i686.rpm | xz -d | cpio -idmv
    rpm2cpio libcom_err-1.41.12-11.el6.i686.rpm | xz -d | cpio -idmv
    rpm2cpio libgcc-4.4.6-3.el6.i686.rpm | xz -d | cpio -idmv
    rpm2cpio libselinux-2.0.94-5.2.el6.i686.rpm | xz -d | cpio -idmv
    rpm2cpio openssl-1.0.0-20.el6.i686.rpm | xz -d | cpio -idmv
    rpm2cpio zlib-1.2.3-27.el6.i686.rpm | xz -d | cpio -idmv

Collect all the libraries to a custom location, i am placing the libraries in /opt/utorrent/lib:

    mkdir -p /opt/utorrent/lib
    cp -av /tmp/centos6rpms/lib/* /opt/utorrent/lib/
    cp -av /tmp/centos6rpms/usr/lib/* /opt/utorrent/lib/
    ln -s /opt/utorrent/lib/libssl.so.1.0.0 /opt/utorrent/lib/libssl.so.0.9.8
    ln -s /opt/utorrent/lib/libcrypto.so.1.0.0 /opt/utorrent/lib/libcrypto.so.0.9.8

You can now remove the temp directory where we downloaded all the rpms initially:

    rm -rf /tmp/centos6rpms

### 3- Setup Utorrent


Create directories for utorrent:

    mkdir -p /opt/utorrent
    mkdir -p /opt/utorrent/conf
    mkdir -p /opt/utorrent/pid
    mkdir -p /opt/utorrent/webui
    mkdir -p /opt/utorrent/data
    mkdir -p /opt/utorrent/log

Get and Extract Utorrent:

    wget -O /tmp/utorrent-server-3.0-25053.tar.gz http://download.utorrent.com/linux/utorrent-server-3.0-25053.tar.gz
    tar --directory /tmp -xzf /tmp/utorrent-server-3.0-25053.tar.gz
    cp /tmp/utorrent-server-v3_0/utserver /opt/utorrent/
    cp /tmp/utorrent-server-v3_0/webui.zip /opt/utorrent/webui/

### 4- Create the configuration

Its up to you, you can create your own configuration file, not use a configuration file or use this default configuration file I've created from the documentation (only root_dir is set to /opt/utorrent/data, all other options are default):

    wget -O /opt/utorrent/conf/utserver.conf http://kxr.me/blog/uts/utserver.conf

At this point your utserver should run, you can test it by running:

    /opt/utorrent/lib/ld-linux.so.2 --library-path /opt/utorrent/lib /opt/utorrent/utserver -settingspath "/opt/utorrent/webui/" -configfile "/opt/utorrent/conf/utserver.conf" -logfile "/opt/utorrent/log/ut.log" -pidfile "/opt/utorrent/pid/utserver.pid"

you should see  "server started" message (ignore no version information warning)


### 5- Create Start/Stop Script

Its up to you, you can create your own init.d/systemd/upstart scripts. I've created this simple bash script to start, stop, reload and check status.

    wget -O /opt/utorrent/utsctl http://kxr.me/blog/uts/utsctl.clib

    chmod +x /opt/utorrent/utsctl

    ln -s /opt/utorrent/utsctl /usr/bin/utsctl


### 6- Start/Stop Utorrent

If you were lucky, starting the utorrent should be as simple:

    utsctl start

You can use this script to start, stop, check status and reload (reload option builds the configuration from the utserver.conf file and restart the utserver).

    utsctl stop

    utsctl status

    utsctl reload
