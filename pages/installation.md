---
title: Installation
description: Install Open Interop
permalink: /installation/
---

## Prerequisites

- PostgreSQL 10 and above (Earlier versions should be supported. However, testing is performed in 10)
- Rabbit MQ 3.8.3
- Install [rabbitmqadmin script](https://www.rabbitmq.com/management-cli.html)
- Node v12.16.*


## via Github

_Ensure you have read the [Prerequisites](/installation#Prerequisites) section._

Installing via Github requires you to download all the microservice repositories individually.

_This documentation site is a work in progress. Check back for updates._

## via Docker Hub

_Ensure you have read the [Prerequisites](/installation#Prerequisites) section._

The advantage of installing Open Interop via Docker is that it will run anywhere that you can run the Docker engine. This includes; Windows, Linux, and Mac OS. Containerisation is a valid method for both development and production systems. Here we will describe how to spin up a working and complete Open Interop backend with [docker-compose](https://docs.docker.com/compose/) for use in development and prototyping.

### Prerequisites

- [Install Docker](https://docs.docker.com/engine/install/)
- Python 3.x (to install `rabbitmqadmin` tool)

Both PostreSQL and Rabbit MQ could be run within a Docker container. However, we have found that most users already have database's configured on their machines and spinning up additional ones is not necessary.

### Images

All of the *released* Open Interop micro-services have been uploaded to Docker Hub as images. Each one will ultimately have a 'latest' tag, a 'dev' tag, and also a 'version-x.x.x' tag depending on the version.

These images can be used with docker-compose, Docker Swarm, and Kubernetes deployments. We will be releasing compatibility matrices for our 'docker' images so that you can determine what micro-service versions work with different micro-services.

### Using docker-compose

In order to use docker-compose, you need to create a configuration file. We have a sample docker-compose file which can be used to get yourself up and running. Please review [docker](https://github.com/open-interop/oop-docker).

Create a new folder called `oop-docker` and include a `docker-compose.yml` file. You will also require a `data` and `config` directory.

#### Example `docker-compose.yml` file

```
version: '3'

services:
  oop-core:
    image: openinterop/oop-core:version-1.1.5
    ports:
      - "9001:9001"
    env_file:
      - all.env
    volumes:
      - gem_cache:/gems
      - './data/logs:/app/logs'
      - './config/database.yml:/app/config/database.yml'
      - './config/storage.yml:/app/config/storage.yml'
      - './config/secrets.yml:/app/config/secrets.yml'
    restart: on-failure

  oop-core-transmissions:
    image: openinterop/oop-core:version-1.1.5
    env_file:
      - all.env
    volumes:
      - gem_cache:/gems
      - './data/logs:/app/logs'
      - './config/database.yml:/app/config/database.yml'
      - './config/storage.yml:/app/config/storage.yml'
      - './config/secrets.yml:/app/config/secrets.yml'

    command: bin/rails runner TransmissionQueue.retrieve_transmissions
    restart: on-failure

  oop-gateway:
    image: openinterop/oop-gateway:version-1.0.5
    ports:
      - "3000:3000"
    env_file:
      - all.env
    restart: on-failure

  oop-authenticator:
    image: openinterop/oop-authenticator:version-1.0.4
    env_file:
      - all.env
    depends_on:
      - oop-core
    restart: on-failure

  oop-tempr:
    image: openinterop/oop-tempr:version-1.0.6
    env_file:
      - all.env
    restart: on-failure

  oop-renderer:
    image: openinterop/oop-renderer:version-1.0.3
    env_file:
      - all.env
    restart: on-failure

  oop-endpoints-http:
    image: openinterop/oop-endpoints-http:version-1.0.4
    env_file:
      - all.env
    restart: on-failure

  oop-relay:
    image: openinterop/oop-relay:version-1.0.4
    env_file:
      - all.env
    restart: on-failure

volumes:
  gem_cache:
```

#### Configuration

You will notice from the above file that it requires a number of configuration files. You can find the contents of the `all.env` file on our config repository [oop-queue-config](https://github.com/open-interop/oop-queue-config) the file `.env.all.example` should give you a good starting point for what environment variables to define. In this .env file the field  `POSTGRES_PASSWORD` will currently be empty - enter the password you set when installing PostgreSQL here. 

##### RabbitMQ

Assuming that you have RabbitMQ install correctly (and added to PATH) and the `rabbitmqadmin` tool installed. You need to do two things, add yourself a user to rabbitmq and import the queue config.

###### Create a user

```
sudo rabbitmqctl add_user oop somepassword
sudo rabbitmqctl list_permissions --vhost /
sudo rabbitmqctl set_permissions -p / oop ".*" ".*" ".*"
```

Ensure this is reflected in `OOP_AMQP_ADDRESS` connection string environment variable.

###### Import the queue config

There is a pre-configured queue config in the [oop-queue-config](https://github.com/open-interop/oop-queue-config) repository. Download `rabbit-config.json` from the repo and run `sudo rabbitmqadmin import rabbit-config.json` to import it.

##### oop-core

`oop-core` is a [Ruby on Rails](https://rubyonrails.org) application and requires a number of configuration files for it to work.

Many of these can also be configured using environment variables:

- `config/database.yml`
- `config/storage.yml`
- `config/secrets.yml`

###### `database.yml`

```
# config/database.yml

default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: <%= ENV.fetch('DATABASE_HOST') %>
  username: <%= ENV.fetch('POSTGRES_USER') %>
  password: <%= ENV.fetch('POSTGRES_PASSWORD') %>
  database: <%= ENV.fetch('POSTGRES_DB') %>
  variables:
    statement_timeout: 5000

production:
  <<: *default
```

###### `storage.yml`

```
# config/storage.yml

local:
  service: Disk
  root: <%= Rails.root.join("storage") %>
```

###### `secrets.yml`

```
# config/secrets.yml

production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

These examples assume you use the development environment. If you're setting up an environment for production, please use the `production` keys rather than `development` in your yml files.


#### Using `docker-compose`

Calling the command `docker-compose up` will use the provided `Dockerfile` and pull down the necessary images.

#### Creating a test account

To create a test account you'll need to access the Docker container running `oop-core` and run some commands in there.

Run `docker exec -it oop-docker_oop-core_1 bash` to open a shell inside the `oop-core` container.
Now run `bin/rails console` followed by:
`account = Account.create!(name: 'Test account', host: 'localhost')`
`account.users.create!(email: "test@example.com", password: "testtest", password_confirmation: "testtest")`

#### Setting up the database

If you already have the database running then skip this step, if not...
Create the database by running `bundle exec rails db:create`
Then run the database migrations `bundle exec rails db:migrate`

You have now created an account which you can use to log in to the interface.


#### Installing the interface

You've now installed [Open Interop](https://openinterop.org). Providing you completed the previous step, Open Interop will be running in Docker and the API endpoints will now work - but to access the interface and see what's going on you'll need to [install the interface](https://github.com/{{ site.github_user}}/oop-core-interface).

## Support

If you need help, please don't hesitate to [open an issue](https://www.github.com/{{ site.github_user }}/{{ site.github_repo }}/issues/new).
