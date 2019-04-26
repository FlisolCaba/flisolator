# Filsolator J to X again

Implementation of a PXE Stack over Docker 

## Tools That We Use

 -  [docker](https://docker.com/)
 -  [docker-compose](https://docs.docker.com/compose/)
 -  [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html)
 -  [tftpd-hpa](http://www.chschneider.eu/linux/server/tftpd-hpa.shtml)
 -  [nginx](https://nginx.org/)
 -  [nfs](https://en.wikipedia.org/wiki/Network_File_System)
 -  [squid](http://www.squid-cache.org/)
 -  [syslinux](https://repo.or.cz/syslinux.git)
 -  [gparted live](https://gparted.org/livecd.php)
 -  [sysrescuecd](http://www.system-rescue-cd.org/) <!-- boot over pxe? -->
 -  [archlinux live](https://www.archlinux.org/)
 -  [Ubuntu](https://ubuntu.com/)
 -  [Manjaro](https://manjaro.org/)
 -  [Mint](https://www.linuxmint.com/)
 -  [Debian](https://www.debian.org/)

## ToDo

 -  [ ] step by step instructions (1%)
 -  [ ] translate this repo to Spanish (0%)
 -  [ ] flink docker-compose file
   -  [ ] dnsmasq config
   -  [ ] squid config
      -  [ ] transparent proxy
      -  [ ] bandwidth limit?
 -  [ ] fnode docker-compose file
   -  [ ] tftp config
   -  [ ] nfs config
   -  [ ] nginx config
 - [ ] distro support
   -  [ ] ubuntu
      -  [ ] mate
      -  [ ] xubuntu
   -  [ ] linuxmint
      -  [ ] cinnamon
      -  [ ] mate
      -  [ ] xfce
   -  [ ] debian
   -  [ ] manjaro
   -  [ ] tools
      -  [ ] sysrescuecd
      -  [ ] archlinux
      -  [ ] gparted live

### requirements

 -  At least one baremetal or vm with linux
    -  [x] tested on debian
    -  [x] tested on archlinux
    -  [ ] tested on ubuntu
    -  [ ] tested on fedora
    -  [ ] tested on centos
 -  docker
 -  docker-compose
   
## steps

```bash
docker run --rm -t --entrypoint=/bin/sh \
-v "$(pwd)/tftp-hpa":/tmp/tftpboot jumanjiman/tftp-hpa \
-c "mkdir -p /tmp/tftpboot && cp -rv /tftpboot /tmp/tftpboot"
```

# Deploying

## Creating flink

You require at least 2 network interfaces.

1. Install the following packages:
  * docker
  * docker-compose
  * htop, iftop, iotop: status
  * fail2ban: protect ssh
  * shorewall: firewall

2. Set a fixed IP for the server in the nic that connects to the internal network. The other one can have dhcp. Edit /etc/network/interfaces accordingly. Example:

```
# wan
allow-hotplug enp2s0
iface enp2s0 inet dhcp

# lan
auto enp3s0
iface enp3s0 inet static
	address 172.16.0.1/16  # this IP/mask is used with dnsmasq and squid
```

3. Configure the firewall: you can use whichever you like, we used shorewall and sample settings are in the shorewall dir. They work out of the box (but you need to change settings in the *params* file), so edit them and put them in /etc/shorewall. The firewall is required to at least redirect traffict from port 80 to 3128 (squid's port).
4. Optional: change default ssh config for a [custom one](https://gist.github.com/HacKanCuBa/fe3653d4fe4eed35e41dcc9a380499c2) (and regenerate server keys).
5. Clone this repo
6. Change dnsmasq settings accordingly. Particularly, add/remove the nodes and edit its mac addresses (see dnsmasq/dnsmasq.conf and dnsmasq/fnodes). Also set the IP and netmask according to the one set before for the interface. If you used a different IP, also change it in the docker-compose-flink.yml file.
7. Change squid settings accordingly (defaults should suffice).
8. Start the *flink*: `docker-compose -f docker-compose-flink.yml up -d`

# License

flisolator Copyright (C) 2019  snkisuke under [GNU AGPL v3.0+](LICENSE). You are free to use, share, modify and share modifications under the terms of that license.

syslinux files are licensed under [GNU GPL v2.0+](https://repo.or.cz/syslinux.git/blob/HEAD:/COPYING)
