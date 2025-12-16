
# nginx-dynamic-module-app

Instructions: https://www.nginx.com/blog/compiling-dynamic-modules-nginx-plus/

Used nginx version: 1.29.0
(dynamic modules need to be compiled against the exact version)

Download and extract NGINX source.

 ```bash 
 wget 'http://nginx.org/download/nginx-1.29.0.tar.gz'
 tar -xzvf nginx-1.29.0.tar.gz
 ```

Download and extract headers-more module.

```bash
wget https://github.com/openresty/headers-more-nginx-module/archive/refs/tags/v0.39.zip
unzip v0.39.zip
```

Compile module. (Compiled on Ubuntu so that the file is compatible with x86 architecture)

```
# Install pcre2 if required
sudo apt install libpcre2-dev

cd nginx-1.29.0
# Here we assume you would install you nginx under /opt/nginx/.
 ./configure --prefix=/opt/nginx \
     --add-dynamic-module=/<PATH-TO-HEADERS-MORE-MODULE>/headers-more-nginx-module-0.39

 make
 make install
```

Include module in Application

1. Create a modules directory in the root of your projects directory.
2. Copy the compiled module from /opt/nginx to ./modules directory.
3. From the root of your project directory, do a cf push to deploy app.

```
cf push nginx-test -b nginx-buildpack
```

## GitHub Actions CI/CD

This repository includes an automated build and deployment pipeline using GitHub Actions and Paketo Buildpacks.

### Workflow

The workflow (`.github/workflows/build-and-push.yml`) automatically:
- Triggers on pushes to the `main` branch
- Uses Paketo Buildpacks to build a Docker image
- Publishes the image to Docker Hub

### Setup Requirements

To use the GitHub Actions workflow, configure the following secrets in your repository settings:

- `DOCKERHUB_USERNAME`: Your Docker Hub username
- `DOCKERHUB_TOKEN`: Your Docker Hub access token

### Build Command

The workflow uses the `pack` CLI with Paketo buildpacks:

```bash
pack build index.docker.io/<username>/vmware:latest \
  --builder paketobuildpacks/builder:base \
  --publish
```

The Paketo buildpack automatically detects the NGINX configuration and creates a containerized application.
