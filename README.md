vagrant-puppet-magento
======================

Working server for magento under Nginx

    server {
        listen 80 default;
        #listen 443 default ssl;
        #ssl_certificate     /etc/nginx/conf.d/DOMAIN.crt;
        #ssl_certificate_key /etc/nginx/conf.d/DOMAIN.key;
        server_name dev.wedo.co.uk;
        root /var/www;
    
        location / {
            index index.html index.php; ## Allow a static html file to be shown first
            try_files $uri $uri/ @handler; ## If missing pass the URI to Magento's front handler
            expires 30d; ## Assume all files are cachable
        }
    
        ## These locations would be hidden by .htaccess normally
        location ^~ /app/                { deny all; }
        location ^~ /includes/           { deny all; }
        location ^~ /lib/                { deny all; }
        location ^~ /media/downloadable/ { deny all; }
        location ^~ /pkginfo/            { deny all; }
        location ^~ /report/config.xml   { deny all; }
        location ^~ /var/                { deny all; }
    
        location /var/export/ { ## Allow admins only to view export folder
            auth_basic           "Restricted"; ## Message shown in login window
            auth_basic_user_file htpasswd; ## See /etc/nginx/htpassword
            autoindex            on;
        }
    
        location  /. { ## Disable .htaccess and other hidden files
            return 404;
        }
    
        location @handler { ## Magento uses a common front handler
            rewrite / /index.php;
        }
    
        location ~ .php/ { ## Forward paths like /js/index.php/x.js to relevant handler
            rewrite ^(.*.php)/ $1 last;
        }
    
        location ~ .php$ { ## Execute PHP scripts
            if (!-e $request_filename) { rewrite / /index.php last; } ## Catch 404s that try_files miss
    
            expires        off; ## Do not cache dynamic content
            fastcgi_pass   unix:/var/run/php5-fpm.sock;
            fastcgi_param  HTTPS $fastcgi_https;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  MAGE_RUN_CODE default; ## Store code is defined in administration > Configuration > Manage Stores
            fastcgi_param  MAGE_RUN_TYPE store;
            include        fastcgi_params; ## See /etc/nginx/fastcgi_params
        }
    }
