---
layout: post
title: Logging Shell Commands in Linux
---

If you own/administer a Linux machine or server which is being used by multiple users, at some time you must have felt the need to log and record the commands being run in the shell by the users. This can be useful for auditing, tracking and saving the commands being used.


Now the problem is that the built-in logging mechanism in bash i.e. .bash_history file (or the "history" command) is not reliable. First, the command is not logged immediately it is entered into the shell, 2nd and most importantly if the bash session is terminated or gets disconnected abnormally, all the commands for that session would be lost.

 This means that if the linux system crashes, you will never see the commands that were last run in the active bash sessions when the system crashed. Well in bash's defence this feature is never intended to be used for tracking or auditing, it is just command history keeper that keeps track of you last executed commands in case you press the up-arrow-key or use reverse command search (ctrl + r).

There are many methods to log shell commands, some involve recompiling bash and others involve installing third party key logging software. Both of these options are discouraged and are risky especially on production servers.

Recently I discovered a very quick, safe, neat and efficient way of logging bash commands. I am using it on many production machines and it is doing its job well.


### Step 1:

Use your favourite text editor to open /etc/bashrc and append the following line at the end:

    export PROMPT_COMMAND='RETRN_VAL=$?;logger -p local6.debug "$(whoami) [$$]: $(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//" ) [$RETRN_VAL]"'


### Step 2:

Set the syslogger to trap local6 to a log file by adding this line in the /etc/syslog.conf file:

    local6.*                /var/log/cmdlog.log

and restart

    service syslog restart

THATS ALL!

Logging bash commands is this easy! Just logoff and log back in and all your commands will be logged in /var/log/cmdlog.log from now on. It is easily customizable what you want to log. Let me first explain how this works


### How it works


Well the crucial key behind this logging mechanism is the "PROMPT_COMMAND" variable in bash. According to the bash manual:

> If set, the value is executed as a command prior to issuing each primary prompt.

In easy words(tldp):

> Bash provides an environment variable called PROMPT_COMMAND. The contents of this variable are executed as a regular Bash command just before Bash displays a prompt. 

So once we know that bash will execute the content of this variable, we can do any thing (limited to your imagintion). For logging this is what I am doing:

#### 1- Save the return code in a variable (before it is lost):

    RETRN_VAL=$?

#### 2- Next I am logging the following (using the logger command):

    logger -p local6.debug "$(whoami) [$$]: $(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//" ) [$RETRN_VAL]"

Further breakdown:  

Current user name

    $(whoami)

PID of current Shell (helps differentiate b/w multiple sessions by the same user)

    [$$]

Last executed command, sed is used to remove the command index number and the whitespaces outputted by the history command

    $(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//" )

Add the return code at the end in square brackets

    [$RETRN_VAL]

### Sample Logs:

    Jan  2 15:20:01 testdb1 oratest: oratest [5583]: cd /testdata [0]
    Jan  2 15:20:02 testdb1 oratest: oratest [5583]: ls -ltr [0]
    Jan  2 15:20:15 testdb1 oratest: oratest [5583]: du -sh * [0]
    Jan  2 15:21:56 testdb1 oratest: oratest [5583]: clear [0]
    Jan  2 15:21:59 testdb1 oratest: oratest [5583]: df -h  [0]
    Jan  2 15:28:00 testdb1 oratest: oratest [5583]: uptime [0]
    Jan  2 15:28:09 testdb1 oratest: oratest [5583]: top c [0]
    Jan  3 10:57:18 testdb1 khizer: khizer [22493]: su - [0]
    Jan  3 10:57:26 testdb1 khizer: khizer [22493]: su - [1]
    Jan  3 10:57:31 testdb1 khizer: root [22646]: ulimit -c [0]
    Jan  3 11:20:49 testdb1 khizer: khizer [808]: ping 10.105.22.71 [0]
    Jan  3 11:29:57 testdb1 khizer: root [22646]: crontab -e [0]
    Jan  3 11:30:01 testdb1 khizer: root [22646]: service crond relaod [1]
    Jan  3 11:30:05 testdb1 khizer: root [22646]: service crond reload [0]
    Jan  3 11:30:20 testdb1 khizer: khizer [808]: wq [127]
    Jan  3 11:30:43 testdb1 khizer: khizer [808]: q [127]
    Jan  3 11:37:13 testdb1 khizer: khizer [808]: date [0]
    Jan  3 12:00:05 testdb1 khizer: root [22646]: crontab -e [0]
    Jan  3 12:00:06 testdb1 khizer: root [22646]: service crond reload [0]
    Jan  4 10:42:10 testdb1 oratest: oratest [23732]: rm -rf p_HPUX-IA64.zip rda p11812741_423_HPUX-IA64.zip readme.txt 6249879 [0]
    Jan  4 10:42:12 testdb1 oratest: oratest [23732]: ls -ltr  [0]
    Jan  4 10:42:17 testdb1 oratest: oratest [23732]: mkdir TEST [0]
    Jan  4 10:42:18 testdb1 oratest: oratest [23732]: pwd [0]
    Jan  4 10:43:56 testdb1 oratest: oratest [23732]: vi backup.sql [0]
    Jan  4 10:44:02 testdb1 oratest: oratest [23732]: vi backup.sql [1]
