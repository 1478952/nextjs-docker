user nginx; # Nginx 워커 프로세스가 실행될 사용자를 지정합니다. 이 경우, nginx 사용자로 실행됩니다.
worker_processes auto; # 자동으로 워커 프로세스 수를 설정합니다. 현재 시스템의 코어 수에 따라 자동으로 설정됩니다.
error_log /var/log/nginx/error.log; # 오류 로그 파일의 경로를 설정합니다.
pid /run/nginx.pid; # Nginx 마스터 프로세스의 PID 파일 경로를 설정합니다.

include /usr/share/nginx/modules/\*.conf;

events { # Nginx 이벤트 설정 블록으로, 워커 프로세스의 연결 수 등을 설정합니다.
worker_connections 1024;
}

http { # HTTP 설정 블록으로, 웹 서버의 주요 설정을 포함합니다.
log_format main '$remote_addr - $remote_user [$time_local] "$request" ' # 로그 형식을 정의합니다.
                      '$status $body_bytes_sent "$http_referer" '
'"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main; # 접근 로그 파일의 경로를 설정합니다.

    sendfile            on; # 파일 전송 및 TCP 설정을 제어합니다.
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65; # 클라이언트와의 keep-alive 연결 시간을 설정합니다.
    types_hash_max_size 4096; # MIME 타입 해시 테이블 크기를 설정합니다.

    include             /etc/nginx/mime.types; # 추가 설정 파일을 포함합니다.
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    	server { # 개별 웹 서버 블록으로, 서버와 관련된 설정을 정의합니다.
        listen       80; # 서버가 리스닝하는 포트 및 IP 주소를 지정합니다.
        server_name devyeh.com www.devyeh.com; # 서버의 도메인 이름을 지정합니다.

        # redirect https setting
        if ($http_x_forwarded_proto != 'https') {
                return 301 https://$host$request_uri;
        }

       location / { # "/" 경로에 대한 설정 블록으로, 여기서는 프론트엔드 애플리케이션에 대한 프록시 설정이 이루어집니다.
               proxy_pass http://front:3000; # 프록시 서버로 전달할 요청 주소를 지정합니다.

               proxy_set_header Host $host; # 프록시에 전달되는 헤더를 설정합니다.
               proxy_set_header X-Real-IP $remote_addr;
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header X-Forwarded-Proto $scheme;

               proxy_read_timeout 300; # 프록시 연결의 타임아웃을 설정합니다.
               proxy_connect_timeout 300;
               proxy_send_timeout 300;
        }

        location /api { # "/api" 경로에 대한 설정 블록으로, 여기서는 백엔드 API에 대한 프록시 설정이 이루어집니다.
                proxy_set_header X-Real-IP $remote_addr; #  프록시에 전달되는 헤더를 설정합니다.
                proxy_set_header HOST $http_host;
                proxy_set_header X-NginX-Proxy true;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

                proxy_pass http://web:8080; #  API 요청을 처리할 백엔드 서버 주소를 지정합니다.
                proxy_redirect off;
        }

    }

}
