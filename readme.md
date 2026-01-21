 Pterodactyl Installation Guide on VPS (Error-Free)  body { font-family: Arial, sans-serif; background: #f7f9fb; color: #333; padding: 20px; } h1, h2, h3 { color: #1e293b; } pre { background: #0f172a; color: #e5e7eb; padding: 15px; overflow-x: auto; border-radius: 6px; } code { color: #38bdf8; } .note { background: #e0f2fe; padding: 10px; border-left: 5px solid #0284c7; margin: 10px 0; } .warn { background: #fff7ed; padding: 10px; border-left: 5px solid #f97316; margin: 10px 0; }

🚀 Pterodactyl Installation Guide on VPS (Error-Free)
=====================================================

This guide shows how to install **Pterodactyl Game Server Panel** on a fresh Ubuntu VPS with all common errors fixed.

* * *

Step 1: Prepare Your VPS
------------------------

Update your system and set the correct timezone. This prevents timezone-related PHP and Composer errors.

    apt update && apt upgrade -y
    timedatectl set-timezone Asia/Singapore

Step 2: Install Basic Packages
------------------------------

These tools are required for downloading and managing software.

    apt install -y curl wget unzip git sudo software-properties-common

Step 3: Install PHP 8.2 (Recommended)
-------------------------------------

Pterodactyl works best with PHP 8.2. Avoid PHP 8.4 which causes issues.

    add-apt-repository ppa:ondrej/php -y
    apt update
    apt install -y php8.2 php8.2-cli php8.2-fpm php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl php8.2-zip php8.2-bcmath php8.2-intl
    systemctl enable php8.2-fpm
    systemctl start php8.2-fpm

Step 4: Install MariaDB & Redis
-------------------------------

Database and caching services required by Pterodactyl.

    apt install mariadb-server redis-server -y
    systemctl enable mariadb redis-server
    systemctl start mariadb redis-server
    mysql_secure_installation

Step 5: Create Database
-----------------------

This database stores all panel data.

    mysql -u root -p

    CREATE DATABASE panel;
    CREATE USER 'ptero'@'localhost' IDENTIFIED BY 'StrongPassword';
    GRANT ALL PRIVILEGES ON panel.* TO 'ptero'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;

Step 6: Install NGINX & Remove Apache
-------------------------------------

Apache causes port 80 conflicts. Use NGINX only.

    apt install nginx -y
    systemctl stop apache2
    systemctl disable apache2
    apt remove apache2 -y
    systemctl enable nginx
    systemctl start nginx

Step 7: Install Pterodactyl Panel
---------------------------------

    mkdir -p /var/www/pterodactyl
    cd /var/www/pterodactyl
    curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
    tar -xzvf panel.tar.gz
    chmod -R 755 storage/* bootstrap/cache/

Step 8: Install Composer
------------------------

Composer installs PHP dependencies.

    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php composer-setup.php
    mv composer.phar /usr/local/bin/composer
    chmod +x /usr/local/bin/composer

Step 9: Install Panel Dependencies
----------------------------------

    composer install --no-dev --optimize-autoloader
    cp .env.example .env
    php artisan key:generate --force
    php artisan p:environment:setup
    php artisan p:environment:database
    php artisan migrate --seed --force
    php artisan p:user:make

Step 10: Fix Permissions (Important)
------------------------------------

This fixes the "page not loading" issue.

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
    }

    ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/
    rm /etc/nginx/sites-enabled/default
    nginx -t
    systemctl restart nginx

Step 12: Install Docker
-----------------------

    curl -fsSL https://get.docker.com | bash
    systemctl enable docker
    systemctl start docker

Step 13: Install Wings
----------------------

    mkdir -p /etc/pterodactyl
    curl -L -o /usr/local/bin/wings https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_amd64
    chmod +x /usr/local/bin/wings

Step 14: Create Wings Service
-----------------------------

    nano /etc/systemd/system/wings.service

    [Unit]
    Description=Pterodactyl Wings Daemon
    After=docker.service
    Requires=docker.service
    
    [Service]
    User=root
    ExecStart=/usr/local/bin/wings
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target

    systemctl daemon-reload
    systemctl enable wings
    systemctl start wings

Step 15: Add Node in Panel
--------------------------

Create a node in the panel and copy the configuration to:

    nano /etc/pterodactyl/config.yml

Step 16: Add Allocations
------------------------

Add IP and ports like **25565** for Minecraft.

Step 17: Create Game Server
---------------------------

Now create your server from panel and start hosting games 🎮

Common Errors Fixed
-------------------

*   Composer timezone issue → Fixed by tzdata + timezone
*   PHP not found → Installed php-cli
*   NGINX port 80 conflict → Removed Apache
*   Page not loading → Fixed permissions
*   Allocation error → Added ports
*   Wings not starting → Created systemd service

Conclusion
----------

You now have a fully working Pterodactyl setup on VPS 🚀
