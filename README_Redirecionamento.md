# Correção de redirecionamento de aplicação com loadbalancer | React + Nginx + Docker |
## Explicação do método utilizado

utilizaremos: 1 loadbalancer e 5 nodes

Sabendo disso precisamos de um arquivo de configuração para o loadbalancer e mais um para os nodes. A seguir criaremos as configurações dos containers e subiremos indicando os volumes com suas determinadas configurações corrigidas.
## Criação de arquivos de configuração
### Arquivo dos nós

```
sudo mkdir -p /home/user/node-config
sudo nano /home/user/loadbalancer/default.conf
```
e colar isso:
```
real_ip_header X-Real-IP;

set_real_ip_from 172.17.0.0/16;

real_ip_recursive on;

log_format realip '$remote_addr - $remote_user [$time_local] '
                  '"$request" $status $body_bytes_sent '
                  '"$http_referer" "$http_user_agent"';

server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;

    access_log /var/log/nginx/access.log realip;

    location / {
        try_files $uri /index.html;

    }
}
```
### Arquivo do loadbalancer
```
sudo mkdir -p /home/user/loadbalancer
sudo nano /home/user/loadbalancer/default.conf
```
e cole isso:
```
upstream nodes {
    server 172.17.0.3;
    server 172.17.0.4;
    server 172.17.0.5;
    server 172.17.0.6;
    server 172.17.0.7;
}

server {
    listen 80;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_pass http://nodes;

    }

    access_log /var/log/nginx/nginx-access.log;
}
```
## Subir os Containers
loadbalancer:
```
docker run -d -p 84:80 --name loadbalancer -v /home/user/loadbalancer/default.conf:/etc/nginx/conf.d/default.conf nginx:1.29.3-alpine
```
Nodes:
```
docker run -d --name node1 -v /var/www/html:/usr/share/nginx/html -v /home/user/node-config/default.conf:/etc/nginx/conf.d/default.conf nginx:1.29.3-alpine
docker run -d --name node2 -v /var/www/html:/usr/share/nginx/html -v /home/user/node-config/default.conf:/etc/nginx/conf.d/default.conf nginx:1.29.3-alpine
docker run -d --name node3 -v /var/www/html:/usr/share/nginx/html -v /home/user/node-config/default.conf:/etc/nginx/conf.d/default.conf nginx:1.29.3-alpine
docker run -d --name node4 -v /var/www/html:/usr/share/nginx/html -v /home/user/node-config/default.conf:/etc/nginx/conf.d/default.conf nginx:1.29.3-alpine
docker run -d --name node5 -v /var/www/html:/usr/share/nginx/html -v /home/user/node-config/default.conf:/etc/nginx/conf.d/default.conf nginx:1.29.3-alpine
```
