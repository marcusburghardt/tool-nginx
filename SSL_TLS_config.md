Here are some security recommendations for a better SSL/TLS configuration of NGINX.

# Some Alignment
In case of NGINX, at least with my experiences, I prefer to set this configurations inside the server section of each VHost I am publishing, even more if I use NGINX as a reverse proxy.
For this configurations, I am thinking firstly in security, then in performance. Sure, you have to scale some configs if you have a high load server.
This configuration was done for a small Virtual Machine with CentOS 7. I removed the comments for better reading, but I strongly recommend you read it in your server or in the [official documentation](http://nginx.org/en/docs/http/configuring_https_servers.html).

## /etc/nginx/conf.d/ssl_<fqdn>.conf
server {  
   `#### Some other options that could be safely left in default. ####`  
   `...`  
    ssl_certificate /etc/letsencrypt/live/<fqdn>/fullchain.pem; # managed by Certbot  
    ssl_certificate_key /etc/letsencrypt/live/<fqdn>/privkey.pem; # managed by Certbot  

    ssl on;  
    ssl_session_cache  builtin:1000  shared:SSL:10m;  
    ssl_protocols TLSv1.1 TLSv1.2;  
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;  
    ssl_prefer_server_ciphers on;
 
    location / {  
        proxy_pass  <url>;  
        proxy_http_version 1.1;  
        proxy_set_header Connection "";  
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;  
        proxy_redirect off;  
        proxy_buffering off;  
        proxy_set_header        Host            <fqdn>;  
        proxy_set_header        X-Real-IP       $remote_addr;  
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;  
    }  
}  

## Restart  
This settings require a restart.  
> systemctl restart nginx  
