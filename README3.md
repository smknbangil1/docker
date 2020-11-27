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
    enp0s8:
      addresses: [172.16.10.10/24]
  version: 2
```

### Ubah Hostname di Ubuntu (untuk Manager/Leader Swarm)
```bash
hostnamectl set-hostname Master
```
### Ubah timezone WIB
```bash
timedatectl set-timezone Asia/Jakarta
```
### ===================
## instalasi docker
```bash
apt update -y && apt upgrade -y
```
enter
```bash
apt install docker.io -y
```
enter dan tunggu proses instalasinya,
jalankan docker
```bash
systemctl start docker
```
enter, 
aktifkan docker saat booting
```bash
systemctl enable docker
```
cek status docker
```bash
systemctl status docker
```
## Pengenalan Docker Swarm

### Inisialisasi docker swarm pada node 1
```bash
root@node1:~# docker swarm init --advertise-addr [ip private node1]
```
### Melihat join token docker swarm pada node 1 dan join node 2 ke cluster swarm
```bash
root@node1:~# docker swarm join-token worker
```
Copy perintah join swarm pada node 2
```bash
root@node2:~# docker swarm join --token [join token] [ip private node1]:2377
```
### Mengecek anggota cluster swarm
```bash
root@node1:~# docker node ls
```
### Menjalankan container pada cluster swarm
```bash
root@node1:~# cd && mkdir latihan5
root@node1:~# cd latihan5
root@node1:~# nano docker-compose.yml
```
Isikan:
```yaml
version: '3.8'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    deploy:
      replicas: 4
```
```bash
root@node1:~# docker stack deploy --compose-file docker-compose.yml [nama stack]
root@node1:~# docker stack ls
root@node1:~# docker stack ps [nama stack]
```
### Menghapus stack dari docker swarm
```bash
root@node1:~# docker stack ls
root@node1:~# docker stack rm [nama stack]
```

## Pengenalan Portainer
### Download file Portainer
```bash
cd && mkdir latihan6
cd latihan6
curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml
```
### Menjalankan stack Portainer
```bash
docker stack deploy --compose-file=portainer-agent-stack.yml portainer
docker stack ls
docker stack services portainer
```
Akses lewat browser menggunakan ip node1:9000
Atur username dan password

## Studi Kasus Moodle dengan Docker Swarm
### Buat file docker-compose.yml moodle
```bash
cd && mkdir latihan7
cd latihan7
nano docker-compose.yml
```
Isikan:
```yaml
version: '3.8'
services:
  mariadb:
    image: 'docker.io/bitnami/mariadb:10.3-debian-10'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_moodle
      - MARIADB_DATABASE=bitnami_moodle
    volumes:
      - 'mariadb_data:/bitnami/mariadb'
  moodle:
    image: 'docker.io/bitnami/moodle:3-debian-10'
    ports:
      - '80:8080'
      - '443:8443'
    environment:
      - MOODLE_DATABASE_HOST=mariadb
      - MOODLE_DATABASE_PORT_NUMBER=3306
      - MOODLE_DATABASE_USER=bn_moodle
      - MOODLE_DATABASE_NAME=bitnami_moodle
      - ALLOW_EMPTY_PASSWORD=yes
    volumes:
      - 'moodle_data:/bitnami/moodle'
      - 'moodledata_data:/bitnami/moodledata'
    depends_on:
      - mariadb
volumes:
  mariadb_data:
    driver: local
  moodle_data:
    driver: local
  moodledata_data:
    driver: local
```
### Menjalankan stack moodle
```bash
docker stack deploy --compose-file docker-compose.yml moodle
docker stack ls
docker stack ps moodle
docker stack services moodle
```
Akses lewat browser dengan ip node1:80
