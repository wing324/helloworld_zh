# Setup React app on AWS EC2 instance

#### 1. yarn run build

First of all, we should build our project by running `yarn run build`, and we will get a director under our project root directory call `dist` or `build`, then we will upload the `dist` folder to our AWS EC2 instance.
With this article, I will put `dist` director to `/data` director on AWS.



#### 2. yum install Nginx

#### 3. Change Nginx config

**Normally, we won't use root user to do it, but at here I will use root user for convenience.**

```shell
vim /etc/nginx/nginx.conf

## updated config 
user root; ## updated
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;


events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
	include /etc/nginx/vhost/*.conf;  ### updated

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

#        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

       location / {
        }


        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

```

#### 4. Make own Nginx config

- Create `/etc/nginx/vhost` directory to save our own Nginx config

  ```she
  mkdir /etc/nginx/vhost
  ```

- Make own Nginx config

  ```shell
  vim /etc/nginx/vhost/react.conf
  
  ### updated
  server {
             listen      3000;
             server_name  localhost;
             root /data/dist;
             location / {
                       try_files $uri $uri/ /index.html;
                  }
             error_page   500 502 503 504  /50x.html;
             location = /50x.html {
                  root   html;
          }
  }
  ```

#### 5. Change user and mode for build folder

```shell
chown -R root:root /data/dist
chmod 755 /data/dist
```

#### 6. Run your Nginx

```shell
systemctl start nginx
```

#### 7. Disable SELinux

```shell
# Temporary
setenforce 0
# permanent
vim /etc/selinux/config

# change SELINUX=enforcing to SELINUX=disabled
```



#### Now you can use IP and port to get the React App.