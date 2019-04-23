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