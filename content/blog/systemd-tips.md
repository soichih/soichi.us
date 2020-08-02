---
title: "systemd startup tips"
date: 2020-08-01T18:26:57-04:00
draft: false
---

Some of my docker containers use NFS mounts that are configured in /etc/fstab. During a reboot, I've seen my containers failing to start when docker engine is started before all the mount points are mounted. There is a relatively simple but not too obvious solution to this which involves tweaking systemd rules such as `Requires` and `After`.

Let me start out by pointing out that, docker engine is controlled by the following service file

`/etc/systemd/system/multi-user.target.wants/docker.service`

```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
Requires=docker.socket
After=network.target docker.socket firewalld.service
...

```

By ready through [https://www.freedesktop.org/software/systemd/man/systemd.unit.html](https://www.freedesktop.org/software/systemd/man/systemd.unit.html) It seems that the `Requires` sets up logical relationships between various units; which services start together or die together, etc.  For example, above configuration states that if docker.socket should be started when the docker.service is started, or conversely if docker.socket is stopped, docker.service should also be stopped (yikes!). 

However, the `Requires` by itself does *not* prevent docker.service from starting up when docker.socket is started. They are both started simultaneously (or shutdown simultaneously). To make sure that docker service will start only after docker.socket is fully started, we need to add docker.socket in another directive; `After`. I think you should almost always want to put the dependencies in both Requires and After.

As I said, `Requires` make your service completely dependent of listed services. If docker.socket is shut down, then docker.service will be shut down also. If you want to let a service running even when the depending service shuts down, then you can use `Wants` instead. `Wants` works like `Requires`, it tries to start those depending service when the service is started.  But it doesn't prevent the service from starting up even those depending services fail to start, or gets shut down. The documentation recommends that Wants/After is usually how most units are hooked together, which makes sense.

However, if I know that certain services are started on boot by something else, like the /etc/fstab mount point case, then all I want to do is to make sure that those mount points gets mounted *before* docker engine starts up. Then, all I have to do is to simply list it under `After` and I don't have to worry about `Requires` nor `Wants`. 

Then I can do,

```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
Requires=docker.socket
After=network.target docker.socket firewalld.service mnt-scratch.mount
```

This seems to work, and I've tested a couple of times by rebooting my machine. Docker was started *after* my scratch mount point was mounted.

The `mnt-scratch.mount` here is a systemd unit used to mount the /mnt/scratch specified in my /etc/fstab. But how do I know the name of this unit? I don't quite understand how systemd translates /etc/fstab to registered units, but you can run the following to dump the current list of known units related to your mount point.

```
$ systemctl list-units | grep /mnt/scratch
  mnt-scratch.mount                                                                                         loaded    active mounted   /mnt/scratch
```

This shows that `mnt-scratch.mount` unit is responsible for mounting /mnt/scratch, which is what I used to update the After= statement with.  You can also see the whole dependency tree with `systemctl list-dependencies`.

Now, if I know that /mnt/scratch is *not* automatically mounted or started by other systemd services, then I believe I could add it to `Wants` to make sure that it's mounted when docker starts up.

```
Requires=docker.socket
Wants=mnt-scratch.mount
After=network.target docker.socket firewalld.service mnt-scratch.mount
```

Or.. if I really need that mount point for the whole docker engine to operate, then...

```
Requires=docker.socket mnt-scratch.mount
After=network.target docker.socket firewalld.service mnt-scratch.mount
```

Be aware that, using `Requires` means that if /mnt/scratch is unmounted (via systemd?), the whole docker engine will be terminated.. which may or may not be what you want.

