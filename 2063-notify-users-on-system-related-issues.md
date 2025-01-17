# 206.3. Notify users on system-related issues

### **206.3 Notify users on system-related issues**

**Weight:** 1

**Description: **Candidates should be able to notify the users about current issues related to the system.

**Key Knowledge Areas:**

* ​Automate communication with users through logon messages
* Inform active users of system maintenance

**Terms and Utilities:**

* /etc/issue
* /etc/issue.net
* /etc/motd
* wall
* /sbin/shutdown
*   systemctl

    This lesson is all about the ways we can notifying other users. An unplanned hardware maintenance might be required, system might need to be rebooted, new kernel has been compiled , ...

There are some active and passive ways in linux, to notify other users. By considering this fact that some users might be logged off the story is made more complicated. Lets start with the simplest command.

### wall

wall displays a message, or the contents of a file, or otherwise its standard input, on the terminals of all currently logged in users.(we are using CentOS7):

```
[root@server1 ~]# cat message.txt 
Hello! This is from message.txt!
[root@server1 ~]# cat message.txt | wall
[root@server1 ~]# 
Broadcast message from root@server1 (Tue Feb  6 02:17:35 2018):

Hello! This is from message.txt!
logout

root@server1 ~]$ echo " This is using echo" | wall

Broadcast message from root@server1 (Tue Feb  6 02:23:30 2018):

 This is using echo
```

and from another user terminal point of view, what he/she sees:

```
[user1@server1 ~]$ whoami
user1
[user1@server1 ~]$ 
Broadcast message from root@server1 (Tue Feb  6 02:17:35 2018):

Hello! This is from message.txt!

Broadcast message from root@server1 (Tue Feb  6 02:23:30 2018):

 This is using echo
```

How about other users users who access our server via ssh?

#### shh login banner message

One of the ways to protect and secure SSH logins is by displaying warming message to UN-authorized users or display welcome or informational messages to authorized users.

```
                                                                 #####
                                                                #######
                   @                                            ##O#O##
  ######          @@#                                           #VVVVV#
    ##             #                                          ##  VVV  ##
    ##         @@@   ### ####   ###    ###  ##### ######     #          ##
    ##        @  @#   ###    ##  ##     ##    ###  ##       #            ##
    ##       @   @#   ##     ##  ##     ##      ###         #            ###
    ##          @@#   ##     ##  ##     ##      ###        QQ#           ##Q
    ##       # @@#    ##     ##  ##     ##     ## ##     QQQQQQ#       #QQQQQQ
    ##      ## @@# #  ##     ##  ###   ###    ##   ##    QQQQQQQ#     #QQQQQQQ
  ############  ###  ####   ####   #### ### ##### ######   QQQQQ#######QQQQQ
```

As a system adminitrator it is a good habit to configure a security banners for ssh logins. The banner contains some security warning information or general information.

```
###############################################################
#                 Authorized access only!                     # 
# Disconnect IMMEDIATELY if you are not an authorized user!!! #
#         All actions Will be monitored and recorded          #
###############################################################
```

There are two way to display messages one is using issue.net file and second one is using MOTD file.

* issue.net : Display a banner message before the password login prompt.
* motd : Display a banner message after the user has logged in.

### Display SSH Warning Message to Users Before Login \[/etc/issue.net] :

To display Welcome or Warning message for SSH users before login. We use issue.net file so :

```
[root@server1 ~]# find /etc/ -name issue.net
/etc/issue.net
[root@server1 ~]# cat /etc/issue.net 
\S
Kernel \r on an \m

[root@server1 ~]# echo "This is from /etc/issue.net" >> /etc/issue.net 
[root@server1 ~]# cat /etc/issue.net 
\S
Kernel \r on an \m
This is from /etc/issue.net
```

Now inside `/etc/ssh/sshd_config` file, We need to edit `Banner /some/path` like this:

```
Banner /etc/issue.net
```

Our last job is to restart the SSH daemon to reflect new changes (based on your distro sysv, upstart,systemd):

```
[root@server1 ~]# systemctl restart sshd.service
```

To see the result lets try to ssh to server1 from server2:

```
root@server2:~# ssh root@192.168.10.132
The authenticity of host '192.168.10.132 (192.168.10.132)' can't be established.
ECDSA key fingerprint is SHA256:QtfM2iXh5pxZeFdAUXEBEnRXNSP40MWIhnSYvpOBMoY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.132' (ECDSA) to the list of known hosts.
\S
Kernel \r on an \m
This is from /etc/issue.net
root@192.168.10.132's password:
```

### Display SSH Warning Message to Users After Login \[/etc/motd] :

To display banner messages after user login, we use motd file:

```
[root@server1 ~]# find /etc/ -name motd
/etc/motd
[root@server1 ~]# cat /etc/motd 
[root@server1 ~]# echo "This is from /etc/motd" > /etc/motd 
[root@server1 ~]# cat /etc/motd 
This is from /etc/motd
```

again let get connected to server1 from server2:

```
root@192.168.10.132's password: 
Connection to 192.168.10.132 closed by remote host.
Connection to 192.168.10.132 closed.
root@server2:~# ssh root@192.168.10.132
\S
Kernel \r on an \m
This is from /etc/issue.net
root@192.168.10.132's password: 
Last login: Tue Feb  6 06:16:00 2018
This is from /etc/motd
```

and we are done!

### shutdown

shutdown schedules a time for the system to be powered down. It may be used to halt, power-off or reboot the machine.

| shutdown command examples | Description                   |
| ------------------------- | ----------------------------- |
| shutdown                  |                               |
| shutdown now              |                               |
| shutdown 10:10            | “hh:mm” for hour/minutes      |
| shutdown -p now           | poweroff the machine          |
| shutdown -H now           | halt the machine              |
| shutdown -r10:10          | reboot the machine at 10:10AM |
| shutdown -c               | Cancel the pending shutdown   |

and we can send message while we are using shutdown :

```
root@server1 etc]# shutdown +15 "we goes down after 15 min"
Shutdown scheduled for Tue 2018-02-06 07:52:33 EST, use 'shutdown -c' to cancel.
[root@server1 etc]# 
Broadcast message from root@server1 (Tue 2018-02-06 07:37:33 EST):

we goes down after 15 min
The system is going down for power-off at Tue 2018-02-06 07:52:33 EST!
```

what other users see:

```
Broadcast message from root@server1 (Tue 2018-02-06 07:37:33 EST):

we goes down after 15 min
The system is going down for power-off at Tue 2018-02-06 07:52:33 EST!
```

that's all .

## Congratulation we have done lpic2-201 !!!
