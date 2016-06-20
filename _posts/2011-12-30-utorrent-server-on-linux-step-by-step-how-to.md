---
layout: post
title: Utorrent server on Linux - step by step howto
---

It has been six years since I migrated to Linux (from windows) and never looked back. I used to use uTorrent on windows as my torrent client. It was small, light-weight and very user friendly. Frankly speaking in these six years of my linux voyage, I never found any alternative to that. Until recently I heard that utorrent is now available for linux. Keeping my hopes high, I gave it a try and it lived up to my expectations. As expected it is free but closed-source, with a little hope from the forums that it might become open-source in the future. But anyways, I am really thankful to the utorrent team for briniging out utorrent for linux.

A couple of things felt missing but still it is  better than the most, infact, all! It is currently an alpha release of version 3.0 build, with only web-based GUI and 32bit release (requires 32bit libraries on x64 systems).

## Here is a complete step by step installation guide:

### Step 0: Pre-Requisites

According to the website:

> System Requirements:  
> - x86 Ubuntu 9.10+, Debian 5+, Fedora 12+  
> - Linux kernel 2.6.13 or newer required  

The utserver binary requires glibc-2.11 or newer. Do check this before continuing. To check your version of glibc:
{% highlight bash %}
rpm -q glibc
{% endhighlight %}

(Note: If you have old libraries, or facing any library related error, I have a different guide for you [HERE]({% post_url 2011-12-27-formerr-in-bind %}) )

I am using a 32 bit Centos 6.2, but this guide should work with most linux distributions. If you are using a x64 distribution, you'll need to install all the 32 bit libraries.

In case of a 32 bit distro, there are hardly any dependencies to be installed. They are installed by default even in the minimal installations. Here is the list of dependencies anyways:

 * glibc &gt;= 2.11
 * libgcc
 * openssl
 * krb5-libs
 * libcom_err
 * zlib
 * keyutils-libs
 * libselinux

If you are using a 32 bit distro:
{% highlight shell %}
yum install glibc libgcc openssl krb5-libs libcom_err zlib keyutils-libs libselinux
{% endhighlight %}

If you are using a 64bit distro, install the 32bit libraries:

{% highlight shell %}
yum install glibc glibc.i[36]86 libgcc libgcc.i[36]86 openssl openssl.i[36]86 krb5-libs krb5-libs.i[36]86 libcom_err libcom_err.i[36]86 zlib zlib.i[36]86 keyutils-libs keyutils-libs.i[36]86 libselinux libselinux.i[36]86
{% endhighlight %}

### Step 1: Make basic directories

I will keep it in /opt (ofcourse you can keep it anywhere you like)

{% highlight shell %}
mkdir /opt/utorrent
mkdir /opt/utorrent/conf
mkdir /opt/utorrent/data
mkdir /opt/utorrent/pid
mkdir /opt/utorrent/webui
mkdir /opt/utorrent/log
{% endhighlight %}

### Step 2: Download µTorrent Server

