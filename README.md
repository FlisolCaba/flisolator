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

# Idea

The main idea is to use a swarm of servers to serve ISOs and packages in parallel, with one director. The director manages the DNS and DHCP, and runs the PXE boot (for both BIOS and UEFI). Then, it directs the request to a node that has a TFTP and NFS/HTTP server to send the required files to boot, run and install any distro.  
We call the director *flink* and each node *fnode*.

The *flink* uses **dnsmasq** to serve DNS, DHCP and PXE, **squid** as a proxy cache for packages and web content and **shorewall** as firewall to both protect the server and redirect port 80 (plain HTTP) to **squid** for caching and balancing port 443 (HTTPS).

The *fnode* uses **nfs** to mount a share with the ISOs and **nginx** to server the same but over HTTP, and **tftpd** for the TFTP-HPA server.  
You can have as many nodes as you want, the more the better and at least one.

# Deploying

## Creating flink

You require at least 2 network interfaces.

1. Install the following packages:
  * docker
  * docker-compose
  * shorewall: firewall
  * htop, iftop, iotop: status
  * fail2ban: protect ssh

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

3. Optional: change default ssh config for a [custom one](https://gist.github.com/HacKanCuBa/fe3653d4fe4eed35e41dcc9a380499c2) (and regenerate server keys).
4. Clone this repo.
5. Configure the firewall: you can use whichever you like, we used shorewall and sample settings are in the shorewall dir. They work out of the box (but you need to change settings in the *params* file), so edit them and put them in /etc/shorewall. The firewall is required to at least redirect traffict from port 80 to 3128 (squid's port).
6. Change dnsmasq settings accordingly. Particularly, add/remove the nodes and edit its mac addresses (see dnsmasq/dnsmasq.conf and dnsmasq/fnodes). Also set the IP and netmask according to the one set before for the interface. If you used a different IP, also change it in the docker-compose-flink.yml file.
7. Change squid settings accordingly (defaults should suffice).
8. Start the *flink*: `docker-compose -f docker-compose-flink.yml up -d`

## Creating a node

Repeat this for each node, named as `fnodeNN` where `N` is [0-9].

1. Install the following packages:
  * docker
  * docker-compose
  * nfs: iso share
  * htop, iftop, iotop: status
  * fail2ban: protect ssh
  * ufw: firewall (pending for future release)
2. Set the IP as dynamic, *flink* will provide (however note that the IP will be fixed for the mac address, so if you want, you can use static IP).
3. Optional: change default ssh config for a [custom one](https://gist.github.com/HacKanCuBa/fe3653d4fe4eed35e41dcc9a380499c2) (and regenerate server keys).
4. Clone this repo.
5. Configure nfs, edit `/etc/exports` and add a line at the end (editing path and IP address accordingly) such as: `/srv/flisolator/fnode/tftp-hpa/tftpboot/isos/    172.16.0.0/16(ro,async,no_subtree_check,all_squash)`
6. Edit the menues as you like in `fnode/tftp-hpa/tftpboot/pxelinux.cfg`. The main menu is the file `default`. *Important note*: some menues (such as `ubuntu.conf`) require to send the kernel a `nfsroot` or similar param which unfortunately requires setting the IP of the node server (hostname can not be used). This is a problem related with the way the kernel works (it does not support DNS at this early stage of booting). Maybe in the future we'll be able to hack this limitation. For now, you need to change the IPs on each menu for each node. As a helper, run `find fnode/tftp-hpa/tftpboot/pxelinux.cfg/ -type f -exec grep "172.16.0.11" "{}" \+` to show all the ocurrences that needs to be changed. Try this sed if you want: `find fnode/tftp-hpa/tftpboot/pxelinux.cfg/ -type f -exec sed -i 's/172.16.0.11/FNODE_IP/g' "{}" \+` (replace `FNODE_IP` with the IP of the *fnode*).
6. Start the *fnode*: `docker-compose -f docker-compose-fnode.yml up -d`
7. Remember to add the IP and name of the node to the *flink* (see *flink*'s step 6).

# License

flisolator Copyright (C) 2019  snkisuke under [GNU AGPL v3.0+](LICENSE). You are free to use, share, modify and share modifications under the terms of that license.

syslinux files are licensed under [GNU GPL v2.0+](https://repo.or.cz/syslinux.git/blob/HEAD:/COPYING)

