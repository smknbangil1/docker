## Studi Kasus Moodle Load Balancer

Pada studi kasus kali ini akan menggunakan 4 node server yang nantinya 1 server akan menjadi load balancer, database, dan cache server, kemudian 3 node yang lain akan menjadi web moodle-nya

### Konfigurasi node 1

**Instalasi mariadb, redis, haproxy, dan nfs server**  
Pertama lakukan instalasi 
```bash
apt-get install mariadb-server redis haproxy nfs-kernel-server -y
```
Konfigurasi redis
```bash
nano /etc/redis/redis.conf
```
Cari baris konfigurasi ```bind 127.0.0.1 ::1```  
Lalu rubah menjadi 
```bash
bind 0.0.0.0
```


Untuk memberikan password ke redis server, masih di dalam file konfigurasi yang sama, cari baris konfigurasi ```requirepass password```  
Lalu rubah dengan password yang diinginkan, contoh:
```bash
requirepass @RedisP4ssw0rd123
```
Setelah itu restart redis dengan perintah
```bash
systemctl restart redis
```


Konfigurasi mariadb  
Jalankan perintah berikut untuk konfigurasi awal mariadb
```bash
mysqL_secure_installation
```
Saat pertama kali diminta mengisi password root, cukup tekan enter karena kita belum membuat password root untuk mariadb.  
Setelah itu, jika diminta untuk membuat password baru, silahkan isi dengan password yang diinginkan.  
Untuk konfigurasi yang lain, silahkan disesuaikan sesuai kebutuhan.

Setelah itu, buka file konfigurasi mariadb
```bash
nano /etc/mysql/mariadb.cnf
```
Tambahkan konfigurasi berikut pada baris paling bawah file konfigurasi
```bash
[mysqld]
bind-address = 0.0.0.0
```
Setelah itu restart mariadb dengan perintah berikut
```bash
systemctl restart mariadb
```

Membuat database moodle  
Masukkan perintah berikut untuk mengakses mariadb
```bash
mysql -u root -p
```
Untuk membuat database moodle, masukkan perintah berikut
```sql
CREATE DATABASE moodle;
```
Setelah itu beri hak akses remote connection ke database yang sudah dibuat, sesuaikan **'root-password'** dengan password root yang sudah dibuat sebelumnya
```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root-password';
```
Lalu kita load kembali privileges yang sudah kita atur dengan perintah berikut
```sql
FLUSH PRIVILEGES;
EXIT;
```
Konfigurasi HAProxy  
Buka file konfigurasi miliki haproxy dengan perintah berikut
```bash
nano /etc/haproxy/haproxy.cfg
```
Kemudian pada bagian paling bawah file konfigurasi, tambahkan konfigurasi berikut
```bash
frontend main
        bind *:80
        option http-server-close
        option forwardfor
        default_backend moodle

backend moodle
        balance roundrobin
        server moodle1 172.16.10.12:80 check
        server moodle2 172.16.10.13:80 check
	    server moodle3 172.16.10.14:80 check
```
Sesuaikan ip address dengan ip address yang digunakan pada node 2 hingga 4  
Setelah itu restart haproxy dengan perintah berikut  
```bash
systemctl restart haproxy
```

Konfigurasi NFS server  
Kita buat folder untuk NFS terlebih dahulu
```bash
mkdir -p /mnt/nfs-moodle
```
Kemudian kita rubah hak akses dan hak kepemilikan folder yang sudah kita buat
```bash
chown -R nobody:nogroup /mnt/nfs-moodle
chown -R 777 /mnt/nfs-moodle
```
Buka file konfigurasi direktori nfs dengan perintah berikut
```bash
nano /etc/exports
```
Kemudian tambahkan dibagian paling bawah file konfigurasi baris berikut
```bash
/mnt/nfs-moodle  *(rw,sync,no_root_squash,no_subtree_check,insecure)
```
Setelah itu, export konfigurasi yang sudah kita atur dengan perintah berikut dan restart NFS server
```bash
exportfs -a
systemctl restart restart nfs-kernel-server
```

### Konfigurasi Node 2

Untuk instalasi awal moodle, kita hanya perlu melakukannya di node 2 saja

