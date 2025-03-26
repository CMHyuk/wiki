### Nginx 기본 문법

#### ✅ Nginx의 설정 파일 위치

1. `/etc/nginx/nginx.conf`
    - Nginx에서 가장 근본이 되는 설정 파일(루트 설정 파일)
    - 전역적으로 설정되어야 하는 내용(워커 프로세스 개수, 로그 저장 위치 등)이 포함되어 있다.

2. `/etc/nginx/conf.d/default.conf`
    - 기본 웹 서버(Web Server) 설정 파일

`/etc/nginx/conf.d/default.conf`

```shell
server {
    listen       80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
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
```

```shell
# server : '하나의 웹 사이트에 관련된 설정'을 관리하는 단위 ('server 블럭'이라고 부름)
server {
    # localhost:80으로 들어오는 요청을 이 server 블럭에서 처리하도록 설정
    # (server_name이 일치하는 server 블럭이 없는 경우 첫 번째 정의되어 있는 server 블럭을 기반으로 처리)
    # (아직은 정확히 몰라도 된다. 나중에 '멀티 도메인' 기능을 배우면 쉽게 이해할 수 있다.)
    listen       80;
    server_name  localhost;

    # / 으로 시작하는 모든 경로를 처리 (ex. /index.html)
    location / {
        # /jscode.html로 요청이 들어오면 /usr/share/nginx/html/jscode.html 파일로 응답
        root   /usr/share/nginx/html;
        
        # /로 요청이 들어오면 /usr/share/nginx/html/index.html로 응답
        # 만약 /usr/share/nginx/html/index.html이 없을 경우, /usr/share/nginx/html/index.htm으로 응답
        index  index.html index.htm;
    }

    # Nginx에서 500, 502, 503, 504의 상태 코드가 발생했을 때 /50x.html로 redirect
    error_page   500 502 503 504  /50x.html;
    
    # /50x.html과 완전히 일치하는 경로를 처리
    location = /50x.html {
        # /50x.html로 요청이 들어오면 /usr/share/nginx/html/50x.html 파일로 응답
        root   /usr/share/nginx/html;
    }
}
```

‘중괄호({...}) 형태의 구문’과 ‘세미 콜론(;)으로 끝나는 구문’ 2가지가 있다. 설정 파일을 작성할 때 세미 콜론(;)을 빠트려서 에러가 뜨는 경우가 많으니 주의하자.


#### ✅ Nginx 기본 웹 사이트 수정

1. 설정 파일 확인
   - `$ cd /etc/nginx/conf.d/default.conf`

```shell
server {
    listen       80; 
    server_name  localhost;


    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # / 으로 시작하는 모든 경로를 처리 (ex. /index.html)
    location / {
        # /jscode.html로 요청이 들어오면 /usr/share/nginx/html/jscode.html 파일로 응답
        root   /usr/share/nginx/html;
        
        # /로 요청이 들어오면 /usr/share/nginx/html/index.html로 응답
        # 만약 /usr/share/nginx/html/index.html이 없을 경우, /usr/share/nginx/html/index.htm으로 응답
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

2. 기본 웹 페이지 수정하기
   - `$ cd /usr/share/nginx/html` `$ sudo vi index.html`

/usr/share/nginx/html/index.html
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to JSCODE!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### ✅Nginx 에러 페이지 수정

1. 설정 파일 확인
   - $ cd /etc/nginx/conf.d/default.conf

/etc/nginx/conf.d/default.conf

```shell
server {
    listen       80; 
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    
    # /50x.html과 완전히 일치하는 경로를 처리
    location = /50x.html {
        # /50x.html로 요청이 들어오면 /usr/share/nginx/html/50x.html 파일로 응답
        root   /usr/share/nginx/html;
    }
}
```

2. 기본 에러 페이지 수정
   - `$ cd /usr/share/nginx/html` `$ sudo vi 50x.html`

/usr/share/nginx/html/50x.html

```html
<!DOCTYPE html>
<html>
<head>
<title>Error</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Error!</h1>
<p>Sorry, the page you are looking for is currently unavailable.<br/>
Please try again later.</p>
<p>If you are the system administrator of this resource then you should check
the error log for details.</p>
<p><em>Faithfully yours, nginx.</em></p>
</body>
</html>
```

#### ✅ Nginx 에러 체크 및 반영
```shell
# Nginx 실행 체크
$ sudo systemctl status nginx

# Nginx 설정 파일 중 문법 에러가 있는 지 체크
$ sudo nginx -t

# Nginx의 설정 파일이 바뀐 경우 아래 명령어를 입력해줘야 설정 파일이 반영된다.
$ sudo nginx -s reload

### 로그 실시간 체크
# 제대로 요청이 들어오고 있는 지 확인
$ sudo tail -f /var/log/nginx/access.log

# 에러 메시지 확인
$ sudo tail -f /var/log/nginx/error.log
```