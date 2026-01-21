 Pterodactyl Installation Guide on VPS (Error-Free) 

🚀 Pterodactyl Installation Guide on VPS (Error-Free)
=====================================================

This guide helps you install **Pterodactyl Game Server Panel** on a fresh Ubuntu VPS with verification steps and no hidden pitfalls.

* * *

Step 1: Prepare Your VPS
------------------------

Update your system and set the correct timezone.

    apt update && apt upgrade -y
    timedatectl set-timezone Asia/Singapore

✔ Verify timezone:

    timedatectl

Step 2: Install Basic Packages
------------------------------

    apt install -y curl wget unzip git sudo software-properties-common

Step 3: Install PHP 8.2
-----------------------

    add-apt-repository ppa:ondrej/php -y
    apt update
    apt install -y php8.2 php8.2-cli php8.2-fpm php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl php8.2-zip php8.2-bcmath php8.2-intl
    systemctl enable php8.2-fpm
    systemctl start php8.2-fpm

✔ Verify PHP:

    php -v
    systemctl status php8.2-fpm

Step 4: Install MariaDB & Redis
-------------------------------

    apt install mariadb-server redis-server -y
    systemctl enable mariadb redis-server
    systemctl start mariadb redis-server

✔ Verify services:

    systemctl status mariadb
    systemctl status redis-server

Step 5: Create Database
-----------------------

    mysql -u root -p

    CREATE DATABASE panel;
    CREATE USER 'ptero'@'localhost' IDENTIFIED BY 'StrongPassword';
    GRANT ALL PRIVILEGES ON panel.* TO 'ptero'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;

Step 6: Install NGINX & Remove Apache
-------------------------------------

    apt install nginx -y
    systemctl stop apache2
    systemctl disable apache2
    apt remove apache2 -y
    systemctl enable nginx
    systemctl start nginx

✔ Verify NGINX:

    systemctl status nginx

Step 7: Install Pterodactyl Panel
---------------------------------

    mkdir -p /var/www/pterodactyl
    cd /var/www/pterodactyl
    curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
    tar -xzvf panel.tar.gz
    chmod -R 755 storage/* bootstrap/cache/

Step 8: Install Composer
------------------------

    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php composer-setup.php
    mv composer.phar /usr/local/bin/composer
    chmod +x /usr/local/bin/composer

✔ Verify Composer:

    composer --version

Step 9: Install Panel Dependencies
----------------------------------

    composer install --no-dev --optimize-autoloader
    cp .env.example .env
    php artisan key:generate --force
    php artisan p:environment:setup
    php artisan p:environment:database
    php artisan migrate --seed --force
    php artisan p:user:make

Step 10: Fix Permissions
------------------------

    chown -R www-data:www-data /var/www/pterodactyl
    chmod -R 755 /var/www/pterodactyl

Step 11: Configure NGINX
------------------------

    nano /etc/nginx/sites-available/pterodactyl.conf

    server {
        listen 80;
        server_name YOUR_SERVER_IP;
        root /var/www/pterodactyl/public;
        index index.php;
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }
        location ~ \.php$ {
            fastcgi_pass unix:/run/php/php8.2-fpm.sock;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
        location ~ /\.ht {
            deny all;
        }
    }

    ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/
    rm /etc/nginx/sites-enabled/default
    nginx -t
    systemctl restart nginx

✔ Open in browser:

    http://YOUR_SERVER_IP

Login to the panel before proceeding further.

Step 12: Install Docker
-----------------------

    curl -fsSL https://get.docker.com | bash
    systemctl enable docker
    systemctl start docker

✔ Verify Docker:

    docker --version
    systemctl status docker

Step 13: Install Wings
----------------------

    mkdir -p /etc/pterodactyl
    curl -L -o /usr/local/bin/wings https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_amd64
    chmod +x /usr/local/bin/wings

✔ Verify Wings binary:

    wings --version

Step 14: Create Node (From Browser)
-----------------------------------

Open your browser → Login to Pterodactyl Panel → Admin → Nodes → Create Node Fill:

*   FQDN: Your Server IP
*   Scheme: HTTP
*   Daemon Port: 8080

Step 15: Connect Wings to Panel
-------------------------------

On your server, create the config file:

    nano /etc/pterodactyl/config.yml

Save and close after following panel instructions (do not paste raw JSON blindly).

Step 16: Create Wings Service
-----------------------------

    nano /etc/systemd/system/wings.service

    [Unit]
    Description=Pterodactyl Wings Daemon
    After=docker.service
    Requires=docker.service
    
    [Service]
    User=root
    WorkingDirectory=/etc/pterodactyl
    ExecStart=/usr/local/bin/wings
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target

    systemctl daemon-reload
    systemctl enable wings
    systemctl start wings

✔ Verify Wings:

    systemctl status wings

Node should now show as connected in the panel.

Step 17: Add Allocations (From Browser)
---------------------------------------

Panel → Node → Allocations Add:

*   IP: Your Server IP
*   Ports: 25565-25575

Step 18: Create Game Server
---------------------------

Panel → Servers → Create Server Select Node, Port, Egg, RAM & Disk Start server 🎮

Common Errors Fixed
-------------------

*   Composer timezone issue → Fixed via timezone setup
*   NGINX port conflict → Apache removed
*   Page not loading → Permissions fixed
*   Wings not connecting → Proper service created
*   Allocation error → Ports added

Conclusion
----------

You now have a fully working Pterodactyl setup on VPS 🚀
