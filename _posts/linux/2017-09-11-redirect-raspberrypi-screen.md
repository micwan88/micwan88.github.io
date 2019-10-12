---
layout: post
title: How to redirect screen to PC from raspberry pi
categories: linux raspberrypi
---

## How to redirect screen to PC from raspberry pi

This article is talking about raspbian in raspberry pi and there is lightdm (X Display Manager) in raspbian running by default.

As X system is working over network in nature, so there are many ways to redirect raspberry pi screen and they are as below:

- Point x-client (it is LXDE @ raspberry pi) to x-server (X-Ming or other x-server @ PC) over network
- Forward X11 from x-client (it is LXDE @ raspberry pi) to x-server (X-Ming or other x-server @ PC) via SSH
- Enable XDMCP (X Display Manager Connection Protocol) in raspberry pi
- Enable VNC in raspberry pi
- Install XRDP in raspberry pi

Here I only talk the first two ways !

### Pre-requisite

lightdm is disable tcp connection by default for security reason, so we need to enable it by login as `pi` and changing the config of `/etc/lightdm/lightdm.conf`

```
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

In PC, open x-server (X-Ming or others) with a display number `0`. Not any other number this time.
Please noted that you may need set the access control by `xhost +` if it is linux.

Now you can ssh to raspberry pi (may be using Putty) with enable `X11 Forwarding`. In linux, it is `ssh -X`.
After login as `pi` via ssh, you can start your x-session by command in raspberry pi. 
Please noted that there is not necessary to change DISPLAY env variable as ssh will handle it.

``` bash
/usr/bin/lxsession -s LXDE-pi -e LXDE
```