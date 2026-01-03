# NGINX
This repo is for NGINX tutorial
<br />
Always run the **nginx** **command** from **nginx** **folder**
<br />
nginx -v
<br />
start nginx
<br />
nginx -s reload
<br />
nginx -s stop
<br />

NGINX is a webserver acts as ReverseProxy doing functionality like 
1. Routing
2. LoadBalancer
3. SSL
4. Config OAuth2 Proxy etc

<br />
we can see the **error** **log** by enabling the following lines.

<br />

error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;

<br />

<br/>

Here we will see some sample configuration of how it will work.
<br />

        listen       80;
        server_name  localhost;
<br />

From the above config you can see the nginx is listening in 80 port in localhost, so user will request the localhost:80

<br />

Routing
------------------
When we are requesting a server with url, based on url Nginx redirects the request to corresponding backend server. To do that we are using **proxy_pass** directive inside **location**
<br />

        location / {
            proxy_pass http://localhost:4200/;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
<br />

LoadBalancer
------------------
When we have 2 or more backend servers and we want to redirect to the different servers based on load, then we can use the load balance feature. To do that we are using **upstream**.
When giving name for **upstream** give it with **hypen** instead of **underscore**.

  upstream **backend-servers** {
        server localhost:4200 weight=1;
        server localhost:4201 weight=1;
    }

  location / {
            proxy_pass http://**backend-servers**;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

SSL
-------------------
When we are requesting a site using Https, we are using this we have to give certificate and private key in config, we can use the **openssl** tool to create certificate for testing.
install the openssl for windows

**openssl command for create certificate**
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout private.key -out certificate.crt

**paste** the certificate and private key in nginx folder under "cert", now we can see the config

       ssl_certificate      ../cert/certificate.crt;
       ssl_certificate_key  ../cert/private.key;

**Full Config for HTTPS**
<br />

 server {
       listen       443 ssl;
       server_name  localhost;

       ssl_certificate      ../cert/certificate.crt;
       ssl_certificate_key  ../cert/private.key;

       ssl_session_cache    shared:SSL:1m;
       ssl_session_timeout  5m;

       ssl_ciphers  HIGH:!aNULL:!MD5;
       ssl_prefer_server_ciphers  on;

       
        location / {
            proxy_pass http://backend-servers;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        location /react {
            proxy_pass http://localhost:4202/;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
        
    }

<br />

**Full Config for Http**

<br />

server {
       listen       80;
       server_name  localhost;
       
        location / {
            proxy_pass http://backend-servers;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        location /react {
            proxy_pass http://localhost:4202/;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
        
  }
