# Rangkuman 2

## Instalasi Docker Pada Node 2
```bash
root@node2:~# apt update -y $$ apt install docker.io -y
root@node2:~# systemctl status docker
root@node2:~# systemctl enable docker
root@node2:~# systemctl start docker
```

## Pengenalan Docker Volume Driver (sshFS)
### Membuat SSH Key Baru pada Node 1
```bash
root@node1:~# ssh-keygen -t rsa
```
### Copy Isi dari Public Key ke Node 2
```bash
root@node1:~# cat ~/.ssh/id_rsa.pub
```
Copy isi dari file id_rsa.pub dan paste ke file authorized_keys pada node 2
```bash
root@node2:~# nano ~/.ssh/authorized_keys
```
### SSH dari Node 1 ke Node 2
```bash
root@node1:~# ssh root@[ip node2]
```
### Buat direktori data pada Node 2
```bash
root@node2:~# mkdir /webdata
root@node2:~# chmod 777 /webdata
```
### Install dan atur plugin sshfs pada Node 1
```bash
root@node1:~# docker plugin install --grant-all-permissions vieux/sshfs
root@node1:~# docker plugin ls
root@node1:~# docker plugin disable [nama plugin / id plugin]
root@node1:~# docker plugin set vieux/sshfs sshkey.source=/root/.ssh/
root@node1:~# docker plugin enable [nama plugin / id plugin]
root@node1:~# docker plugin ls
```
### Buat volume sshfs pada Node 1
```bash
root@node1:~# docker volume create --driver vieux/sshfs -o sshcmd=root@[ip node2]:/webdata -o allow_other sshvolume
```
### Menjalankan container nginx dengan volume driver sshfs pada Node 1
```bash
root@node1:~# docker run -d -p 8080:80 -v sshvolume:/usr/share/nginx/html nginx:latest
```
### Menambahkan file index.html pada Node 2
```bash
root@node2:~# nano /webdata/index.html
```
Isikan:
```html
<h1>Halo dari node 2</h1>
```
Akses lewat browser menggunakan ip node1:8080

## Pengenalan Docker Network
### Membuat Docker Network Baru
```bash
docker network create --driver bridge [nama network]
docker network ls
```
### Membuat Docker Network dengan Subnet
```bash
docker network create --driver bridge --subnet [ip subnet, contoh: 192.168.0.0/24] [nama network]
docker network ls
```
### Menjalankan container nginx dengan subnet baru
```bash
docker run -d -p 8080:80 --network [nama network] nginx
```
### Mengecek informasi network baru
```bash
docker network ls
docker network inspect [nama network]
```

## Pengenalan Docker Machine

### Instalasi perintah docker-machine
```bash
base=https://github.com/docker/machine/releases/download/v0.16.0 &&
curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
install /tmp/docker-machine /usr/local/bin/docker-machine
```
### Deploy docker-machine pada Google Cloud Platform (GCP)
Pastikan sudah memiliki akun **GCP** dan **Google Application Credentials**
```bash
root@node1:~# export GOOGLE_APPLICATION_CREDENTIALS=/lokasi/file/gce-credentials.json
root@node1:~# docker-machine create --driver google --google-project [project gcp] \
--google-zone us-central1-a --google-machine-type n1-standard-1 [namavm]
```
### Mengecek daftar docker-machine yang sedang aktif
```bash
root@node1:~# docker-machine ls
```

### Menggunakan docker-machine yang sedang aktif
```bash
root@node1:~# eval $(docker-machine env [namavm])
```
Untuk kembali menggunakan docker local, masukkan perintah berikut:
```bash
root@node1:~# eval $(docker-machine env -u)
```
### Menghentikan docker-machine yang sedang aktif
```bash
root@node1:~# docker-machine stop [namavm]
```
### Menghapus docker-machine
```bash
root@node1:~# docker-machine rm [namavm]
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
