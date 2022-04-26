# Proxmox-Pritunl-Container
How to install Pritunl on Proxmox 7


## STEP 1 Download Ubuntu Template
Open the Proxmox Host Shell and run:
```
pveam update
pveam download local ubuntu-20.04-standard_20.04-1_amd64.tar.gz
```

## STEP 2 Setup Container
Create a new container by clicking "Create CT" in Proxmox.
	- Uncheck Unprivileged Container
	- Select Template: pveam download local ubuntu-20.04-standard_20.04-1_amd64.tar
	- Use a disk size of 15 GB
	- Dont start container after finishing
	
## STEP 3 edit container config on host
In the host shell edit the configuration file of the container:
```
nano /etc/pve/lxc/{container id}.conf
```

And add the line:
```
lxc.cgroup2.devices.allow: c 10:200 rwm
```
Ctrl + S to save


## STEP 4 create rc-local.service
In container Shell

```
nano /etc/systemd/system/rc-local.service
```

Add lines:
```
[Unit]
 Description=/etc/rc.local Compatibility
 ConditionPathExists=/etc/rc.local

[Service]
 Type=forking
 ExecStart=/etc/rc.local start
 TimeoutSec=0
 StandardOutput=tty
 RemainAfterExit=yes
 SysVStartPriority=99

[Install]
 WantedBy=multi-user.target
```

## STEP 5 create rc.local
In container Shell
```
nano /etc/rc.local
```

Add lines:
```
#!/bin/bash
if ! [ -c /dev/net/tun ]; then
mkdir -p /dev/net
mknod -m 666 /dev/net/tun c 10 200
fi
```

And fix rights with:
```
chmod +x /etc/rc.local
```

And enable serivce rc.local
```
systemctl enable rc-local
```

Reboot container.

## STEP 6 install Pritunl
In container Shell, follow the steps of the pritunl documentation:

```
tee /etc/apt/sources.list.d/mongodb-org-4.4.list << EOF
deb https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse
EOF

tee /etc/apt/sources.list.d/pritunl.list << EOF
deb https://repo.pritunl.com/stable/apt focal main
EOF

apt-get --assume-yes install gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | apt-key add -
apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 7568D9BB55FF9E5287D586017AE645C0CF8E292A
apt-get update
apt-get --assume-yes install pritunl mongodb-org
systemctl start pritunl mongod
systemctl enable pritunl mongod
```

## STEP 7 install Pritunl
In the browser go to the ip of the server and follow installation instruction.
Voila!

