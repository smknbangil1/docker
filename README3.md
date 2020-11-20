# Install Ubuntu + LAN + Docker Swarm

## periksa seluruh interface NIC
```bash
ifconfig -a
```

## konfig LAN
```bash
nano /etc/netplan/00-installer-config.yaml
```
### terus masukin syntax yaml sesuai kebutuhan dan topologi Anda
contoh di jaringan saya, berikut ini
```bash
network:
  ethernets:
    enp0s3:
      dhcp4: true
  version: 2

network:
  ethernets:
    enp0s8:
      addresses: [172.16.10.10/24]
  version: 2
```


## instalasi docker
```bash
apt update -y && apt upgrade -y
```
enter
```bash
apt install docker.io -y
```
enter dan tunggu proses instalasinya
```bash
systemctl start docker
```
enter
```bash
systemctl enable docker
```
