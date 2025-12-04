
# nginx-dynamic-module-app

Instructions: https://www.nginx.com/blog/compiling-dynamic-modules-nginx-plus/

Used nginx version: 1.25.5
(dynamic modules need to be compiled against the exact version)

 ```bash 
 wget 'http://nginx.org/download/nginx-1.29.0.tar.gz'
 tar -xzvf nginx-1.29.0.tar.gz
 cd nginx-1.29.0/
 ```

```bash
wget https://github.com/openresty/headers-more-nginx-module/archive/refs/tags/v0.39.zip
unzip v0.39.zip
cd nginx-1.29.0

# Install pcre2 if required
sudo apt install libpcre2-dev

# Here we assume you would install you nginx under /opt/nginx/.
 ./configure --prefix=/opt/nginx \
     --add-dynamic-module=/path/to/headers-more-nginx-module

 make
 make install
```

```
cf push testapp -b https://github.com/cloudfoundry/nginx-buildpack#v1.2.5
```
