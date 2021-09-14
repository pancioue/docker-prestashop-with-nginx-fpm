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
  https://github.com/PrestaShop/PrestaShop/pull/9047/files
  (最後參考版，不需要或不確定需不需要的都沒加上去)