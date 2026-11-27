work@debian:/tmp$ sudo cat /etc/nginx/sites-enabled/moodle
server {
    listen 443 ssl;
    server_name quinque.local;

    ssl_certificate /etc/nginx/ssl/bundle.pem;
    ssl_certificate_key /etc/nginx/ssl/private.key;

    root /var/www/html/moodle;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ [^/]\.php(/|$) {
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        fastcgi_pass unix:/run/php/php8.4-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;

        fastcgi_param HTTPS on;
        fastcgi_param HTTP_X_FORWARDED_PROTO https;
     }

    location ^~ /phpmyadmin/ {
        alias /usr/share/phpmyadmin/;
        index index.php;
        location ~ ^/phpmyadmin/(.+.php)$ {
            alias /usr/share/phpmyadmin/$1;
            fastcgi_pass unix:/run/php/php8.4-fpm.sock;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_param HTTPS on;
            fastcgi_param HTTP_X_FORWARDED_PROTO https;
        }
     }
}
work@debian:/tmp$ sudo cat /etc/nginx/sites-enabled/moodle-new
server {
    listen 80;
    server_name moodle;

    root /var/www/html;
    index index.php index.html;

    location = /quinque.local {
        return 301 /quinque.local/;
    }

    location /quinque.local/ {
        try_files $uri $uri/ /quinque.local/index.php?$query_string;
    }

    location ~ ^/quinque\.local/.+\.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.4-fpm.sock;
    }
}
work@debian:/tmp$ sudo cat /etc/nginx/sites-enabled/nextcloud
server {
    listen 80;
    listen [::]:80;

    server_name quinque.local;

    root /var/www/nextcloud;
    index index.php index.html /index.php$request_uri;

    client_max_body_size 512M;
    fastcgi_buffers 64 4K;
    fastcgi_read_timeout 300;
    fastcgi_connect_timeout 60;
    fastcgi_send_timeout 300;


    location / {
        try_files $uri $uri/ /index.php$request_uri;
    }

    location ~ \.php(?:$|/) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        set $path_info $fastcgi_path_info;

        try_files $fastcgi_script_name =404;

        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;

        fastcgi_pass unix:/run/php/php8.4-fpm.sock;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
