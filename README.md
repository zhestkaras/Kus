щwork@debian:/tmp$ sudo cat /etc/nginx/sites-enabled/moodle
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




work@debian:/tmp$ ls -la /var/www/html/moodle/
total 1512
drwxr-xr-x 65 www-data www-data   4096 Nov 25 23:21 .
drwxr-xr-x  3 root     root       4096 Nov 25 22:57 ..
drwxr-xr-x 16 www-data www-data   4096 Nov 11 09:16 admin
drwxr-xr-x  8 www-data www-data   4096 Nov 11 09:16 ai
drwxr-xr-x  5 www-data www-data   4096 Nov 11 09:16 analytics
drwxr-xr-x 14 www-data www-data   4096 Nov 11 09:16 auth
drwxr-xr-x  8 www-data www-data   4096 Nov 11 09:16 availability
drwxr-xr-x  8 www-data www-data   4096 Nov 11 09:16 backup
drwxr-xr-x  8 www-data www-data   4096 Nov 11 09:16 badges
-rw-r--r--  1 www-data www-data    311 Nov 11 09:16 behat.yml.dist
drwxr-xr-x 47 www-data www-data   4096 Nov 11 09:16 blocks
drwxr-xr-x  4 www-data www-data   4096 Nov 11 09:16 blog
-rw-r--r--  1 www-data www-data   1162 Nov 11 09:16 brokenfile.php
drwxr-xr-x  7 www-data www-data   4096 Nov 11 09:16 cache
drwxr-xr-x  7 www-data www-data   4096 Nov 11 09:16 calendar
drwxr-xr-x  5 www-data www-data   4096 Nov 11 09:16 cohort
drwxr-xr-x  5 www-data www-data   4096 Nov 11 09:16 comment
drwxr-xr-x  6 www-data www-data   4096 Nov 11 09:16 communication
drwxr-xr-x  4 www-data www-data   4096 Nov 11 09:16 competency
drwxr-xr-x  5 www-data www-data   4096 Nov 11 09:16 completion
-rw-r--r--  1 www-data www-data   2215 Nov 11 09:16 composer.json
-rw-r--r--  1 www-data www-data 180038 Nov 11 09:16 composer.lock
-rw-r--r--  1 www-data www-data  72402 Nov 11 09:16 config-dist.php
-rw-r--r--  1      644 www-data  72502 Nov 25 23:21 config.php
-rw-r--r--  1      644 www-data  72495 Nov 25 23:20 config.php.backup
drwxr-xr-x  7 www-data www-data   4096 Nov 11 09:16 contentbank
-rw-r--r--  1 www-data www-data   2651 Nov 11 09:16 CONTRIBUTING.md
-rw-r--r--  1 www-data www-data  35147 Nov 11 09:16 COPYING.txt
drwxr-xr-x 10 www-data www-data   4096 Nov 11 09:16 course
drwxr-xr-x  7 www-data www-data   4096 Nov 11 09:16 customfield
drwxr-xr-x  8 www-data www-data   4096 Nov 11 09:16 dataformat
-rw-r--r--  1 www-data www-data   2675 Nov 11 09:16 draftfile.php
-rw-r--r--  1 www-data www-data   1382 Nov 11 09:16 editmode.php
drwxr-xr-x 19 www-data www-data   4096 Nov 11 09:16 enrol
drwxr-xr-x  2 www-data www-data   4096 Nov 11 09:16 error
-rw-r--r--  1 www-data www-data   8426 Nov 11 09:16 .eslintrc
drwxr-xr-x  4 www-data www-data   4096 Nov 11 09:16 favourites
-rw-r--r--  1 www-data www-data   3919 Nov 11 09:16 file.php
drwxr-xr-x  5 www-data www-data   4096 Nov 11 09:16 files
drwxr-xr-x 18 www-data www-data   4096 Nov 11 09:16 filter
-rw-r--r--  1 www-data www-data    597 Nov 11 09:16 .gherkin-lintrc
-rw-r--r--  1 www-data www-data     92 Nov 11 09:16 .gitattributes
-rw-rw-r--  1 www-data www-data    957 Nov 11 12:40 githash.php
drwxr-xr-x  3 www-data www-data   4096 Nov 11 09:16 .github
drwxr-xr-x 12 www-data www-data   4096 Nov 11 09:16 grade
drwxr-xr-x  6 www-data www-data   4096 Nov 11 09:16 group
drwxr-xr-x  5 www-data www-data   4096 Nov 11 09:16 .grunt
-rw-r--r--  1 www-data www-data   9671 Nov 11 09:16 Gruntfile.js
drwxr-xr-x  9 www-data www-data   4096 Nov 11 09:16 h5p
-rw-r--r--  1 www-data www-data   1423 Nov 11 09:16 help_ajax.php
-rw-r--r--  1 www-data www-data   1872 Nov 11 09:16 help.php
-rw-r--r--  1 www-data www-data   5678 Nov 11 09:16 index.php
-rw-r--r--  1 root     root         20 Nov 18 22:37 info.php
drwxr-xr-x  3 www-data www-data   4096 Nov 11 09:16 install
-rw-r--r--  1 www-data www-data  25681 Nov 11 09:16 install.php
-rw-r--r--  1 www-data www-data    664 Nov 11 09:16 INSTALL.txt
drwxr-xr-x  3 www-data www-data   4096 Nov 11 09:16 iplookup
-rw-r--r--  1 www-data www-data     16 Nov 11 09:16 .jshintignore
-rw-r--r--  1 www-data www-data   1545 Nov 11 09:16 .jshintrc
drwxr-xr-x  3 www-data www-data   4096 Nov 11 09:16 lang
drwxr-xr-x 75 www-data www-data   4096 Nov 11 09:16 lib
drwxr-xr-x  2 www-data www-data   4096 Nov 11 09:16 local
drwxr-xr-x  3 www-data www-data   4096 Nov 11 09:16 login
drwxr-xr-x  4 www-data www-data   4096 Nov 11 09:16 media
drwxr-xr-x  7 www-data www-data   4096 Nov 11 09:16 message
drwxr-xr-x  5 www-data www-data   4096 Nov 11 09:16 mnet
drwxr-xr-x 25 www-data www-data   4096 Nov 11 09:16 mod
drwxr-xr-x  2 www-data www-data   4096 Nov 11 09:16 moodlenet
drwxr-xr-x  5 www-data www-data   4096 Nov 11 09:16 my
drwxr-xr-x  4 www-data www-data   4096 Nov 11 09:16 notes
-rw-r--r--  1 www-data www-data 496446 Nov 11 09:16 npm-shrinkwrap.json
-rw-r--r--  1 www-data www-data      8 Nov 11 09:16 .nvmrc
-rw-r--r--  1 www-data www-data   1861 Nov 11 09:16 package.json
drwxr-xr-x  7 www-data www-data   4096 Nov 11 09:16 payment
-rw-r--r--  1 www-data www-data    145 Nov 11 09:16 phpcs.xml.dist
drwxr-xr-x  2 www-data www-data   4096 Nov 11 09:16 .phpstorm.meta.php
-rw-r--r--  1 www-data www-data  15927 Nov 11 09:16 phpunit.xml.dist
drwxr-xr-x 14 www-data www-data   4096 Nov 11 09:16 pix
drwxr-xr-x  4 www-data www-data   4096 Nov 11 09:16 plagiarism
-rw-r--r--  1 www-data www-data   1953 Nov 11 09:16 pluginfile.php
drwxr-xr-x  7 www-data www-data   4096 Nov 11 09:16 portfolio
drwxr-xr-x  6 www-data www-data   4096 Nov 11 09:16 privacy
drwxr-xr-x 13 www-data www-data   4096 Nov 11 09:16 question
drwxr-xr-x  4 www-data www-data   4096 Nov 11 09:16 rating
-rw-r--r--  1 www-data www-data   1903 Nov 11 09:16 README.md
drwxr-xr-x 23 www-data www-data   4096 Nov 11 09:16 report
drwxr-xr-x  6 www-data www-data   4096 Nov 11 09:16 reportbuilder
drwxr-xr-x 25 www-data www-data   4096 Nov 11 09:16 repository
-rw-r--r--  1 www-data www-data   3035 Nov 11 09:16 r.php
drwxr-xr-x  4 www-data www-data   4096 Nov 11 09:16 rss
drwxr-xr-x  8 www-data www-data   4096 Nov 11 09:16 search
-rw-r--r--  1 www-data www-data    670 Nov 11 09:16 security.txt
-rw-r--r--  1 www-data www-data     67 Nov 11 09:16 .shifter.json
drwxr-xr-x  7 www-data www-data   4096 Nov 11 09:16 sms
-rw-r--r--  1 www-data www-data   4940 Nov 11 09:16 .stylelintrc
drwxr-xr-x  5 www-data www-data   4096 Nov 11 09:16 tag
drwxr-xr-x  4 www-data www-data   4096 Nov 11 09:16 theme
-rw-r--r--  1 www-data www-data   1770 Nov 11 09:16 tokenpluginfile.php
-rw-r--r--  1 www-data www-data   3367 Nov 11 09:16 TRADEMARK.txt
drwxr-xr-x  2 www-data www-data   4096 Nov 11 09:16 .upgradenotes
-rw-r--r--  1 www-data www-data 147417 Nov 11 09:16 UPGRADING.md
drwxr-xr-x  9 www-data www-data   4096 Nov 11 09:16 user
drwxr-xr-x  2 www-data www-data   4096 Nov 11 09:16 userpix
-rw-r--r--  1 www-data www-data   1641 Nov 11 09:16 version.php
drwxr-xr-x  7 www-data www-data   4096 Nov 11 09:16 webservice
work@debian:/tmp$ ls -la /var/www/html/nextcloud/
ls: cannot access '/var/www/html/nextcloud/': No such file or directory
work@debian:/tmp$ ls -la /var/www/nextcloud/
total 836
drwxr-xr-x 15 www-data www-data   4096 Nov 26 20:57 .
drwxr-xr-x  5 root     root       4096 Nov 26 19:05 ..
drwxr-xr-x 45 www-data www-data   4096 Nov 12 09:11 3rdparty
drwxr-xr-x 59 www-data www-data   4096 Nov 12 09:06 apps
-rwxr-xr-x  1 www-data www-data  26416 Nov 12 09:03 AUTHORS
-rwxr-xr-x  1 www-data www-data   3527 Nov 12 09:03 composer.json
-rwxr-xr-x  1 www-data www-data   3428 Nov 12 09:03 composer.lock
drwxr-xr-x  2 www-data www-data   4096 Nov 26 21:07 config
-rwxr-xr-x  1 www-data www-data   4219 Nov 12 09:03 console.php
-rwxr-xr-x  1 www-data www-data  34520 Nov 12 09:03 COPYING
drwxr-xr-x 24 www-data www-data   4096 Nov 12 09:11 core
-rwxr-xr-x  1 www-data www-data   1913 Nov 12 09:03 cron.php
drwxrwx---  4 www-data www-data   4096 Nov 26 21:07 data
drwxr-xr-x  2 www-data www-data  65536 Nov 12 09:03 dist
-rwxr-xr-x  1 www-data www-data   6807 Nov 26 21:07 .htaccess
-rwxr-xr-x  1 www-data www-data    331 Nov 12 09:03 index.html
-rwxr-xr-x  1 www-data www-data   3863 Nov 12 09:03 index.php
drwxr-xr-x  7 www-data www-data   4096 Nov 12 09:03 lib
drwxr-xr-x  2 www-data www-data   4096 Nov 12 09:03 LICENSES
-rwxr-xr-x  1 www-data www-data   1123 Nov 12 09:03 occ
drwxr-xr-x  2 www-data www-data   4096 Nov 12 09:03 ocs
drwxr-xr-x  2 www-data www-data   4096 Nov 12 09:03 ocs-provider
-rwxr-xr-x  1 www-data www-data   3904 Nov 12 09:03 package.json
-rwxr-xr-x  1 www-data www-data 557603 Nov 12 09:03 package-lock.json
-rwxr-xr-x  1 www-data www-data   3108 Nov 12 09:03 public.php
-rwxr-xr-x  1 www-data www-data   4961 Nov 12 09:03 remote.php
drwxr-xr-x  4 www-data www-data   4096 Nov 12 09:03 resources
-rwxr-xr-x  1 www-data www-data  30788 Nov 12 09:03 REUSE.toml
-rwxr-xr-x  1 www-data www-data     26 Nov 12 09:03 robots.txt
-rwxr-xr-x  1 www-data www-data   1490 Nov 12 09:03 status.php
drwxr-xr-x  3 www-data www-data   4096 Nov 12 09:03 themes
drwxr-xr-x  2 www-data www-data   4096 Nov 12 09:05 updater
-rwxr-xr-x  1 www-data www-data    273 Nov 12 09:03 .user.ini
-rwxr-xr-x  1 www-data www-data    446 Nov 12 09:11 version.php
work@debian:/tmp$ sudo grepo
sudo: grepo: command not found
work@debian:/tmp$ sudo grep -n "wwwroot" /var/www/html/moodle/config.php
176:$CFG->wwwroot   = 'https://moodle/quinque.local';
442:// it will be used instead of $CFG->wwwroot/course/view.php
987:// $CFG->behat_wwwroot = 'http://127.0.0.1/moodle';
1101://           'behat_wwwroot' => 'http://127.0.0.1/moodle',
1134://   $CFG->behat_ionic_wwwroot = 'http://localhost:8100';



