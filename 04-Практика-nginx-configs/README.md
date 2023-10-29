### Практика nginx configs 

Все собранные ранее приложения проходят тест на А+ :
    [ibs-corenet.truedev.ru](https://ibs-corenet.truedev.ru), [ibs-python.truedev.ru](https://ibs-python.truedev.ru), [ibs-java.truedev.ru](ibs-python.truedev.ru)

#### ibs.conf
```
limit_req_zone $binary_remote_addr zone=one:10m rate=90r/m;
limit_conn_zone $binary_remote_addr zone=addr:10m;
server {
    listen 80;
    server_name ibs-python.truedev.ru ibs-corenet.truedev.ru ibs-java.truedev.ru;
    if ($host = ibs-python.truedev.ru) {
        return 301 https://$host$request_uri;
    }
    if ($host = ibs-corenet.truedev.ru) {
        return 301 https://$host$request_uri;
    }
    if ($host = ibs-java.truedev.ru) {
        return 301 https://$host$request_uri;
    }
    include /etc/nginx/templates/letsencrypt.conf;

    return 404;
}
# todolist app
server {
    server_name ibs-python.truedev.ru;
    listen 8443 ssl http2;
    ssl_certificate /etc/letsencrypt/live/ibs-python.truedev.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/ibs-python.truedev.ru/privkey.pem;

    proxy_read_timeout 1d;
    proxy_buffering off;
    location / {
        limit_req zone=one;
        limit_conn addr 10;
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_buffering off;
    }

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header Content-Security-Policy "block-all-mixed-content";

    include /etc/nginx/ssl_params;
    include /etc/nginx/templates/letsencrypt.conf;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

    access_log /var/log/nginx/ibs-python.truedev.ru-access.log;
    error_log /var/log/nginx/ibs-python.truedev.ru-error.log;
}
```
#### ssl_params
```
ssl_protocols TLSv1.2 TLSv1.3; # Only allow TLS 1.2 and TLS 1.3
ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384';
ssl_stapling on;
ssl_stapling_verify on;
ssl_prefer_server_ciphers off; # Disable prefer_server_ciphers

```

#### lets-encrypt.conf
```
location ^~ /.well-known/acme-challenge/ {
   default_type "text/plain";
   root /var/www/letsencrypt;
}
location = /.well-known/acme-challenge/ {
   return 404;
}

```
