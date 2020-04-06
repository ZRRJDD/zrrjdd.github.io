# Docker_Nginx



```shell

docker run --name nginx -d -p 80:80 -v /Users/zrrjdd/software/nginx/docker_nginx_data/conf/nginx.conf:/etc/nginx/nginx.conf -v /Users/zrrjdd/software/nginx/docker_nginx_data/log:/var/log/nginx -v /Users/zrrjdd/software/nginx/docker_nginx_data/html:/usr/share/nginx/html nginx


docker run --name nginx -d -p 80:80 -v /Users/zrrjdd/software/nginx/docker_nginx_data/conf/nginx.conf:/etc/nginx/nginx.conf nginx


docker run --name nginx -d -p 80:80  -v /Users/zrrjdd/software/nginx/docker_nginx_data/html:/usr/share/nginx/html  -v /Users/zrrjdd/software/nginx/docker_nginx_data/conf/nginx.conf:/etc/nginx/nginx.conf nginx
```


```json
location ~ .*\.(gif|jpg|png|css|js)(.*) {
  	     	proxy_pass http://127.0.0.1:8081;
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_cache_valid 200 302 24h;
                proxy_cache_valid 301 30d;
                proxy_cache_valid any 5m;
		add_header wall "hey!guys!give me a star.";
                expires 3d;              
	}
```