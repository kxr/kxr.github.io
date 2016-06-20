---
layout: post
title: Reverse SSH tunnel manager (remote SSH forwardings)
---

Port forwarding over ssh is a great feature of ssh (a.k.a ssh tunnels). It provide you extra layer of security and accessibility. I happen to use this feature a lot especially the remote ssh port forwarding (a.k.a reverse ssh tunnels). I administer many Linux based systems that don't have any kind of direct access from internet and reverse ssh tunnels come in real handy for accessing these systems remotely. The idea is that I forward the ssh port from these systems to a random remote port on my Linux server (that has a live static IP). So whenever I need to access these systems, I login to my Linux server and ssh to these forwarded ssh ports. (If you don't know how ssh tunnels or port forwarding work have a look at this great article explaining the topic in great detail)


I have written a bash script to manage these remote tunnels. Using this script you can easily forward as many ports as you want to a remote server. It will handle the ssh based reverse tunnel sessions and try its best to retain them. It is easily deployable and configurable. Here is how to use this script:


### 1- Get the script on the source system

This will be run on the system from which you want to create the tunnel to your server. We will clone it from git in a temporary location, copy the script to /opt and make it executeable.

    cd /tmp/
    git clone https://github.com/kxr/rssht.git
    mv /tmp/rssht/rssht /opt/rssht
    chmod +x /opt/rssht

Note: You can put this script where ever you want and rename this script to whatever you want; Just make sure that it can be excecuted by the user you are planning to run it as.


### 2- Edit the script and set the host/port configuration variables:

    REMOTE_HOST=live.host.com   # The remote host to which you want to forward the ports

    REMOTE_SSH_PORT=22          # SSH port of the remote host

    REMOTE_USER=root            # SSH user on the remote host

Note: There are other advance configuration options as well. Check the script for more details.

### 3- Set the ports to forward via the tunnel:

This is the list of ports forwarding, and the format is exactly what you would use in ssh command after the -R switch. This variable is an array and you can have as many values (forwardings) as you want. Suppose you want port 22 (ssh) and port 80 (http) on this system to be forwarded to port 10122 and 10180 respectively on the remote server, then this variable will look something like this:

    REMOTE_FWDS=(   

    10122:localhost:22

    10180:localhost:80

    )

### 4- Setup password less login from this machine to the remote server:

Yes, this script expects a password-less ssh connection to the remote host (if you want to enter password every time, you don't need this script, run the ssh command manually).

Running the script with --help or install as first argument will give you quick instructions (based on the configuration variables you set) on how to setup the password-less ssh login to the remote server and add the script to cron job when required.

    [root@host ~]# /opt/rssht install
    INSTALLATION INSTRUCTIONS:

    # Set the configuration variables and forwardings

    # Make sure you have ssh keys generated
    ssh-keygen

    # Setup password-less login to the remote host
    ssh-copy-id 'live.host.com -l root -p 22'

    # Add the cron job
    echo '*/5 * * * * root /opt/rssht' > /etc/cron.d/rssht_live.host.com


### 5- Run the script:

Once the password less login is setup, simply run the script. It should give you a success message with the pid of the ssh session:

    [root@host ~]# /opt/rssht
    RSSHT to host root@live.host.com:22  started successfully ssh pid: 12779

Once the reverse tunnel is started, running the script again should check the status of the running session:

    [root@host ~]# /opt/rssht
    ssh connection is fine, exiting
    [root@host ~]# /opt/rssht
    ssh connection is fine, exiting

You can pass the stop argument if you want to kill the session and remove the tunnel:

    [root@hplt ~]# /opt/rssht stop
    Stop argument passed, killing ..

Alternatively, if you want the tunnel to be permanent, you can add the cron entry for this script. Running the script with the "install" argument will give you a quick command to enable cron job to run this script every 5 minutes:

    echo '*/5 * * * * root /opt/rssht_1' &gt; /etc/cron.d/rssht_live.host.com


> QUICK TIP: When using the script in persistent/cron mode, there is a configuration variable in the script called REFRESH_SOCKET. You can set it to a non-zero positive number and the script will reset (exit and reconnect) the ssh socket connection if its that many minutes old. For example, if this variable is set to 60, the script will reset the connection ever hour. This could be very useful if the connection is unstable and gets stuck a lot. To disable this feature set the variable to 0.


Each script file has the ability to create tunnel to only one remote server. If you want to create tunnels to more than one server, you can have multiple copies of this script each with its own configuration ( and with different file names e.g: /opt/rssht_2).


Here is the complete script code:

{% gist kxr/b0f1f6b63fef7f20c4f3a225c54366f1 %}