At the time of this writing, µTorrent Server alpha (3.0 build 25053) was the latest available one. You can find the latest release [here](http://www.utorrent.com/downloads/linux). Download the package to a tmp location, extract and then copy the required files:

{% highlight shell %}
wget -O /tmp/utorrent-server-3.0-25053.tar.gz http://download.utorrent.com/linux/utorrent-server-3.0-25053.tar.gz
tar --directory /tmp -xzf /tmp/utorrent-server-3.0-25053.tar.gz
cp /tmp/utorrent-server-v3_0/utserver /opt/utorrent/
cp /tmp/utorrent-server-v3_0/webui.zip /opt/utorrent/webui/
{% endhighlight %}

### Step 3: Libraries check

Before proceeding further, it will be a good idea to locate any missing libraries.

Use ldd command to find out any missing library:

{% highlight shell %}
ldd -r /opt/utorrent/utserver
{% endhighlight %}


You might get the following error:

> /opt/utorrent/utserver: error while loading shared libraries: libssl.so.0.9.8: cannot open shared object file: No such file or directory

or

> /opt/utorrent/utserver: error while loading shared libraries: libcrypto.so.0.9.8: cannot open shared object file: No such file or directory

If openssl is installed properly, this might be because you have a newer version of the shared library. Creating a softlink should solve the problem.


First find out what version you have:

{% highlight shell %}
find  /*/lib /*lib  -type f -name "libssl.so.*"
find  /*/lib /*lib  -type f -name "libcrypto.so.*"
{% endhighlight %}

This will show you the library you have installed, for example in my case it showed me that I've libssl.so.1.0.0 and libcrypto.so.1.0.0 installed:

{% highlight shell %}
[root@localhost ~]# find  /*/lib /*lib  -type f -name "libssl.so.*"

/usr/lib/libssl.so.1.0.0


[root@localhost ~]# find  /*/lib /*lib  -type f -name "libcrypto.so.*"

/usr/lib/libcrypto.so.1.0.0
{% endhighlight %}

Now simply create a soft link from the one you have to the one required by utserver:

{% highlight shell %}
ln -s /usr/lib/libssl.so.1.0.0 /usr/lib/libssl.so.0.9.8
ln -s /usr/lib/libcrypto.so.1.0.0 /usr/lib/libcrypto.so.0.9.8
{% endhighlight %}

(ofcourse you can always install the correct version of openssl instead of doing this softlink trick)

If every thing is fine you should get output some what like this:

{% highlight shell %}
[root@his apps]$ ldd -r /opt/utorrent/utserver
/opt/utorrent/utserver: /usr/lib/libcrypto.so.0.9.8: no version information available (required by /opt/utorrent/utserver)
/opt/utorrent/utserver: /usr/lib/libssl.so.0.9.8: no version information available (required by /opt/utorrent/utserver)
    linux-gate.so.1 =&gt;  (0x0042c000)
    libssl.so.0.9.8 =&gt; /usr/lib/libssl.so.0.9.8 (0x00cb8000)
    libcrypto.so.0.9.8 =&gt; /usr/lib/libcrypto.so.0.9.8 (0x009dc000)
    libpthread.so.0 =&gt; /lib/libpthread.so.0 (0x00110000)
    libm.so.6 =&gt; /lib/libm.so.6 (0x00800000)
    librt.so.1 =&gt; /lib/librt.so.1 (0x001a2000)
    libgcc_s.so.1 =&gt; /lib/libgcc_s.so.1 (0x00465000)
    libc.so.6 =&gt; /lib/libc.so.6 (0x001ab000)
    /lib/ld-linux.so.2 (0x005a8000)
    libgssapi_krb5.so.2 =&gt; /lib/libgssapi_krb5.so.2 (0x00c25000)
    libkrb5.so.3 =&gt; /lib/libkrb5.so.3 (0x0033b000)
    libcom_err.so.2 =&gt; /lib/libcom_err.so.2 (0x0012b000)
    libk5crypto.so.3 =&gt; /lib/libk5crypto.so.3 (0x00622000)
    libresolv.so.2 =&gt; /lib/libresolv.so.2 (0x00130000)
    libdl.so.2 =&gt; /lib/libdl.so.2 (0x0045b000)
    libz.so.1 =&gt; /lib/libz.so.1 (0x00767000)
    libkrb5support.so.0 =&gt; /lib/libkrb5support.so.0 (0x004be000)
    libkeyutils.so.1 =&gt; /lib/libkeyutils.so.1 (0x00991000)
    libselinux.so.1 =&gt; /lib/libselinux.so.1 (0x0014a000)
{% endhighlight %}

(You can ignore the "no version information" warning). This shows every requirement is fulfilled and utorrent server can now run on this machine. Lets now do some final config tweaks.

### Step 4: Create config file


In theory you can just run the utserver binary (like you did in the previous step), and utorrent server should be ready and useable. You don't necessarily need a config file But it is always a good idea to have a config file in hand, it'll make editing and experimenting with the configurations much easier and give you a lot more options to play with.

I wasn't able to find a complete and comprehensive config file. Users are either not using config file or using it just for a couple of options.

I've created this config file from the documentation and added all the options I found. All the directives are explained in the comments and set to the default value.

Here it is (only dir_root is set to /opt/utorrent/data all other values are default):

{% highlight conf %}
###########################################
###                                     ###
### Utorrent Server v3.0 Config File    ###
###                                     ###
### By Khizer Naeem                     ###
### khizernaeem@gmail.com               ###
### Date: 29/12/2011                    ###
###                                     ###
###########################################
###########################################
# Note: - Don't use quotes or double quotes when giving a value
#       - Don't add trailing / when specifying any directory

#####################
## Regular Settings #
#####################

#bind_port (integer):
#    Default value: 6881. Port used for BitTorrent protocol. This can be any value in the range 1025-65000.
bind_port: 6881

#max_ul_rate (integer):
#    Default value: -1. Maximum total upload rate in kilobytes per second. -1 means unlimited. We recommend setting it to -1.
max_ul_rate: -1

#max_ul_rate_seed (integer):
#    Default value: -1. Maximum per-torrent upload rate when seeding, in kilobytes per second. -1 means unlimited. We recommend setting it to -1.
max_ul_rate_seed: -1

#conns_per_torrent (integer):
#    Default value: 50. Maximum number of connections for a given torrent.
conns_per_torrent: 50

#max_total_connections (integer):
#    Default value: 200. Maximum number of connection opened at the same time.
max_total_connections: 200

#auto_bandwidth_management (boolean):
#    Default value: true. If true, upload bandwidth is automatically throttled in order to not impact other applications using TCP/IP.
auto_bandwidth_management: true

#max_dl_rate (integer):
#    Default value: -1. Maximum total download rate in kilobytes per second. -1 means unlimited. We recommend setting it to -1.
max_dl_rate: -1

#seed_ratio (integer):
#    Default value: 0. Seed ratio in percent (%) (e.g. 100 means 100%). If not 0, seeding will stop after reaching this upload/download ratio.
seed_ratio: 0

#seed_time (integer):
#    Default value: 0. Time after which seeding will stop, in seconds. 0 means seeding won't stop.
seed_time: 0


#####################
# Internal Settings #
#####################

#bind_ip (string):
# IP address to use for socket connections. If not provided, a default IP address will be used. We do not recommend changing this value.

#ut_webui_port (integer):
# Default value: 8080. Port number where the utserver process accepts HTTP RPC API calls to support the µTorrent-compatible HTTP interface.
ut_webui_port: 8080

#token_auth_enable (boolean)
# Default value: true. If true, the µTorrent HTTP interface defends against cross-site request forgeries.
# If false, the µTorrent HTTP interface will not be protected in this manner.
token_auth_enable: true

#dir_root (string):
# Default value: "". If not empty, dir_active, dir_completed, and dir_torrent_files are relative to this directory.
dir_root:/opt/utorrent/data

#dir_active (string):
# Default value: "./". Directory in which currently downloaded data is saved.
# Can be an absolute path or relative to dir_root or the current working directory if dir_root is not defined or an empty string.
#dir_active: ./

#dir_completed (string):
# Default value: "". Directory where completed downloads are stored.
dir_completed:

#dir_download (string):
# Default value: "". Optional directory where completed downloads can be stored, instead of in dir_completed.
# If no value is specified for this setting, the value of dir_completed is used.
# This option can be specified multiple times in the file - once for each directory to be designated as such.
# This option can be used when adding torrents via the µTorrent HTTP interface, not via the SDK interface.
# Use the action list-dirs to obtain a list of download directories from the µTorrent HTTP interface.
# Use the option download_dir to specify which of these directories to use when adding a torrent by URL or file through the µTorrent HTTP interface;
# The index of each entry will be in order in which each entry appears in utserver.conf
dir_download:

#dir_torrent_files (string):
# Default value: "". Directory where torrent files are stored. If the empty string, the value of dir_active is used.
dir_torrent_files:

#dir_temp_files (string):
# Default value: "". Directory where temporary files are stored. If the empty string, the value of dir_active is used.
# Using a separate directory just for temporary files allows for deleting the files in this directory on boot and/or periodically.
# The utserver process creates temporary files with a .utt extension,
# if a value for this setting is specified, the utserver process will delete all files with that extension in that directory at process startup.
# The value should specify a directory, not a symbolic link to a directory.
dir_temp_files:

#dir_autoload (string):
# Default value: "". Directory where torrent files will be recognized and auto-loaded. If the empty string, auto-load is disabled.
dir_autoload:

#dir_autoload_delete (boolean):
# Default value: false. If true, torrent files in the autoload directory will be deleted after being loaded,
# else they will be renamed with an extension of .loaded. The dir_autoload setting must be specified for this setting to have an effect.
dir_autoload_delete: false

#upnp (boolean):
# Default value: true. If true, UPNP functionality for mapping ports is used by utserver. We recommend setting this value to true.
upnp: true

#natpmp (boolean):
# Default value: true. If true, NAT-PMP functionality for mapping ports is used by utserver. We recommend setting this value to true.
natpmp: true

#lsd (boolean):
#    Default value: true. If true, Local Service Discovery is enabled. We recommend setting this value to true.
lsd: true

#dht (boolean):
# Default value: true. If true, Distributed Hash Table extension is enabled. We recommend setting this value to true.
dht: true

#pex (boolean):
# Default value: true. If true, Peer Exchange extension is enabled. We recommend setting this value to true.
pex: true

#rate_limit_local_peers (boolean):
# Default value: false. If true, rate limiting also applies to communications with peers in the local subnet. We recommend setting this value to false.
rate_limit_local_peers: false

#disk_cache_max_size (integer):
# Default value: 0. Maximum amount of memory used by each of the read, write, and piece caches. Value is in megabytes.
# If 0, accepts the SDK's default choices on selecting sizes of disk caches. Maximum value is 512.
disk_cache_max_size: 0

#preferred_interface (string):
# Default value: "". If defined, name of network interface to be preferred,
# when attempting to search among network interfaces for an external IP and hardware address.
# If empty string, preferred interface is ignored.
preferred_interface:

#admin_name (string):
# Default value: "admin". If defined, name that must be supplied (along with the password) when authenticating to the server via the HTTP interface.
# This allows the administrator to define an initial non-default value for this name.
# This value will not be applied from utserver.conf if settings.dat already exists.
admin_name: admin

#admin_password (string):
# Default value: "". If defined, password that must be supplied (along with the name) when authenticating to the server via the HTTP interface.
# This allows the administrator to define an initial non-default value for this password.
# This value will not be applied from utserver.conf if settings.dat already exists.
admin_password:

#logmask (integer):
# Default value: 0. A mask whose bits when set allow certain categories of log messages to be generated.
# The bits (0 - 31) in the value of this setting correspond to a set of internal events and subsystems.
#
#        3 - send have
#        6 - hole punch
#        7 - got bad piece request
#        8 - trace
#        9 - piece picker
#        10 - got bad cancel
#        11 - got bad unchoke
#        12 - got bad piece
#        13 - rss
#        14 - rss error
#        15 - got have
#        16 - got bad have
#        17 - error
#        18 - aggregated
#        19 - disconnect
#        20 - out connect
#        21 - in connect
#        22 - UPnP
#        23 - UPnP error
#        24 - NATPMP
#        25 - NATPMP error
#        26 - metadata finish
#        27 - web UI
#        28 - got bad reject
#        29 - pex
#        30 - peer messages
#        31 - blocked connect
logmask: 0

#dir_request (string):
# Default value: "". Directory where maintenance request files will be recognized, loaded, and deleted.
# If the empty string, maintenance request handling is disabled.
#
# Your software running on your device can create the following files in this directory in order to request the following maintenance procedures.
#
#    If the file c.utmr is created in or moved into this directory,
#    the credentials necessary to access the µTorrent HTTP interface will be reset to username admin and a blank password.
#
#    If the file wipl.utmr is created in or moved into this directory,
#    the IP restriction list that limits the IPs that can use the µTorrent HTTP interface is cleared,
#    so that there will be no restrictions on IP address.
#
#    If the file rcf.utmr is created in or moved into this directory,
#    the server will reload the configuration file. If you always use this method to request a configuration file reload,
#    you can safely change the value of this setting while the server is running.

#ut_webui_dir (string):
# Default value: "". Directory where the web UI file archive webui.zip is stored,
# or which contains a webui subdirectory within which the unarchived web UI files are stored.
# It can be an absolute path or set relative to the current directory.
ut_webui_dir:

#finish_cmd (string), state_cmd (string):
# Default value: "". If defined,
# finish_cmd is a command that will be executed upon completion of each torrent.
# state_cmd is a command that will be executed when a torrent changes state.
# The command is run asynchronously as the same user that runs the server process.
#
#    The server permits substitutions in the command text as follows:
#
#    %F Name of downloaded data file (for single-file torrents)
#    %D directory where torrent data files are saved
#    %N torrent title
#    %S torrent state
#    %P previous state of torrent
#    %L label associated with torrent
#    %T tracker
#    %M status message
#    %I hex-encoded info-hash
#
#    State (%S) and previous state (%P) are integers that have the following values:
#
#    1 (error)
#    2 (checked)
#    3 (paused)
#    4 (super seeding)
#    5 (seeding)
#    6 (downloading)
#    7 (super seeding (forced))
#    8 (seeding (forced))
#    9 (downloading (forced))
#    10 (queued seed)
#    11 (finished)
#    12 (queued)
#    13 (stopped)
finish_cmd:
state_cmd:
{% endhighlight %}

Don't worry most of the lines in the config are comments (only 34 directives in total). I recommend going through it once.

Copy-Paste this configuration into the file utserver.conf in /opt/utorrent/conf/ directory using your favourite text editor like vim or download it directly:

{% highlight shell %}
wget -O /opt/utorrent/conf/utserver.conf http://kxr.me/blog/uts/utserver.conf
{% endhighlight %}

Now you can start the utorrent server with the following command:

{% highlight shell %}
/opt/utorrent/utserver -settingspath "/opt/utorrent/webui/" -configfile "/opt/utorrent/conf/utserver.conf" -logfile "/opt/utorrent/log/ut.log" -pidfile "/opt/utorrent/pid/utserver.pid" -daemon
{% endhighlight %}

and stop it with:

{% highlight shell %}
kill `cat /opt/utorrent/pid/utserver.pid`
{% endhighlight %}

That is not very convenient, so lets create a script to make things easier.


{% highlight shell %}
#!/bin/sh

uts_bin="/opt/utorrent/utserver"
pid_file="/opt/utorrent/pid/utserver.pid"
settings_path="/opt/utorrent/webui/"
config_file="/opt/utorrent/conf/utserver.conf"
log_file="/opt/utorrent/log/ut.log"

start(){
        if [ -s "$pid_file" ]
        then
                kill -s 0 `cat $pid_file` &gt; /dev/null 2&gt;&amp;1
                if  [ "$?" == "0" ]
                then
                        echo "Err: Utorrent seems to be running, PID `cat $pid_file`"
                else
                        echo "Starting Utorrent Server.."
                        cd $settings_path;$uts_bin -settingspath $settings_path -configfile $config_file -logfile $log_file -pidfile $pid_file -daemon
                fi
        else
                echo "Starting Utorrent Server.."
                cd $settings_path;$uts_bin -settingspath $settings_path -configfile $config_file -logfile $log_file -pidfile $pid_file -daemon
        fi
}
stop(){
        if [ -s "$pid_file" ]
        then
                kill -s 0 `cat $pid_file` &gt; /dev/null 2&gt;&amp;1
                if  [ "$?" == "0" ]
                then
                        echo "Stopping Utorrent Server.."
                        kill `cat $pid_file`
                        tail -f /dev/null --pid `cat $pid_file`
                        rm -f $pid_file
                else
                        echo "Err: Utorrent seems to be stopped, PID file $pid_file not found or empty"
                fi
        else
                echo "Err: Utorrent seems to be stopped, PID file $pid_file not found or empty"
        fi
}
status(){
        if [ -s "$pid_file" ]
        then
                kill -s 0 `cat $pid_file` &gt; /dev/null 2&gt;&amp;1
                if  [ "$?" == "0" ]
                then
                        echo "Utorrent seems to be running, PID `cat $pid_file`"
                else
                        echo "PID file present, but no process with PID `cat $pid_file` running"
                fi
        else
                echo "Utorrent seems to be stopped, PID file $pid_file not found or empty"
        fi
}

case "$1" in
        start)
                start
                ;;
        stop)
                stop
                ;;
        reload)
                stop
                rm -vf $settings_path/settings.dat*
                start
                ;;
        status)
                status
                ;;
        *)
                echo "Usage $0 {start|stop|reload|status}"
                ;;
esac
{% endhighlight %}

Create a file called utsctl in /opt/utorrent and copy-paste the code into it using your favourite editor or download it directly:

{% highlight shell %}
wget -O /opt/utorrent/utsctl http://kxr.me/blog/uts/utsctl
{% endhighlight %}

Once you have created this script file, lets make it executeable:

{% highlight shell %}
chmod +x /opt/utorrent/utsctl
{% endhighlight %}

And also make it easily accessible, so that we don't have to type the whole path again and again:

{% highlight shell %}
ln -s /opt/utorrent/utsctl /usr/bin/utsctl
{% endhighlight %}

Now starting stoping and checking the status is as easy as this:

{% highlight shell %}
utsctl start

utsctl stop

utsctl status
{% endhighlight %}

### Step 5: Start µTorrent Server

Lets now start the utorrent server:

{% highlight shell %}
utsctl start
{% endhighlight %}

It should output "server started - using locale en_US.UTF-8" and return back to bash prompt. (Optionally, you can add this command to rc.local to auto-start it at startup)

You should now be able to access the utorrent server gui on localhost:

> http://localhost:8080/gui

or via the ipaddress/hostname if accessing from a remote computer:

> http://1.2.3.4:8080/gui

(don't forget to allow this port in the system's iptable firewall and any other firewall in between)

Here is a screen-shot:
![placeholder]({{ site.baseurl }}public/media/utserver.png "Utorrent Server GUI Screenshot")

#### Playing with the configuration:

When you change a configuration in utserver.conf, just restarting the utserver daemon won't update the changes in WebUI. You'll have to remove the settings.dat files and then start the server again i.e,

{% highlight shell %}
    utsctl stop

    rm -f /opt/utorrent/webui/settings.dat*

    utsctl start
{% endhighlight %}

I've added the "reload" option in the utsctl script that will do all these three things:

{% highlight shell %}
    utsctl reload
{% endhighlight %}

Good Luck :)

Questions/Suggestions/Comments/Corrections/Additions about this HowTo are welcome.
