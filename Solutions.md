- Сконфигурируйте nginx сервер таким образом, чтобы запросы проходили через
nginx и перенаправлялись на сервер из лабораторной работы №1.
```
http {
    include mime.types;
    server{
    	listen 8000; 
    	location /server {
	proxy_set_header proxied nginx;
	proxy_pass http://localhost:5000/;
     	}    
   	location /nginxorg {
	proxy_pass https://nginx.org/;
     	}
    }
}
```

- Используйте nginx отдачи статического контента. Как изменилось время ответа
сервера?
```
http {
    server{
    	listen 8080;
   	location /{
    	root /data/www;
	}
    	location /images{
    	root /data;
	}
    }
}
```

- Настройте кеширование и gzip сжатие файлов. Как изменилось время ответа
сервера?

**Настройки GZip**
```    
    gzip on;
    gzip_min_length 100;
    gzip_comp_level 3;
    gzip_types text/plain;
    gzip_types text/css;
    gzip_types text/javascript;
    gzip_disable "msie6";
```

**Сохранение файла с использованием GZip**
```
curl -H 'Accept-Encoding:gzip,deflate' http://127.0.0.1:8080/style.css > style_compress.css
```
**Результат**
```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   137    0   137    0     0   6227      0 --:--:-- --:--:-- --:--:--  6523
```
**Сохранение файла без использования GZip**
```
curl http://127.0.0.1:8080/style.css > style_compress.cs
```
**Результат**
```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   148  100   148    0     0   7789      0 --:--:-- --:--:-- --:--:--  7789
```
*Время загрузки значительно улучшилось*

**Настройка кеширования**
```
Server Software:        nginx/1.16.0
Server Hostname:        localhost
Server Port:            8080

Document Path:          /
Document Length:        178 bytes

Concurrency Level:      10
Time taken for tests:   0.497 seconds
Complete requests:      100
Failed requests:        0
Total transferred:      32300 bytes
HTML transferred:       17800 bytes
Requests per second:    201.41 [#/sec] (mean)
Time per request:       49.650 [ms] (mean)
Time per request:       4.965 [ms] (mean, across all concurrent requests)
Transfer rate:          63.53 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       1
Processing:    27   49  34.5     35     153
Waiting:       27   49  34.3     35     153
Total:         27   49  34.6     35     153

Percentage of the requests served within a certain time (ms)
  50%     35
  66%     43
  75%     47
  80%     62
  90%    138
  95%    151
  98%    153
  99%    153
 100%    153 (longest request)
 ```
**Применяем следующие настройки:**
```
http {    
    fastcgi_cache_path /tmp/nginx_cache levels=1:2 keys_zone=microcache:10m max_size=500m;
    fastcgi_cache_key "$scheme$request_method$host$request_uri";
    add_header microcache-status $upstream_cache_status;    
    server{
    listen 8080;    
    location /{    
    include fastcgi_params;
    include fastcgi.conf;
    fastcgi_cache microcache;
    fastcgi_cache_valid 200 60m;
    fastcgi_cache_bypass $no_cache;
    fastcgi_no_cache $no_cache;
    fastcgi_pass 127.0.0.1:5000;
    }
    set $no_cache=0;  
    if($request_method=POST) {set $no_cache=1;}
    if($request_uri!=""){set $no_cache=1;}
    if($request_uri~*"wp-admin"){set $ no_cache=1;}
    }
}
```
**Результат**
```
Server Software:        nginx/1.16.0
Server Hostname:        localhost
Server Port:            8080

Document Path:          /
Document Length:        157 bytes

Concurrency Level:      10
Time taken for tests:   0.162 seconds
Complete requests:      100
Failed requests:        0
Non-2xx responses:      100
Total transferred:      30900 bytes
HTML transferred:       15700 bytes
Requests per second:    616.84 [#/sec] (mean)
Time per request:       16.212 [ms] (mean)
Time per request:       1.621 [ms] (mean, across all concurrent requests)
Transfer rate:          186.14 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   1.0      0       6
Processing:     2   15   6.5     13      33
Waiting:        2   15   6.1     13      32
Total:          3   15   7.3     14      38

Percentage of the requests served within a certain time (ms)
  50%     14
  66%     15
  75%     16
  80%     17
  90%     34
  95%     34
  98%     34
  99%     38
 100%     38 (longest request)
```
*Время загрузки значительно улучшилось*

- Запустите еще 2 инстанса вашего сервера из лабораторной работы №1,
настройте перенаправление таким образом, чтобы на серверы приходили
запросы в соотношении 3:1.

Запускаем три сервера php
```
php -S localhost:10001 s1
php -S localhost:10002 s2
php -S localhost:10003 s3
```
Создадим конфигурационный файл с использованием `upstream` - это контекст в котором сгрупперовано несколько серверов:
```
http{
    upstream php_servers{
        server localhost:10001;
        server localhost:10002;
        server localhost:10003;
    }
    server{
       listen 8888;
       location /{
        proxy_pass 'http://php_servers';
        }
    }
}
```
`ip_hash` - 
`least_conn` -

**Запускаем цикл из консоли**
```
while sleep 0.5;do curl http://localhost:8888;done
```
**Результат**
```
Server 3
Server 1
Server 2
Server 3
Server 1
Server 2
Server 3
Server 1
Server 2
Server 3
Server 1
Server 2
```

- Настройте отдачу страницы о состоянии сервера
```
location=/basic_status{
stub_status;
}
```
**Результат**
```
Active connections: 1 
server accepts handled requests
 764 764 802 
Reading: 0 Writing: 1 Waiting: 0 
```
