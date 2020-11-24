# belajar docker
# =============================

# Rangkuman 1

## Instalasi Docker

```bash
sudo -i
apt update -y && apt install docker.io -y
systemctl status docker
systemctl enable docker
systemctl start docker
```

## Perintah Dasar Docker

### Cek Versi Docker
```bash
docker version
```

### Cek Informasi Docker
```bash
docker info
```

### Cek image docker
```bash
docker image ls
```

### Cek container docker
```bash
docker container ls 
docker container ls -a 
```

### Menjalankan container pertama kali
```bash
docker run hello-world
```

### Menghapus image docker
```bash
docker rmi [nama image]
```

### Menghentikan container
```bash
docker container stop [nama container atau id container]
```

### Menghapus container
```bash
docker container rm [nama container atau id container]
```

### Contoh menjalankan container nginx
```bash
docker run -d -p 8080:80 nginx
```
akses ke browser dengan alamat ip server:8080
```bash
docker container ls
docker stop [nama container nginx]
```

## Pengenalan Dockerfile

### Membuat Dockerfile baru
```bash
mkdir latihan1
cd latihan1
nano Dockerfile.  
```
Isikan:
```dockerfile
FROM docker/whalesay:latest
RUN apt update -y && apt install fortunes -y
CMD /usr/games/fortune -a | cowsay
```

### Membuat Image baru dengan Dockerfile
```bash
docker build -t [username dockerhub]/docker-whale .
docker image ls
docker run [username dockerhub]/docker-whale
```

### Upload Image baru ke Docker Hub
```bash
docker login
docker push [username dockerhub]/docker-whale
cek repository docker hub kita
```

### Menambahkan Tag baru pada Image
```bash
docker build -t [username dockerhub]/docker-whale:latest [username dockerhub]/docker-whale:v1.0
docker push -t [username dockerhub]/docker-whale:v1
cek repository docker hub kita
```

## Pengenalan Docker Volume

### Membuat volume baru
```bash
docker volume create volume1
```

### Cek daftar volume pada docker
```bash
docker volume ls
docker volume inspect [nama volume]
```

### Menjalankan container nginx dengan volume
```bash
docker run -d -p 8080:80 -v volume1:/usr/share/nginx/html nginx
```

## Pengenalan Docker Compose

### Instalasi docker-compose
```bash
apt install python3-pip -y
pip3 install docker-compose
```

### Membuat docker-compose.yml
```bash
cd && mkdir latihan2
cd latihan2
nano docker-compose.yml
```
Isikan:
```yaml
version: '3.8'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

### Menjalankan docker-compose
```bash
docker-compose up -d
akses browser ke alamat ip server:8080
```

### Menghentikan docker-compose
```bash
docker-compose down
```

### Membuat docker-compose.yml dan membuat image baru dengan Dockerfile
```bash
cd && mkdir latihan3
cd latihan3
mkdir data
nano Dockerfile
```
Isikan:
```dockerfile
FROM nginx
COPY data/index.html /usr/share/nginx/html/index.html
```
```bash
nano docker-compose.yml
```
Isikan:
```yaml
version: '3.8'
services:
  website:
    build: .
    ports:
      - "8080:80"
```
```bash
cd data
nano index.html
```
Isikan:
```html
<h1>Halo Dunia!</h1>
```
```bash
cd ~/latihan3
docker-compose up -d
akses browser ke alamat ip server:8080
```

### Membuat service Wordpress dengan docker-compose
```bash
cd && mkdir latihan4
cd latihan4
nano docker-compose.yml
```
Isikan:
```yaml
version: '3.8'
services:
  db:
    image: mariadb
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
  wordpress:
    depends_on:
      - db
    image: wordpress
    ports:
      - "80:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress

volumes:
  db_data:
```
```bash
docker-compose up -d
```
akses browser lewat ip server

