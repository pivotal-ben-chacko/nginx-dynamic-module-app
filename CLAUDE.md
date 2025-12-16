# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an NGINX application with a dynamically loaded module (`headers-more-nginx-module`) that allows modification of HTTP headers. The application is designed to be deployed using Cloud Foundry or containerized with Paketo buildpacks.

## Architecture

### Core Components

- **nginx.conf**: Main NGINX configuration that:
  - Loads the `ngx_http_headers_more_filter_module.so` dynamic module
  - Uses template variable `{{port}}` for port binding (replaced at runtime by buildpack)
  - Serves static content from the `public/` directory
  - Uses the headers-more module to override the Server header to "Fiserv"

- **modules/**: Contains the pre-compiled dynamic NGINX module
  - `ngx_http_headers_more_filter_module.so`: Compiled for x86-64 Linux against NGINX 1.29.0
  - This module MUST be compiled against the exact NGINX version used at runtime

- **public/**: Static web content served by NGINX
  - `index.html`: Simple welcome page

### Deployment Architecture

The application supports two deployment methods:

1. **Cloud Foundry** (Primary):
   - Uses the `nginx-buildpack` which processes `nginx.conf` template variables
   - Deploy command: `cf push nginx-test -b nginx-buildpack`

2. **Docker with Paketo Buildpacks** (CI/CD):
   - GitHub Actions workflow builds and pushes to Docker Hub
   - Uses `paketobuildpacks/builder:base`
   - Automatically detects NGINX configuration

## Module Compilation

If you need to recompile the dynamic module:

1. The module MUST be compiled against the exact NGINX version (currently 1.29.0)
2. Compilation should be done on Ubuntu for x86-64 compatibility
3. Reference: https://www.nginx.com/blog/compiling-dynamic-modules-nginx-plus/

```bash
# Download NGINX source
wget 'http://nginx.org/download/nginx-1.29.0.tar.gz'
tar -xzvf nginx-1.29.0.tar.gz

# Download headers-more module
wget https://github.com/openresty/headers-more-nginx-module/archive/refs/tags/v0.39.zip
unzip v0.39.zip

# Install dependencies
sudo apt install libpcre2-dev

# Compile
cd nginx-1.29.0
./configure --prefix=/opt/nginx \
    --add-dynamic-module=/<PATH-TO-HEADERS-MORE-MODULE>/headers-more-nginx-module-0.39
make
make install
```

## Important Constraints

- Dynamic modules are tightly coupled to specific NGINX versions - version mismatches will cause runtime failures
- The `{{port}}` template variable in nginx.conf is replaced by the buildpack at deployment time
- The Server header modification demonstrates the headers-more module functionality
