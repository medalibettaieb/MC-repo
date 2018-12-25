# MC-repo
vim Dockerfile
On the top of the file, add a line with the base image (Ubuntu 16.04) that we want to use.

#Download base image ubuntu 16.04
FROM ubuntu:16.04
Update the Ubuntu software repository inside the dockerfile with the 'RUN' command.

# Update Ubuntu Software repository
RUN apt-get update
Then install the applications that we need for the custom image. Install Nginx, PHP-FPM and Supervisord from the Ubuntu repository with apt. Add the RUN commands for Nginx and PHP-FPM installation.

# Install nginx, php-fpm and supervisord from ubuntu repository
RUN apt-get install -y nginx php7.0-fpm supervisor && \
    rm -rf /var/lib/apt/lists/*
At this stage, all applications are installed and we need to configure them. We will configure Nginx for handling PHP applications by editing the default virtual host configuration. We can replace it our new configuration file, or we can edit the existing configuration file with the 'sed' command.

In this tutorial, we will replace the default virtual host configuration with a new configuration by using the 'COPY' dockerfile command.

#Define the ENV variable
ENV nginx_vhost /etc/nginx/sites-available/default
ENV php_conf /etc/php/7.0/fpm/php.ini
ENV nginx_conf /etc/nginx/nginx.conf
ENV supervisor_conf /etc/supervisor/supervisord.conf
 
# Enable php-fpm on nginx virtualhost configuration
COPY default ${nginx_vhost}
RUN sed -i -e 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' ${php_conf} && \
    echo "\ndaemon off;" >> ${nginx_conf}
Next, configure Supervisord for Nginx and PHP-FPM. We will replace the default Supervisord configuration with a new configuration by using the 'COPY' command.

#Copy supervisor configuration
COPY supervisord.conf ${supervisor_conf}
Now create a new directory for the php-fpm sock file and change the owner of the /var/www/html directory and PHP directory to www-data.

RUN mkdir -p /run/php && \
    chown -R www-data:www-data /var/www/html && \
    chown -R www-data:www-data /run/php
Next, define the volume so we can mount the directories listed below to the host machine.

# Volume configuration
VOLUME ["/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx", "/var/www/html"]
Finally, setup the default container command 'CMD' and open the port for HTTP and HTTPS. We will create a new start.sh file for default 'CMD' command when container is starting. The file contains the 'supervisord' command, and we will copy the file to the new image with the 'COPY' dockerfile command.

# Configure Services and Port
COPY start.sh /start.sh
CMD ["./start.sh"]
 
EXPOSE 80 443


# create default
vim default

server {
    listen 80 default_server;
    listen [::]:80 default_server;
 
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
 
    server_name _;
 
    location / {
        try_files $uri $uri/ =404;
    }
 
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    }
 
    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny all;
    #}
}

# vim supervisord.conf

vim supervisord.conf

[unix_http_server]
file=/dev/shm/supervisor.sock   ; (the path to the socket file)
 
[supervisord]
logfile=/var/log/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)
loglevel=info                ; (log level;default info; others: debug,warn,trace)
pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=false               ; (start in foreground if true;default false)
minfds=1024                  ; (min. avail startup file descriptors;default 1024)
minprocs=200                 ; (min. avail process descriptors;default 200)
user=root             ;
 
; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface
 
[supervisorctl]
serverurl=unix:///dev/shm/supervisor.sock ; use a unix:// URL  for a unix socket
 
; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.
 
[include]
files = /etc/supervisor/conf.d/*.conf
 
 
[program:php-fpm7.0]
command=/usr/sbin/php-fpm7.0 -F
numprocs=1
autostart=true
autorestart=true
 
[program:nginx]
command=/usr/sbin/nginx
numprocs=1
autostart=true
autorestart=true

# vim start.sh

vim start.sh

#!/bin/sh
 
/usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf

chmod +x start.sh

docker build -t nginx_image .

mkdir -p /webroot

docker run -d -v /webroot:/var/www/html -p 80:80 --name hakase nginx_image

# Testing Nginx and PHP-FPM in the Container
echo '<h1>Nginx and PHP-FPM 7 inside Docker Container</h1>' > /webroot/index.html

echo '<?php phpinfo(); ?>' > /webroot/info.php

http://192.168.1.248/info.php