```bash
apt-get install apache2 nfs-common php php-xml php-gd php-fpm php-mbstring php-soap php-intl php-mysqlnd php-xmlrpc php-zip php-curl php-gd php-redis -y && \ 
phpenmod curl && \ 
phpenmod gd && \ 
a2enmod proxy_fcgi && \ 
a2enconf php7.4-fpm && \ 
systemctl restart apache2
```
Konfigurasi redis sebagai session handler di PHP  
Buka file konfigurasi php-fpm dengan perintah berikut
```bash
nano /etc/php/php7.4/apache2/php.ini
```
Setelah itu, cari baris konfigurasi ```session.save_handler = files``` dan rubah menjadi seperti berikut
```bash
session.save_handler = redis
```
Kemudian masih di file konfigurasi yang sama, cari bagian baris konfigurasi ```session.save_path = ""``` dan hapus tanda ```;``` di bagian kiri sebelum baris konfigurasi tersebut dan atur seperti berikut
```bash
session.save_path = "tcp://172.16.10.11:6379?auth=redis-password"
```
Sesuaikan alamat ip dengan alamat ip redis server yang sudah ditentukan, juga rubah bagian **'redis-password'** dengan password redis yang sudah ditentukan sebelumnya.
Restart apache2 server
```bash
systemctl restart apache2
``` 

Mount folder NFS yang sudah kita atur sebelumnya.  
Sebelum melakukan mount, kita buat folder mount terlebih dahulu dengan perintah berikut
```bash
mkdir /var/www/moodledata
```
Kemudian mount nfs dengan perintah berikut
```bash
mount -t nfs 172.16.10.11:/mnt/nfs-moodle /var/www/moodledata
```
Agar ketika server reboot nfs yang sudah di mount tidak hilang, masukkan perintah berikut
```bash
echo '@reboot root    mount -t nfs 172.16.10.11:/mnt/nfs-moodle /var/www/moodledata' >> /etc/crontab
```
Sesuaikan ip address dengan ip address nfs server yang sudah ditentukan

Instalasi moodle  
Sebelumnya, download terlebih dahulu file moodle-nya dengan perintah berikut
```bash
wget https://download.moodle.org/download.php/direct/stable310/moodle-3.10.tgz
```
Kemudian ekstrak file yang sudah di download ke folder /var/www/html dan hapus file index.html bawaan
```bash
tar -xzf moodle-3.10.tgz --strip-components=1 -C /var/www/html && rm /var/www/html/index.html
```
Atur hak akses dan hak kepemilikan untuk folder moodle dan moodledata seperti sebelumnya
```bash
chown www-data:www-data -R /var/www/html
chmod -R 777 /var/www/html 
```
Setelah itu akses web moodle melalui ip address node 1 dan lakukan instalasi moodle seperti biasa.  

Jika saat mengakses web terdapat error 502, maka cukup reload kembali sampai muncul web instalasi moodle, itu artinya load balancing di haproxy sudah berjalan sesuai dengan konfigurasi, hanya saja moodle di node 3 dan 4 belum kita install.

### Konfigurasi Node 3 dan 4
Untuk konfigurasi node 3 dan 4, lakukan seperti konfigurasi pada node 2 hingga pada proses mounting folder NFS dan tidak perlu melakukan instalasi moodle dari awal.  

Copy file moodle dari node 2  
Sebelum melakukan copy, pada node 3 dan 4 kita atur terlebih dahulu hak akses dari folder /var/www/html
```bash
chown www-data:www-data -R /var/www/html
```
Kemudian pada node 2, masukkan perintah berikut untuk mengirim file moodle yang sudah di konfigurasi ke node 3 dan 4
```bash
rsync -r /var/www/html user@ip-node3:/var/www/html
rsync -r /var/www/html user@ip-node4:/var/www/html
```
Setelah itu atur kembali hak akses folder /var/www/html pada node 3 dan 4  
Node 3:
```bash
chmod -R 777 /var/www/html
```
Node 4:
```bash
chmod -R 777 /var/www/html
```
Setelah itu, silahkan coba akses kembali web moodle.