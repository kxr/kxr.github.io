---
layout: post
title: Stand alone Utorrent server x64 (64bit) on Linux without system dependecies
---

Utorrent has released a 64 bit build for Linux platforms (see details here). The release is built on Ubuntu 12.04 (LTS). The problem is that if you try to run it on a distribution like RHEL/CentOS or any older distribution with old libraries, it will fail because of the newer dependencies like glibc 2.14 and libssl 1.0.0 etc.

    ./utserver: /usr/lib64/libssl.so.1.0.0: no version information available (required by ./utserver)
    ./utserver: /usr/lib64/libcrypto.so.1.0.0: no version information available (required by ./utserver)
    ./utserver: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by ./utserver)

Here is a solution to run utorrent server standalone on any 64 bit distribution without using/touching the system's libraries (hence no dependencies). The trick is to collect all the required libraries separately and run utorrent using those in isolation from the system. Here are the steps:

#### Collect the required libraries

The list is not huge, utorrent binary (utserver) only has a few dependecies namely:

    glibc, libgcc, libselinux, libkeyutils, libssl, libcomerr, libkrb5, zlib

So what we are going to do is to collect all these pre-compiled libraries from a newer distribution like Fedora-19-x64 or Ubuntu-12.04-x64 or any distribution that has the required newer libraries, you can collect them from the online software repositories or from where ever you feel comfortable.

I am going to use the safest (not the most convenient) option, get these libraries from Ubuntu server 12.04.2 (x64) distribution CD. I say it is the safest because the distributed utorrent package (Ubuntu 12.04 build x64 (27079)) is built on this version of Ubuntu, so one can be sure that libraries on this distro will meet the requirement (By the way, I tried this method using the packages from Fedora 19 and it worked perfectly fine).

Lets download the Ubuntu server 12.04.2 (x64) CD:

    cd
    wget http://releases.ubuntu.com/12.04.2/ubuntu-12.04.2-server-amd64.iso

#### Mount the CD:

    mkdir /mnt/ubuntu12042
    mount -o loop,ro /root/ubuntu-12.04.2-server-amd64.iso /mnt/ubuntu12042

#### Copy the required packages to the temporary location:

    mkdir /tmp/debs
    cp /mnt/ubuntu12042/pool/main/g/gcc-4.6/libgcc1_4.6.3-1ubuntu5_amd64.deb /tmp/debs/
    cp /mnt/ubuntu12042/pool/main/k/krb5/libkrb5-3_1.10+dfsg~beta1-2ubuntu0.3_amd64.deb /tmp/debs/
    cp /mnt/ubuntu12042/pool/main/e/e2fsprogs/libcomerr2_1.42-1ubuntu2_amd64.deb /tmp/debs/
    cp /mnt/ubuntu12042/pool/main/z/zlib/zlib1g_1.2.3.4.dfsg-3ubuntu4_amd64.deb /tmp/debs/
    cp /mnt/ubuntu12042/pool/main/k/keyutils/libkeyutils1_1.5.2-2_amd64.deb /tmp/debs/
    cp /mnt/ubuntu12042/pool/main/libs/libselinux/libselinux1_2.1.0-4.1ubuntu1_amd64.deb /tmp/debs/
    cp /mnt/ubuntu12042/pool/main/e/eglibc/libc6_2.15-0ubuntu10.3_amd64.deb /tmp/debs/
    cp /mnt/ubuntu12042/pool/main/o/openssl/libssl1.0.0_1.0.1-4ubuntu5.5_amd64.deb /tmp/debs/

#### Extract the packages:

    cd /tmp/debs/
    for package in *.deb; do
    ar p $package data.tar.gz | tar -zx
    done

#### Download utorrent:

    cd
    wget 'http://download.utorrent.com/linux/utorrent-server-3.0-ubuntu-12.04-27079.x64.tar.gz'
    tar -xzf utorrent-server-3.0-ubuntu-12.04-27079.x64.tar.gz
    mv ./utorrent-server-v3_0 /opt/utorrent

#### Copy the libraries from the extracted packages:

    mv /tmp/debs/lib /opt/utorrent/
    #rm -rf /tmp/debs

That's it! Now you have the 64 bit utorrent in /opt/utorrent and all the required libraries in /opt/utorrent/lib. Now you can run utorrent using the following command:

    /opt/utorrent/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 --library-path /opt/utorrent/lib/x86_64-linux-gnu /opt/utorrent/utserver


In the following steps I'll setup a secure, proper and clean environment for utorrent running it with a limited user and with proper configuration, log and pid files etc.


#### Make the required directories in utorrent:

    cd /opt/utorrent
    mkdir conf log pid settings webui data
    mv webui.zip webui/

#### Get the default configuration file:

    wget -O /opt/utorrent/conf/utserver.conf http://kxr.me/blog/uts/utserver.conf.customlibs.ubuntu64

#### Get the control script (to start/stop/reload utorrent)

    wget -O /opt/utorrent/utsctl http://kxr.me/blog/uts/utsctl.customlibs.ubuntu64
    chmod +x /opt/utorrent/utsctl
    ln -s /opt/utorrent/utsctl /usr/local/bin/utsctl

#### Set the owner permissions:

I am using the utorrent user, but you can use any user. If you use a different user, update the shell_user variable in /opt/utorrent/utsctl accordingly

    useradd -d /opt/utorrent
    chown -R utorrent.utorrent /opt/utorrent

There you go, if you did all the steps correctly, you should now be able to start utorrent server by:

    utsctl start

The configuration file is located in:

    /opt/utorrent/conf/utserver.conf

Note: after changing the configuration file do a "utsctl reload" for the new changes to take effect. Stopping and starting will not load the new changes.

Don't forget to set the user name, password and directories in the configuration file. Right now the configuration file we downloaded is set to download every thing in /opt/utorrent/data/ and the login credentials are:

    user: admin
    pass: 

You can use the same exact steps to setup a stand alone 32 bit utorrent server that will run on both 64bit and 32bit systems without requiring any system library/dependency. In order to do so use the 32 bit ubuntu server 12.04.2 CD for the 32 bit pacakges and use the 32 bit utorrent package: Ubuntu 10.10 x86 (27079). Don't forget to set the lib_path variable in /opt/utorrent/utsctl to "/opt/utorrent/lib/i386-linux-gnu"


Suggestions and Comments are welcomed :)

