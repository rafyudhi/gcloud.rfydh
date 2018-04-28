# google cloud ubuntu 16.04 nginx, php-fpm 7.2, mariadb, phalcon for php-fpm 7.2

sudo adduser devops
sudo usermod -aG sudo devops
sudo nano /etc/ssh/sshd_config

# Change to no to disable tunnelled clear text passwords
PasswordAuthentication yes

sudo service ssh restart

# install nginx
sudo nano /etc/apt/sources.list

# add this line
deb http://nginx.org/packages/mainline/ubuntu/ xenial nginx

sudo wget http://nginx.org/keys/nginx_signing.key
sudo apt-key add nginx_signing.key

sudo apt update
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# install mariadb
sudo apt-get install mariadb-server mariadb-client
sudo mysql_secure_installation

# set mysql root, and remove user anonym, test database and disable remote login
sudo /etc/init.d/mysql stop
sudo mysqld_safe --skip-grant-tables &
mysql -u root mysql

# run this mysql command 
UPDATE user SET plugin="";
# if you need update the root password
UPDATE user SET password=PASSWORD("xxxx") WHERE user="root";
FLUSH PRIVILEGES;

sudo /etc/init.d/mysql restart
mysql -u root -p

CREATE DATABASE testdb;
CREATE USER 'testuser' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON testdb.* TO 'testuser';
quit
mysql -u testuser -p

# install php-fpm 7.2
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt list --upgradable
sudo apt install php7.2-fpm \
   php7.2-common \
   php7.2-mbstring \
   php7.2-xmlrpc \
   php7.2-soap \
   php7.2-gd \
   php7.2-xml \
   php7.2-intl \
   php7.2-mysql \
   php7.2-cli \
   php7.2-zip \
   php7.2-curl \
   curl \
   vim \
   wget \
   git 

sudo sed -i 's/listen.owner = www-data/listen.owner = nginx/g' /etc/php/7.2/fpm/pool.d/www.conf
sudo sed -i 's/listen.group = www-data/listen.group = nginx/g' /etc/php/7.2/fpm/pool.d/www.conf
sudo sed -i 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' /etc/php/7.2/fpm/php.ini

sudo nano /etc/php/7.2/fpm/php.ini

# update php.ini
file_uploads = On
allow_url_fopen = On
max_file_uploads = 200
max_input_vars = 10000
max_execution_time = 360
memory_limit = 512M
post_max_size = 200M
upload_max_file_size = 200M
date.timezone = Asia/Jakarta

sudo systemctl restart nginx.service
sudo systemctl restart php7.2-fpm.service

# install phalcon for php-fpm 7.2
curl -s https://packagecloud.io/install/repositories/phalcon/stable/script.deb.sh | sudo bash
sudo apt-get install php7.2-phalcon
sudo cp /etc/php/7.2/mods-available/phalcon.ini /etc/php/7.2/fpm/conf.d/20-phalcon.ini 
sudo service php7.2-fpm restart
sudo systemctl restart nginx.service

#install composert
sudo curl -sS http://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# set ssl
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
# [-d domain.. -d domain ..n..]
sudo certbot --nginx -m ***email*** -d ***domain***
sudo nano /etc/nginx/conf.d/***domain***.conf

# value ***domain***.conf

server {
   listen 80;
   listen [::]:80;
   server_name  ***domain***;

   #access_log  /var/log/nginx/host.access.log  main;

   root   /home/devops/web/***domain***;
   index  index.php index.html index.htm;

   #error_page  404              /404.html;

   # redirect server error pages to the static page /50x.html
   #

   error_page   500 502 503 504  /50x.html;
   location = /50x.html {
      root   /usr/share/nginx/html;
   }

   client_max_body_size 200M;

   location / {
      #try_files $uri $uri/ =404;
      try_files $uri $uri/ /index.php?$args;
   }

   location ~* \.php$ {
      fastcgi_pass unix:/run/php/php7.2-fpm.sock;
      include         fastcgi_params;
      fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
      fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
   }

   location ~ /\.ht {
      deny  all;
   }

   listen [::]:443 ssl ipv6only=on; # managed by Certbot
   listen 443 ssl; # managed by Certbot
   ssl_certificate /etc/letsencrypt/live/***domain***/fullchain.pem; # managed by Cert$
   ssl_certificate_key /etc/letsencrypt/live/***domain***/privkey.pem; # managed by Ce$
   include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
   ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

   if ($scheme != "https") {
      return 301 https://$host$request_uri;
   }
}

sudo systemctl restart nginx.service

# install redis
sudo apt-get install redis-server
sudo apt-get install php-redis
sudo nano /etc/redis/redis.conf

# tambahin di end line : 
maxmemory 128mb
maxmemory-policy allkeys-lru

sudo systemctl restart redis-server.service
sudo systemctl enable redis-server.service
redis-cli

# install phpmyadmin
# pas install, jangan pilih apache2 or lighthttpd
# ignore aja utk database nya

sudo apt-get update
sudo apt-get install phpmyadmin
sudo ln -s /usr/share/phpmyadmin /var/www/web/mysqlground
sudo phpenmod mcrypt
sudo systemctl restart php7.2-fpm

# install postgresql & phppgadmin

sudo apt-get -y install postgresql postgresql-contrib
sudo apt-get install phppgadmin
sudo ln -s /usr/share/phppgadmin /var/www/web/pgsqlground
sudo su
su - postgres
psql
\password postgres

# ikutin langkah utk set password
sudo nano phppgadmin/conf/config.inc.php
ubah parameter ini : 
$conf['extra_login_security'] = true;

sudo systemctl restart postgresql
