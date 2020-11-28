# Install Ubuntu + Docker Swarm + Moodle
# Di SMKN 1 Bangil

## periksa seluruh interface NIC
```bash
ifconfig -a
```

## konfig LAN
```bash
nano /etc/netplan/00-installer-config.yaml
```
### terus masukin syntax yaml sesuai kebutuhan dan topologi di sekolah
contoh di jaringan saya, berikut ini
```bash
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses: [172.16.10.11/24]
  version: 2
```

### Ubah Hostname di Ubuntu
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
## Docker Swarm

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
## Install Portainer
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
### Saya masukkan script yaml dibawah ini ke Menu Portainer > Stack

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
#### klik tombol Deploy the stack
#### ..tunggu sampai proses selesai
#### lalu buka browser http://ip_node1

```
sampai disini moodle sudah bisa di akses tapi belum bisa menyimpan data, ketika saya restart server  node 1 sampai node 3 
aplikasi moodle kembali seperti semula, kosong lagi datanya... padahal sebelum restart server saya sempat memasukkan beberapa akun siswa hilang setelah restart
