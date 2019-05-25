# lab2Web
### Установка NGINX Ubuntu

Установите пакеты, необходимые для подключения apt-репозитория:
```sh
    sudo apt install curl gnupg2 ca-certificates lsb-release
```
Для подключения apt-репозитория для стабильной версии nginx, выполните следующую команду:

    echo "deb http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
       | sudo tee /etc/apt/sources.list.d/nginx.list

Если предпочтительно использовать пакеты для основной версии nginx, выполните следующую команду вместо предыдущей:

    echo "deb http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
        | sudo tee /etc/apt/sources.list.d/nginx.list

Теперь нужно импортировать официальный ключ, используемый apt для проверки подлинности пакетов:

    curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -

Проверьте, верный ли ключ был импортирован:

    sudo apt-key fingerprint ABF5BD827BD9BF62

Вывод команды должен содержать полный отпечаток ключа 573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62:

    pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
          573B FD6B 3D8F BC64 1079  A6AB ABF5 BD82 7BD9 BF62
    uid   [ unknown] nginx signing key <signing-key@nginx.com>

Чтобы установить nginx, выполните следующие команды:

    sudo apt update
    sudo apt install nginx
    
### Запуск NGINX

После того, как все настройки завершены и сайт перемещен в нужную директорию, мы можем запускать наш веб-сервер следующей командой:

    sudo service nginx start

После какого-либо изменения в файле конфигурации, мы должны заставить процесс NGINX перечитать конфиг (без перезагрузки) такой командой:
	
    service nginx reload

И наконец, мы можем проверить состояние процесса командой:
	
    service nginx status
    
### Конфигурирование – основы

Основной файл настроек NGINX – это файл nginx.conf, который по умолчанию выглядит так:
    
    user www-data;
    worker_processes 4;
    pid /run/nginx.pid;

    events {
	    worker_connections 768;
	    # multi_accept on;
    }

    http {

	    sendfile on;
	    tcp_nopush on;
	    tcp_nodelay on;
	    keepalive_timeout 65;
	    types_hash_max_size 2048;
	    # server_tokens off;

	    # server_names_hash_bucket_size 64;
	    # server_name_in_redirect off;

	    include /etc/nginx/mime.types;
	    default_type application/octet-stream;

	    access_log /var/log/nginx/access.log;
	    error_log /var/log/nginx/error.log;

	    gzip on;
	    gzip_disable "msie6";

	    # gzip_vary on;
	    # gzip_proxied any;
	    # gzip_comp_level 6;
	    # gzip_buffers 16 8k;
	    # gzip_http_version 1.1;
	    # gzip_types text/plain text/css application/json application/x-javascript text/xml 
              application/xml application/xml+rss text/javascript;

	    include /etc/nginx/conf.d/*.conf;
	    include /etc/nginx/sites-enabled/*;
    }
    
Файл состоит из директив. Первая из них – events, а вторая – http. Структура из этих блоков создает основу всей конфигурации. Параметры и свойства родителей наследуется всеми дочерними блоками, и могут быть переопределены при необходимости.

### Конфигурирование – параметры

Различные параметры этого файла могут быть изменены при необходимости, но NGINX настолько прост, что все будет работать с настройками по умолчанию. Рассмотрим некоторые важные параметры конфигурационного файла:

*worker_processes: эта настройка описывает число рабочих процессов, которые сервер будет использовать. Поскольку NGINX является однопоточным, это число обычно эквивалентно количеству ядер процессора.
*worker_connections: максимальное число одновременных подключений для каждого worker_processes, оно сообщает, сколько людей одновременно NGINX сможет обслужить. Чем это число больше, тем больше запросов пользователей сможет обработать веб-сервер.
*access_log & error_log: это файлы, которые сервер использует для логирования всех ошибок и попыток входа. Эти логи нужно в первую очередь проверять при возникновении проблем и при поиске неисправностей.
*gzip: это свойство устанавливает настройки сжатия GZIP для NGINX ответов. Если включить его в сочетании с другими параметрами, производительность ресурса может значительно вырасти. Из дополнительных параметров GZIP следует отметить gzip_comp_level, который означает уровень компрессии и бывает от 1 до 10. Обычно, это значение не должно превышать 6 т. к. при большем числе прирост производительности незначительный. gzip_types – это список типов ответов, к которым будет применяться сжатие.

Ваш установленный сервер может поддерживать больше одного сайта, а файлы, которые описывают сайты вашего сервера, находятся в каталоге /etc/nginx /sites-available.

Файлы в этом каталоге не являются «живыми» – у вас может быть столько файлов, описывающих сайты, сколько вы хотите, но веб-сервер ничего не будет с ними делать, если они не будут привязаны символической ссылкой на папку /etc/nginx/site-enabled.

Это дает вам возможность быстро помещать сайты в онлайн и отправлять их в автономный режим без фактического удаления каких-либо файлов. Когда вы будете готовы перевести сайт в онлайн – делайте символическую ссылку на sites-enabled и перезапускайте процесс.
Рабочая директория

В директории sites-available находится конфигурационный файл для виртуальных хостов. Этот файл позволяет настроить веб-сервер на мультисайтовость, чтобы каждый сайт имел свой отдельный конфиг. Сайты в этом каталоге не активны и будут включены только в том случае, если мы создадим символическую ссылку на папку sites-enabled.

Теперь создайте новый файл для своего веб-приложения или отредактируйте дефолтный. Типичный конфиг выглядит так:

    upstream remoteApplicationServer {
        server 10.10.10.10;
    }

    upstream remoteAPIServer {
        server 20.20.20.20;
        server 20.20.20.21;
        server 20.20.20.22;
        server 20.20.20.23;
    }


    server {
        listen 80;
        server_name www.customapp.com customapp.com
        root /var/www/html;
        index index.html

            location / {
                alias /var/www/html/customapp/;
                try_files $uri $uri/ =404;
            }

            location /remoteapp {
                proxy_set_header   Host             $host:$server_port;
                proxy_set_header   X-Real-IP        $remote_addr;
                proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
                proxy_pass http://remoteAPIServer/;
            }

            location /api/v1/ {
                proxy_pass https://remoteAPIServer/api/v1/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
                proxy_redirect http:// https://;
            }
    }

Этот файл имеет похожую структуру на nginx.conf – все та же блочная иерархия (и все они также вложены внутри HTTP-контекста nginx.conf, поэтому они также наследуют все от него).

Блок server описывает специфику работы виртуального сервера, который будет обрабатывать запросы пользователей. У вас может быть несколько таких блоков, и веб-сервер сам будет выбирать между ними на основании директив listen и server_name.

Внутри блока сервера мы определяем несколько контекстов location, которые используются для определения того, как обрабатывать клиентские запросы. Всякий раз, когда приходит запрос, сервер будет пытаться сопоставить свой URI с одним из этих определений location и обрабатывать его соответствующим образом.

Существует много важных директив, которые могут быть использованы, среди них:

**try_files**: указывает на статический файл, находящийся в root-директории.
**proxy_pass**: запрос будет отправлен на указанный прокси-сервер.
**rewrite**: переписывает входящий URI основываясь на регулярном выражении, чтобы другой блок обработал его.

Блок upstream определяет пул серверов, на который будут отправляться запросы. После того, как мы создадим блок upstream и определим сервер внутри него, мы можем ссылаться на него по имени внутри наших блоков location. Кроме того, в upstream контексте может быть назначено множество серверов, поэтому NGINX будет выполнять некоторую балансировку нагрузки при проксировании запросов.
