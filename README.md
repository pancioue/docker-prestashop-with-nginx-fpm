* container nginx,app都要綁定 volumes ```./app:/var/www/html``` (volume綁定需要一些時間)

* __www.conf__ 是允許prestashop app 可以接受.css .js等檔名 (security.limit_extensions)
  這是因為一開始 __site.conf__設定錯誤，把所有的request都送到php app
  實際上設定好後，應只有```.php```的request會送到php app其他如 ```.jpg .css .js ```的靜態檔案
  都是送到nginx本機，這也是為何上面提到的volume兩邊都必須綁定。
  此檔案實際不需要，但過程中扮演重要的角色，故保留。

* __./nginx/nginx.conf__ 檔案最主要是為了修改timeout，否則會出現504 timeout
```
fastcgi_connect_timeout     75;
fastcgi_read_timeout           1000;
fastcgi_send_timeout         1000;
```

* __site.conf__ 設定參考網址：
1. https://github.com/PrestaShop/PrestaShop/pull/9047/files. 
2. https://github.com/PrestaShop/PrestaShop/blob/bffb73a2f893f8e3caf613ba51d2f8f2d5c7c8e5/docs/server_config/nginx.conf.dist


Friendly url
-----------
```
location / {
    try_files $uri $uri/ /index.php?$args;
    
    # Images
    rewrite ^/([0-9])(-[_a-zA-Z0-9-]*)?(-[0-9]+)?/.+.jpg$ /img/p/$1/$1$2$3.jpg last;
    rewrite ^/([0-9])([0-9])(-[_a-zA-Z0-9-]*)?(-[0-9]+)?/.+.jpg$ /img/p/$1/$2/$1$2$3$4.jpg last;
    rewrite ^/([0-9])([0-9])([0-9])(-[_a-zA-Z0-9-]*)?(-[0-9]+)?/.+.jpg$ /img/p/$1/$2/$3/$1$2$3$4$5.jpg last;
    rewrite ^/([0-9])([0-9])([0-9])([0-9])(-[_a-zA-Z0-9-]*)?(-[0-9]+)?/.+.jpg$ /img/p/$1/$2/$3/$4/$1$2$3$4$5$6.jpg last;
    rewrite ^/([0-9])([0-9])([0-9])([0-9])([0-9])(-[_a-zA-Z0-9-]*)?(-[0-9]+)?/.+.jpg$ /img/p/$1/$2/$3/$4/$5/$1$2$3$4$5$6$7.jpg last;
    rewrite ^/([0-9])([0-9])([0-9])([0-9])([0-9])([0-9])(-[_a-zA-Z0-9-]*)?(-[0-9]+)?/.+.jpg$ /img/p/$1/$2/$3/$4/$5/$6/$1$2$3$4$5$6$7$8.jpg last;
    rewrite ^/([0-9])([0-9])([0-9])([0-9])([0-9])([0-9])([0-9])(-[_a-zA-Z0-9-]*)?(-[0-9]+)?/.+.jpg$ /img/p/$1/$2/$3/$4/$5/$6/$7/$1$2$3$4$5$6$7$8$9.jpg last;
    rewrite ^/([0-9])([0-9])([0-9])([0-9])([0-9])([0-9])([0-9])([0-9])(-[_a-zA-Z0-9-]*)?(-[0-9]+)?/.+.jpg$ /img/p/$1/$2/$3/$4/$5/$6/$7/$8/$1$2$3$4$5$6$7$8$9$10.jpg last;
    rewrite ^/c/([0-9]+)(-[.*_a-zA-Z0-9-]*)(-[0-9]+)?/.+.jpg$ /img/c/$1$2$3.jpg last;
    rewrite ^/c/([a-zA-Z_-]+)(-[0-9]+)?/.+.jpg$ /img/c/$1$2.jpg last;
    
    # Web Services
    rewrite ^/api/?(.*)$ /webservice/dispatcher.php?url=$1 last;
}
    
location /admin/ { #Change this to your admin folder
    if (!-e $request_filename) {
        rewrite ^/.*$ /admin/index.php last; #Change this to your admin folder
    }
}

# PHP FPM part
location ~ [^/]\.php(/|$) {
    # Verify that the file exists, redirect to index if not
    try_files $fastcgi_script_name /index.php$uri&$args;

    fastcgi_index  index.php;

    # Envirnoment variables for PHP
    fastcgi_split_path_info ^(.+\.php)(/.+)$;

    include       fastcgi_params;

    fastcgi_param PATH_INFO       $fastcgi_path_info;
    fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

    # [REQUIRED EDIT] Connection to PHP-FPM - choose one
    fastcgi_pass app:9000;

    fastcgi_keep_conn on;
    fastcgi_read_timeout 30s;
    fastcgi_send_timeout 30s;

    # In case of long loading or 502 / 504 errors
    # fastcgi_buffer_size 256k;
    # fastcgi_buffers 256 16k;
    # fastcgi_busy_buffers_size 256k;
    client_max_body_size 10M;
}
```
* -e 如果filename存在，則為真  
* last与break都停止處理後續rewrite指令集，最大的不同是，last会重新發起一个新请求，并重新匹配location
* - ~*: 不區分大小寫
  - ~: 區分大小寫
* - `^`: 表示從開頭匹配，在括號中代表相反(否定)的意思
  - `[^/]`: 表示不匹配`/`
* `\`: 跳脫字元
