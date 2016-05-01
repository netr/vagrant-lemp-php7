$script = <<SCRIPT
#!/usr/bin/env bash

export DEBIAN_FRONTEND=noninteractive

# MariaDB repo
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db
sudo add-apt-repository 'deb [arch=amd64,i386] http://lon1.mirrors.digitalocean.com/mariadb/repo/10.1/ubuntu trusty main'

cat << 'EOF' > /tmp/mariadb.pref
Package: *
Pin: origin lon1.mirrors.digitalocean.com
Pin-Priority: 1000
EOF
sudo mv /tmp/mariadb.pref /etc/apt/preferences.d/mariadb.pref
# MariaDB repo

# PHP 7 repo
sudo add-apt-repository -y ppa:ondrej/php
# PHP 7 repo

sudo apt-get update

sudo apt-get install -y -q nginx

sudo apt-get install -y -q php7.0-fpm
sudo apt-get install -y -q php7.0-mysql
sudo apt-get install -y -q php7.0-mbstring

sudo rm /etc/nginx/sites-available/default
sudo touch /etc/nginx/sites-available/default

cat << 'EOF' > /tmp/default
server {
  listen 80;

  root /vagrant;
  index index.php index.html index.htm;

  # Make site accessible from any domain
  server_name _;

  location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to index.html
    try_files $uri $uri/ /index.html;
  }

  location /doc/ {
    alias /usr/share/doc/;
    autoindex on;
    allow 127.0.0.1;
    deny all;
  }

  # redirect server error pages to the static page /50x.html
  #
  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/html;
  }

  # pass the PHP scripts to FastCGI server listening on /tmp/php5-fpm.sock
  #
  location ~ \.php$ {
    try_files $uri =404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
    fastcgi_index index.php;
    include fastcgi_params;
  }

  ### phpMyAdmin ###
  location /phpmyadmin {
    root /usr/share/;
    index index.php index.html index.htm;
    location ~ ^/phpmyadmin/(.+\.php)$ {
      client_max_body_size 4M;
      client_body_buffer_size 128k;
      try_files $uri =404;
      root /usr/share/;

      # Point it to the fpm socket;
      fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      include /etc/nginx/fastcgi_params;
    }

    location ~* ^/phpmyadmin/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt)) {
      root /usr/share/;
    }
  }
  location /phpMyAdmin {
    rewrite ^/* /phpmyadmin last;
  }
  ### phpMyAdmin ###
}
EOF

sudo mv /tmp/default /etc/nginx/sites-available/default

sudo service nginx restart

sudo service php7.0-fpm restart

debconf-set-selections <<< 'mariadb-server-5.5 mysql-server/root_password password rootpass'
debconf-set-selections <<< 'mariadb-server-5.5 mysql-server/root_password_again password rootpass'
sudo apt-get install -y -q mariadb-server
mysql -uroot -prootpass -e "SET PASSWORD = PASSWORD('');"

# Install composer
sudo apt-get install -y -q curl git
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer

# Install phpmyadmin from source
wget -q https://github.com/phpmyadmin/phpmyadmin/archive/STABLE.tar.gz -O /tmp/phpmyadmin.tar.gz
sudo tar -xzf /tmp/phpmyadmin.tar.gz -C /usr/share
sudo mv /usr/share/phpmyadmin-STABLE/ /usr/share/phpmyadmin/
cd /usr/share/phpmyadmin && composer update --no-dev
cat << 'EOF' > /tmp/config.inc.php
<?php
/*
 * Generated configuration file
 * Generated by: phpMyAdmin 4.6.0 setup script
 * Date: Sun, 01 May 2016 00:55:23 +0000
 */

/* Servers configuration */
$i = 0;

/* Server: localhost [1] */
$i++;
$cfg['Servers'][$i]['verbose'] = '';
$cfg['Servers'][$i]['host'] = 'localhost';
$cfg['Servers'][$i]['port'] = '';
$cfg['Servers'][$i]['socket'] = '';
$cfg['Servers'][$i]['connect_type'] = 'socket';
$cfg['Servers'][$i]['nopassword'] = true;
$cfg['Servers'][$i]['auth_type'] = 'config';
$cfg['Servers'][$i]['user'] = 'root';
$cfg['Servers'][$i]['password'] = '';
$cfg['Servers'][$i]['AllowNoPassword'] = true;

/* End of servers configuration */

$cfg['blowfish_secret'] = '572553549888b7.75743085';
$cfg['DefaultLang'] = 'en';
$cfg['ServerDefault'] = 1;
$cfg['UploadDir'] = '';
$cfg['SaveDir'] = '';
EOF
sudo mv /tmp/config.inc.php /usr/share/phpmyadmin/config.inc.php
SCRIPT

# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # config.vm.synced_folder "./", "/usr/share/nginx/"

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "1024"
  end

  #config.vm.provision "shell", path: "provision.sh"
  config.vm.provision "shell", inline: $script
end
