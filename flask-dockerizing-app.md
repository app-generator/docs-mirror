This page explains how to Dockerize a Flask application with ease.

For newcomers, Flask is a micro web framework built in Python and Docker is a virtualization software used to isolate the execution of a service or product using virtual containers.

To dockerize our Flask application, we have to make sure that the Flask application is working and explain the services we'll be using.

## Flask App

The codebase used for this tutorial is the [Boilerplate Code - Flask Dashboard | AppSeed](https://github.com/app-generator/boilerplate-code-flask-dashboard). 
We'll be going through the configuration of Docker for this application.

## Services used

Many services are used to make sure we have a powerful and stable version of dockerized Flask application.

### Nginx - a powerful HTTP proxy 

[NGINX](https://www.nginx.com/) is an open-source HTTP and reverse proxy server, HTTP cache, and load balancer.
You can see the configuration of this server [here](https://github.com/app-generator/boilerplate-code-flask-dashboard/blob/master/nginx/appseed-app.conf).

```
upstream webapp {
    server appseed_app:5005;
}

server {
    listen 85;
    server_name localhost;

    location / {
        proxy_pass http://webapp;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}
```

### Gunicorn - the WSGI server

[Gunicorn](https://gunicorn.org/) is a stable WSGI server built for Python. It powers web applications such as Instagram because it's commonly used for web app deployments.
The Python package also allows you to make some configurations such as the [workers](https://docs.gunicorn.org/en/stable/design.html) or the [server socket](https://docs.gunicorn.org/en/stable/settings.html#server-socket) for example. 
You can see a configuration for this [here](https://github.com/app-generator/boilerplate-code-flask-dashboard/blob/master/gunicorn-cfg.py).

```python
# gunicorn-cfg.py

bind = '0.0.0.0:5005'
workers = 1
accesslog = '-'
loglevel = 'debug'
capture_output = True
enable_stdio_inheritance = True
```

## Adding Docker

First of all, make sure you have Docker installed and running on your machine. Refer to the official [documentation](https://docs.docker.com/engine/install/) and follow the steps.
Once it's done, create a `Dockerfile` file. This will contains all the commands that could be called on the command line to create an image.

```
FROM python:3.9

COPY . .

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install python dependencies
RUN pip install --upgrade pip
RUN pip install --no-cache-dir -r requirements.txt

# gunicorn
CMD ["gunicorn", "--config", "gunicorn-cfg.py", "run:app"]
```

Once it's done, you can build your image.

```
$ docker build -t flask_app .
$ docker run -p 5000:5000 flask_app
```

You can now access your app on http://localhost:5000


## Docker Compose

If you noticed, we are not running the NGINX service. It can be done by writing another `Dockerfile` but then we'll need again to build a new image.

Docker Compose saves from this by allowing you to use a `YAML` file to operate multi-container applications at once and run it just with one command. 

Follow the official [documentation](https://docs.docker.com/compose/install/) to install docker-compose on your machine.

Once it's done, create a `docker-compose.yml` file at the root of your project. Make sure to have an `.env` file at the root of your project.

```yml
version: '3.8'
services:
  appseed-app:
    container_name: appseed_app
    restart: always
    env_file: .env
    build: .
    networks:
      - db_network
      - web_network
  nginx:
    container_name: nginx
    restart: always
    image: "nginx:latest"
    ports:
      - "85:85"
    volumes:
      - ./nginx:/etc/nginx/conf.d
    networks:
      - web_network
    depends_on: 
      - appseed-app
networks:
  db_network:
    driver: bridge
  web_network:
    driver: bridge
```

And now let's use `docker-compose` command to build and run the image.

```shell
docker-compose up --build
```

Then visit your application at https://localhost:85
