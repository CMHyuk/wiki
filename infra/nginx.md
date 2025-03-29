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

### Spring Boot 서버에 HTTPS 적용하기

- HTTPS 인증서 발급받기
  - `sudo certbot --nginx -d api.jscode.p-e.kr // 반드시 도메인을 먼저 연결한 뒤에 위 명령어를 쳐야 정상 작동한다.`
- Nginx 설정 파일 확인

```shell
server {
        server_name api.jscode.p-e.kr;

        location / {
                proxy_pass http://localhost:8080;
        }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api.jscode.p-e.kr/fullchain.pem # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api.jscode.p-e.kr/privkey.pm; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = api.jscode.p-e.kr) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80;
        server_name api.jscode.p-e.kr;
    return 404; # managed by Certbot
    
}
```

### 리버스 프록시

**✅ 프록시(Proxy)란?**  
프록시(Proxy)란 ‘중계(중간에서 연결해주는 것)’의 의미를 가진다. 그러면 프록시 서버(Proxy Server)는 중간 역할을 해주는 서버다.  
클라이언트(사용자)와 서버가 직접 통신하지 않고 중간 역할을 하는 서버를 거쳐서 통신을 하는데, 중간 역할을 하는 서버를 보고 프록시 서버(Proxy Server)라고 부른다.

**✅ 포워드 프록시(Forward Proxy)란?**  
보내려고 하는 요청을 관리 또는 보안 처리를 위한 용도로 사용하는 서버를 포워드 프록시(Foward Proxy) 서버라고 얘기를 한다. 포워드 프록시 서버의 대표적인 예시로 회사 방화벽이 있다. 회사 내부에 있는 컴퓨터로 ChatGPT에 접속하려는데 차단되는 경우가 있다. 포워드 프록시 서버가 보내려고 하는 요청을 감시하면서 위험하다고 판단되는 사이트에 접속하지 못하게 차단 설정을 한 것이다.

**✅ 리버스 프록시(Reverse Proxy)란?**  
들어오는 요청을 관리 또는 보안 처리를 하기 위한 용도로 사용하는 서버를 리버스 프록시(Reverse Proxy) 서버라고 얘기를 한다. 리버스 프록시 서버의 대표적인 예시가 HTTPS 처리, 요청 수 제한, 로드 밸런싱을 하는 용도로 사용하는 Nginx가 있다. 리버스 프록시 서버가 들어오는 요청의 보안 처리를 하기 위해 HTTPS 처리를 한다. 그리고 들어오는 요청을 감시하다가 일정 요청 수 이상을 보낼 때도 차단을 하게끔 설정할 수 있다. 또한 들어오는 요청을 여러 대의 서버로 분배해주는 역할인 로드밸런싱 기능을 하게끔 셋팅할 수 있다.

### 요청 처리율 제한

```shell
# limit_req_zone : 요청 수를 제한하기 위한 메모리 공간(zone)과 요청 속도(rate)를 정의
# $binary_remote_addr : 요청 수를 제한하는 기준을 클라이언트의 IP로 설정
# zone=mylimit:10m : 메모리 공간(zone)의 이름을 mylimit이라고 지정, 
                     메모리 공간의 크기를 10MB 제한 (약 16만개의 IP 주소를 관리할 수 있음)
# rate=3r/s : 1초에 최대 3개의 요청만 허용
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=3r/s;

server {
				# limit_req_zone에서 정의한 mylimit이라는 조건을 이 server 블럭에 적용
        limit_req zone=mylimit;
        # 요청이 제한됐을 때 429(Too Many Requests) 상태 코드를 반환
        limit_req_status 429;
        server_name api.jscode.p-e.kr;

        location / {
                proxy_pass http://localhost:8080;
        }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api.jscode.p-e.kr/fullchain.pem # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api.jscode.p-e.kr/privkey.pm; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = api.jscode.p-e.kr) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
    
        listen 80;
        server_name api.jscode.p-e.kr;
    return 404; # managed by Certbot

}
```

### 로드밸런서

```shell
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=3r/s;

# 로드 밸런싱 대상 서버들을 upstream이라는 그룹으로 묶음
# upstream 그룹의 이름은 backend라고 지정
upstream backend {
        server localhost:8080;
        server localhost:8081;
}

server {
        limit_req zone=mylimit;
        limit_req_status 429;
        server_name api.jscode.p-e.kr;

				# upstream 그룹에서 지정한 서버들로 요청이 분산됨
        location / {
                proxy_pass http://backend;
        }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api.jscode.p-e.kr/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api.jscode.p-e.kr/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = api.jscode.p-e.kr) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80;
        server_name api.jscode.p-e.kr;
    return 404; # managed by Certbot


}
```
