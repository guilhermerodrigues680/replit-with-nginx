docker run -itv `pwd`:/host ubuntu:18.04 /bin/bash

https://www.cyberciti.biz/faq/bash-shell-change-the-color-of-my-shell-prompt-under-linux-or-unix/

export PS1="\[\e[32m\][\[\e[m\]\[\e[31m\]\u\[\e[m\]\[\e[33m\]@\[\e[m\]\[\e[32m\]\h\[\e[m\]:\[\e[36m\]\w\[\e[m\]\[\e[32m\]]\[\e[m\]\[\e[32;47m\]\\$\[\e[m\] "



### apt-get install -y gcc

```
[root@2b3abb347c8f:/host/nginx-1.18.0]# ./configure 
checking for OS
 + Linux 4.4.0-96-generic x86_64
checking for C compiler ... not found

./configure: error: C compiler cc is not found
```

### apt-get install libpcre3 libpcre3-dev

```
./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.
```

### apt-get install libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev

```
./configure: error: the HTTP gzip module requires the zlib library.
You can either disable the module by using --without-http_gzip_module
option, or install the zlib library into the system, or build the zlib library
statically from the source with nginx by using --with-zlib=<path> option.
```

### ./configure --error-log-path=stderr --http-log-path=/dev/stdout --prefix=/tmp/local/nginx

https://stackoverflow.com/questions/22541333/have-nginx-access-log-and-error-log-log-to-stdout-and-stderr-of-master-process

https://stackoverflow.com/questions/51966915/how-to-use-an-environment-variable-as-prefix-path-while-configuring-nginx

```txt
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/tmp/local/nginx"
  nginx binary file: "/tmp/local/nginx/sbin/nginx"
  nginx modules path: "/tmp/local/nginx/modules"
  nginx configuration prefix: "/tmp/local/nginx/conf"
  nginx configuration file: "/tmp/local/nginx/conf/nginx.conf"
  nginx pid file: "/tmp/local/nginx/logs/nginx.pid"
  nginx logs errors to stderr
  nginx http access log file: "/dev/stdout"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

### apt-get install build-essential

```txt
bash: make: command not found
```

##

mkdir bin
mv objs/nginx bin/

##

daemon off;
listen 8080;

##

https://stackoverflow.com/questions/36249744/interactive-shell-using-docker-compose

Even I got the same issue. Its better to run docker-compose run <service_name> 