# Deployment

## Vue JS Deployment with Docker

    ref:  https://cli.vuejs.org/guide/deployment.html#docker-nginx

### Docker (Nginx)

Deploy your application using nginx inside of a docker container.

1. Install [docker](https://www.docker.com/get-started)

2. Create a `Dockerfile` file in the root of your project.

    ```docker
    FROM node:latest as build-stage
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY ./ .
    RUN npm run build

    FROM nginx as production-stage
    RUN mkdir /app
    COPY --from=build-stage /app/dist /app
    COPY nginx.conf /etc/nginx/nginx.conf
    ```

3. Create a `.dockerignore` file in the root of your project

    Setting up the `.dockerignore` file prevents `node_modules` and any intermediate build artifacts from being copied to the image which can cause issues during building.

    ```
    **/node_modules
    **/dist
    ```

4. Create a `nginx.conf` file in the root of your project

    `Nginx` is an HTTP(s) server that will run in your docker container. It uses a configuration file to determine how to serve content/which ports to listen on/etc. See the [nginx configuration documentation](https://www.nginx.com/resources/wiki/start/topics/examples/full/) for an example of all of the possible configuration options.

    The following is a simple `nginx` configuration that serves your vue project on port `80`. The root `index.html` is served for `page not found` / `404` errors which allows us to use `pushState()` based routing.

    ```nginx
    user  nginx;
    worker_processes  1;
    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;
    events {
      worker_connections  1024;
    }
    http {
      include       /etc/nginx/mime.types;
      default_type  application/octet-stream;
      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';
      access_log  /var/log/nginx/access.log  main;
      sendfile        on;
      keepalive_timeout  65;
      server {
        listen       80;
        server_name  localhost;
        location / {
          root   /app;
          index  index.html;
          try_files $uri $uri/ /index.html;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
          root   /usr/share/nginx/html;
        }
      }
    }
    ```

5. Build your docker image

    ```bash
    docker build . -t my-app
    # Sending build context to Docker daemon  884.7kB
    # ...
    # Successfully built 4b00e5ee82ae
    # Successfully tagged my-app:latest
    ```

6. Run your docker image

    This build is based on the official `nginx` image so log redirection has already been set up and self daemonizing has been turned off. Some other default settings have been setup to improve running nginx in a docker container. See the [nginx docker repo](https://hub.docker.com/_/nginx) for more info.

    ```bash
    docker run -d -p 8080:80 my-app
    curl localhost:8080
    # <!DOCTYPE html><html lang=en>...</html>
    ```
