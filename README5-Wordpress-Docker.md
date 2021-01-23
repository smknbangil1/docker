# version: '3.7'
services:
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: database
      WORDPRESS_DB_USER: wpdb_user
      WORDPRESS_DB_PASSWORD: wpdbJakarta470
      WORDPRESS_DB_NAME: wpdb
    volumes:
      - ./wordpress:/var/www/html

  database:
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: wpdb
      MYSQL_USER: wpdb_user
      MYSQL_PASSWORD: wpdbJakarta470
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - ./db:/var/lib/mysql
