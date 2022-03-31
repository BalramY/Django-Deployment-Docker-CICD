# Django-Deployment-Docker-CICD


How to Dockerize Django Project?

Let's see-

1) Create a file and Name it Dockerfile
2) Paste the following Content
========Dockerfile==========
# syntax=docker/dockerfile:1
FROM python:3
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt

RUN apt-get update
RUN apt-get update && apt-get install -y python3.8-dev python3-pip build-essential libpq-dev postgis python3-gdal

COPY ./docker/entrypoint.sh /code/docker/entrypoint.sh


RUN pip3 install --upgrade pip setuptools wheel

RUN pip3 install psycopg2-binary --no-binary psycopg2-binary
RUN chmod +x /code/docker/entrypoint.sh

COPY . /code/

=========Dockerfile End================

Let me explain you the above file.

We have taken python 3 docker image (Just like we import any module in python similarly we import the python3 docker image.
We set 2 ENV variable

Then we make /code folder as working directory for docker
WORKDIR /code

We copy our requirements.txt file inside code folder in docker (Note: you should create a requirements.txt file if you didn't create yet.

Now just like we run any command in ubuntu so if we want to run same command in docker we start line by RUN like 
RUN pip install -r requirements.txt


RUN apt-get update
RUN apt-get update && apt-get install -y python3.8-dev python3-pip build-essential libpq-dev postgis python3-gdal

Next?

We will create a docker folder in project directory and insdie that we will create entrypoint.sh file
so we copy that file to inside docker
COPY ./docker/entrypoint.sh /code/docker/entrypoint.sh

We install setup tools inside docker
RUN pip3 install --upgrade pip setuptools wheel

Install postgres
RUN pip3 install psycopg2-binary --no-binary psycopg2-binary

given permission to entrypoint.sh
RUN chmod +x /code/docker/entrypoint.sh

#Copied the all files and folder inside /code folder in docker.
COPY . /code/

=============

3) Create a folder docker in project folder
4) create a file docker-compose.yaml
5) Paste the following content inside the docker-compose.yaml

=========docker-compose.yaml===================
# Setup docker compose for postgres, pgadmin, redis, celery
# it'd be nice if the postgres container could run the migrations when first initialised

version: '3.3'
services:
  redis:
    container_name: redis
    image: "redis:6.2-alpine"
    command: redis-server --requirepass $REDIS_PASSWORD
    ports:
      - "6379:6379"
    volumes:
      - "redis_data:/data"
    networks:
      - middleware-net

  postgres:
    container_name: postgres
    image: kartoza/postgis
    restart: always
    environment:
      POSTGRES_USER: $DB_USER
      POSTGRES_PASSWORD: $DB_PASSWORD
      POSTGRES_DB: $DB_NAME
    ports:
      - 5433:5433
    volumes:
      - "postgres_data:/var/lib/postgresql"
    networks:
      - middleware-net

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4
    restart: always
    environment:
      PGADMIN_LISTEN_PORT: 5050
      PGADMIN_DEFAULT_EMAIL: $SAFEBEAT_ADMIN_EMAIL
      PGADMIN_DEFAULT_PASSWORD: $SAFEBEAT_ADMIN_PASSWORD
    ports:
      - "5050:5050"
    networks:
      - middleware-net

  backend:
    container_name: backend
    restart: always
    build:
      context: '../'
    expose:
      - "8000"
    ports:
      - "8000:8000"
    links:
      - "redis:redis"
    volumes:
      - ../:/code
      - backend_static:/static
      - backend_media:/media
    depends_on:
      - postgres
    networks:
      - middleware-net
    env_file:
      - '../.env.development'
    environment:
      DEBUG: 'true'
    command: 'sh -c "/code/docker/entrypoint.sh && exec python3 manage.py runserver 0.0.0.0:8000"'

volumes:
  backend_static:
  backend_media:
  redis_data:
  postgres_data:
  # pg_data:

networks:
  middleware-net:
    driver: bridge
============================docker-compose end=============

We are using docker compose version 3.3

Now see the services in the docker compose -
if we want to run redis we can keep this service
redis:
    container_name: redis
    image: "redis:6.2-alpine"
    command: redis-server --requirepass $REDIS_PASSWORD
    ports:
      - "6379:6379"
    volumes:
      - "redis_data:/data"
    networks:
      - middleware-net

The redis will run on 6379 port, if we don't need we can remove it from here and links and volumes and other places where redis used.