work@debian:/tmp$ getent hosts moodle
10.0.2.15       moodle
work@debian:/tmp$ curl -I -H "Host: moodle" http://127.0.0.1/quinque.local/
HTTP/1.1 200 OK
Server: nginx
Date: Fri, 27 Nov 2026 02:51:54 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Set-Cookie: MoodleSession=a455a405bb76f592d98b9abb2e33130d; path=/quinque.local/; secure; HttpOnly; SameSite=Lax
Content-Language: en
Content-Script-Type: text/javascript
Content-Style-Type: text/css
X-UA-Compatible: IE=edge
Cache-Control: no-store, no-cache, must-revalidate, no-transform
Pragma: no-cache
Expires: Mon, 20 Aug 1969 09:23:00 GMT
Last-Modified: Fri, 27 Nov 2026 02:51:54 GMT
Accept-Ranges: none
X-Frame-Options: sameorigin

work@debian:/tmp$ sudo ss -tlnp | grep nginx
LISTEN 0      511          0.0.0.0:443       0.0.0.0:*    users:(("nginx",pid=1761,fd=6),("nginx",pid=833,fd=6))
LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=1761,fd=5),("nginx",pid=833,fd=5))
LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=1761,fd=11),("nginx",pid=833,fd=11))

sudo grep -nE 'wwwroot|dirroot|dataroot' /var/www/html/moodle/config.php

sudo nginx -T 2>/dev/null | grep -n -A15 -B3 'server_name moodle'

sudo nginx -T 2>/dev/null | grep -n -A15 -B3 'server_name quinque.local'

curl.exe -k -I --resolve moodle:443:192.168.56.101 https://moodle/quinque.local/

Get-Content C:\Windows\System32\drivers\etc\hosts | Select-String "moodle|quinque"

curl.exe -k -I --resolve quinque.local:443:192.168.56.101 https://quinque.local/

curl.exe -k -I --resolve moodle:443:192.168.56.101 https://moodle/quinque.local/
