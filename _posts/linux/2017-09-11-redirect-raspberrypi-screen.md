---
title: How to redirect screen to PC from raspberry pi
categories: linux raspberrypi
---

This article is talking about raspbian in raspberry pi and there is lightdm (X Display Manager) in raspbian running by default.

As X system is working over network in nature, so there are many ways to redirect raspberry pi screen and they are as below:

- Point x-client (it is LXDE @ raspberry pi) to x-server (X-Ming or other x-server @ PC) over network
- Forward X11 from x-client (it is LXDE @ raspberry pi) to x-server (X-Ming or other x-server @ PC) via SSH
- Enable XDMCP (X Display Manager Connection Protocol) in raspberry pi
- Enable VNC in raspberry pi
- Install XRDP in raspberry pi

Here I only talk the first two ways !

### Pre-requisite

- Enable lightdm to listen on TCP

lightdm is disable tcp connection by default for security reason, so we need to enable it by login as `pi` and changing the config of `/etc/lightdm/lightdm.conf`

Under `[Seat:*]` section, put `xserver-allow-tcp=true` and it like below:

```
[Seat:*]
xserver-allow-tcp=true
```

After that, restart lightdm by

``` bash
sudo service lightdm restart
```

To verify if lightdm enabled tcp, check the process whether there is a `nolisten` param or not. If everything is fine, we should not find any process under below command:

``` bash
ps -ef | grep lightdm | grep nolisten
```

- Ensure `xauth` is installed

Checking if `xauth` is installed by below command:

``` bash
xauth list

#You should see some output like below
#please noted that 0, 10 is display number and you may get 0 only if X11Forward is not yet enable
raspberrypi/unix:0  MIT-MAGIC-COOKIE-1  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
raspberrypi/unix:10  MIT-MAGIC-COOKIE-1  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Otherwise, please install `xauth` via `apt-get` and then configure `/home/pi/.Xauthority`.

### Point x-client to x-server over network

In PC, open x-server (X-Ming or others) with a display number. Let say 1.
Please noted that you may need set the access control by `xhost +` if it is linux.

In raspberry pi, login as `pi` and change the DISPLAY env variable as below:

``` bash
#export DISPLAY=<pc's IP>:<display number>
export DISPLAY=192.168.0.3:1
```

Now everything is ready and you can start your x-session by command in raspberry pi.

``` bash
/usr/bin/lxsession -s LXDE-pi -e LXDE
```

### Forward X11 from x-client to x-server via SSH

- Configure `sshd` to enable X11 forwarding

By using SSH X11 Forwarding, you need to add below items in `/etc/ssh/sshd_config` from raspberry pi.

```
AllowTcpForwarding yes
X11Forwarding yes
X11DisplayOffset 10
```

After that, you need restart sshd by below command:

``` bash
#sshd service is ssh in raspbian ...
sudo service ssh restart
```

- Checking if ssh set the DISPLAY env variable properly

After enable X11 forwarding in sshd, now you can ssh to raspberry pi (may be using Putty) with enable `X11 Forwarding`.
In linux, it is `ssh -X`. After login as `pi` via ssh, checking if `DISPLAY` env variable has been set by sshd properly.

``` bash
echo $DISPLAY

#It should showing localhost:10.0
#Please don't set the DISPLAY env variable by yourself
#the sshd will handle it for you automatically (if both ssh client & sshd enabled X11 forwarding)
localhost:10.0
```

Otherwise, check the sshd log `/var/log/auth.log` or `/var/log/messages` to see if there is any error like below:

``` bash
sshd[4134]: error: Failed to allocate internet-domain X11 display socket.
```

If it is the case, then revise the `/etc/ssh/sshd_config` with `AddressFamily inet` and restart to try again. Or you may need to use another linux to show the ssh debug message via `ssh -vvv -X raspberrypi`.

- Start your x-session

In PC, open x-server (X-Ming or others) with a display number `0`. Not any other number this time. (Please noted that you may need set the access control by `xhost +` before you ssh to raspberry pi if it is linux).

In `pi` ssh session, type below command to start the x-session. After that, you should see the LXDE screen showing in PC :)

``` bash
/usr/bin/lxsession -s LXDE-pi -e LXDE
```

- Start x-session after `su` to other user like root

In `pi` ssh session, type below command to get magic cookie:

``` bash
xauth list

#You should see some output as below
#Please noted that 10 is the display number and we care the cookie only for display number is 10 (due to X11DisplayOffset)
raspberrypi/unix:10  MIT-MAGIC-COOKIE-1  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Copy the whole line of magic cookie output and then it to your target user like root.

``` bash
#Switch to your target user, here use root
su -l root

#add the magic cookie from pi to root's xauth data
xauth add raspberrypi/unix:10  MIT-MAGIC-COOKIE-1  the-actual-cookie-output-from-pi
```

Set the `DISPLAY` env variable manually (this is required when you switch to other user after ssh only)

``` bash
#10 is X11DisplayOffset
export DISPLAY=localhost:10.0
```

After that, type below command to start the x-session and you should see the LXDE screen showing in PC again.

``` bash
/usr/bin/lxsession -s LXDE-pi -e LXDE
```
