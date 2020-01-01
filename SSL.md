 # SSL安裝

 ## OPENSSL

 >安裝git後，可以從安裝目錄C:\Program Files\Git\usr\bin中找到openssl.exe

 ## 建立SSL

 >建立一個新的檔案ssl.conf並把他放入與openssl.exe同一層目錄

### ssl.conf內容如下:

>[req]
<br/>prompt = no
<br/>default_md = sha256
<br/>default_bits = 2048
<br/>distinguished_name = dn
<br/>x509_extensions = v3_req
<br/>
<br/>[dn]
<br/>C = TW
<br/>ST = Taiwan
<br/>L = Taipei
<br/>O = Duotify Inc.
<br/>OU = IT Department
<br/>emailAddress = admin@example.com
<br/>
<br/>CN = localhost
<br/>[v3_req]
<br/>subjectAltName = @alt_names
<br/>[alt_names]
<br/>DNS.1 = *.localhost
<br/>DNS.2 = localhost

>開啟CMD(系統管理員)出入<br/>
>cd C:\Program Files\Git\usr\bin
>openssl req -x509 -new -nodes -sha256 -utf8 -days 3650 -newkey rsa:2048 -keyout server.key -out server.crt -config ssl.conf<br/>
>此時已經建立了SSL憑證，但是是未加密。輸入一下指令才會加密。<br/>
>openssl pkcs12 -export -in server.crt -inkey server.key -out server.pfx<br/>
>產生了server.pfx與server.crt與server.key

![憑證](/assets/image/憑證.JPG)

## NGINX安裝SSL

1. 先把server.key與server.crt複製到ngisx文件夾下的SSL(自行建立)
2. 開啟nginx.conf

### nginx.conf設定

    #user  nobody;
    worker_processes  1;

    #error_log  logs/error.log;
    #error_log  logs/error.log  notice;
    #error_log  logs/error.log  info;

    #pid        logs/nginx.pid;


    events {
        worker_connections  1024;
    }
    

    http {
        include       mime.types;
        default_type  application/octet-stream;

        #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        #                  '$status $body_bytes_sent "$http_referer" '
        #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

        sendfile        on;
        #tcp_nopush     on;

        #keepalive_timeout  0;
        keepalive_timeout  65;

        #gzip  on;

        server {
            listen       80;
            server_name  localhost;
    
            #http跳https
    		rewrite ^(.*) https://$host$1 permanent;

    		#charset koi8-r;

            #access_log  logs/host.access.log  main;

            #location / {
            #    root   html;
            #    index  index.html index.htm;
            #}

            #error_page  404              /404.html;

            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }

            # proxy the PHP scripts to Apache listening on 127.0.0.1:80
            #
            #location ~ \.php$ {
            #    proxy_pass   http://127.0.0.1;
            #}

            # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
            #
            #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    server {
        listen       443 ssl;
        server_name  localhost;

		# 憑證與金鑰的路徑
		ssl_certificate C:/nginx-1.16.1/SSL/server.crt;
		ssl_certificate_key C:/nginx-1.16.1/SSL/server.key;
        
        #    ssl_session_cache    shared:SSL:1m;
        #    ssl_session_timeout  5m;

        #    ssl_ciphers  HIGH:!aNULL:!MD5;
        #    ssl_prefer_server_ciphers  on;

            location / {
                root   html;
                index  index.html index.htm;
            }
        }

    }

-------

最後打開遊覽器輸入http:localhost，會自動跳到https:localhost