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

### Prerequisites for Github

- yarn 1.22.x (npm install -g yarn)
- Go [1.15.x](https://golang.org/doc/install)
- Ruby [2.6.6](https://www.ruby-lang.org/en/downloads/) (with [bundler](https://bundler.io/) installed)


Installing via Github requires you to download all the microservice repositories individually. Including the interface, there are ten services you'll need to install and run - this will take some time, and installing through Docker is recommended.

### .env

A file including all environment variables will need to be created outside of any of the microservice directories. A symlink will need to be created in each directory to mirror .env, the command for this will be included in the instructions for each microservice individually.

##### `.env.example`
```
NODE_ENV=production
RAILS_ENV=production

# All repos
OOP_AMQP_ADDRESS=amqp://oop:somepasssword@localhost
RABBITMQ_URL=amqp://oop:somepassword@localhost
OOP_ERROR_EXCHANGE_NAME=oop.errors
OOP_JSON_ERROR_Q=oop.errors.json
OOP_EXCHANGE_NAME=oop
OOP_QUEUE_PREFETCH_LIMIT=5

# Gateway
OOP_GATEWAY_OUTPUT_Q=oop.noauth.raw_messages
OOP_LISTEN_PORT=3000
OOP_DB_ADDRESS=http://admin:admin@localhost:5984

# Authenticator
OOP_AUTHENTICATOR_INPUT_Q=oop.noauth.raw_messages
OOP_AUTHENTICATOR_OUTPUT_Q=oop.hasauth.messages
OOP_CORE_TOKEN=foobar
OOP_CORE_DEVICE_UPDATE_EXCHANGE=oop.core.devices

# Tempr
OOP_TEMPR_INPUT_Q=oop.hasauth.messages
OOP_TEMPR_OUTPUT_Q=oop.hasauth.temprs
OOP_CORE_API_URL=http://{CORE_IP_ADDRESS}:9001/services/v1

# Scheduler
OOP_SCHEDULER_OUTPUT_Q=oop.hasauth.messages

# Renderer
OOP_RENDERER_INPUT_Q=oop.hasauth.temprs
OOP_ENDPOINTS_EXCHANGE_NAME=oop.endpoints
OOP_ENDPOINT_Q=oop.endpoints

# Endpoints HTTP
OOP_ENDPOINTS_HTTP_OUTPUT_Q=oop.hasauth.relay
OOP_REQUEST_TIMEOUT=1000
OOP_ENDPOINTS_HTTP_MAX_RETRIES=3

# Relay
OOP_RELAY_INPUT_Q=oop.hasauth.relay
OOP_RECURSIVE_TEMPR_Q=oop.hasauth.temprs
OOP_CORE_RESPONSE_Q=oop.core.transmissions

# Core
OOP_AMQP_ADDRESS=amqp://localhost
OOP_RENDERER_PATH=/path/to/renderer/

OOP_CORE_SCHEME=http://
OOP_CORE_PORT=9001
OOP_CORE_PATH=/

PORT=9001

OOP_CORE_INTERFACE_SCHEME=http://
OOP_CORE_INTERFACE_PORT=80
OOP_CORE_INTERFACE_PATH=/
OOP_CORE_FROM_ADDRESS=example@mail.co.uk

# Mail setup
OOP_CORE_FROM_ADDRESS=
OOP_CORE_SMTP_ADDRESS=
OOP_CORE_SMTP_PORT=25
OOP_CORE_SMTP_DOMAIN=
OOP_CORE_SMTP_USER_NAME=
OOP_CORE_SMTP_PASSWORD=
OOP_CORE_SMTP_AUTHENTICATION=
OOP_CORE_SMTP_ENABLE_STARTTLS_AUTO=

DATABASE_HOST=localhost
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DB=oop_core_development

SECRET_KEY_BASE={SECRETKEY}
```

### oop-core

Clone the application `git clone http://github.com/open-interop/oop-core.git`
In the app directory install all dependencies using  `bundle install`
Create a symlink between
Ensure all variables are set up correctly in the config directory:

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


Create the database - `bundle exec rails db:create`
Run the database migrations - `bundle exec rails db:migrate`


To allow login from the interface run the following commands from within the `oop-core` directory to create a dummy user:
```
account = Account.create!(name: 'Test account', host: 'localhost')

account.users.create!(email: "test@example.com", password: "testtest", password_confirmation: "testtest")
```

To start core run - `bin/rails server -b HOSTNAME -p 9001` where HOSTNAME is the host core is accessible from - this should be reflected in the `.env` file above

`oop-core` includes the transmission queue, to start this run - `bin/rails runner TransmissionQueue.retrieve_transmissions`

### Microservices:

The following services can all be installed and set up in exactly the same way, cloning the repo and using yarn to install any dependencies. 

#### oop-gateway

`git clone http://github.com/open-interop/oop-gateway.git`

`cd oop-gateway`

`ln ../.env`

`yarn install`

When all microservices are installed and core is up, run `yarn start`

#### oop-tempr

`git clone http://github.com/open-interop/oop-tempr.git`

`cd oop-tempr`

`ln ../.env`

`yarn install`

When all microservices are installed and core is up, run `yarn start`

#### oop-renderer

`git clone http://github.com/open-interop/oop-renderer.git`

`cd oop-renderer`

`ln ../.env`

`yarn install`

When all microservices are installed and core is up, run `yarn start`

#### oop-authenticator

`git clone http://github.com/open-interop/oop-authenticator.git`

`cd oop-authenticator`

`ln ../.env`

`yarn install`

When all microservices are installed and core is up, run `yarn start`

#### oop-relay

`git clone http://github.com/open-interop/oop-relay.git`

`cd oop-relay`

`ln ../.env`

`yarn install`

When all microservices are installed and core is up, run `yarn start`

#### oop-endpoints-http

`git clone http://github.com/open-interop/oop-endpoints-http.git`

`cd oop-endpoints-http`

`ln ../.env`

`yarn install`

When all microservices are installed and core is up, run `yarn start`

### oop-blacklist

Blacklist has been built slightly differently to those above, and will be build and run using Go.

Still clone the repo and move into it:

`git clone http://github.com/open-interop/oop-blacklist.git`

`cd oop-blacklist`

The blacklist will need it's own .env file, so don't create a symlink to the original .env. Create a .env file as below:

##### .env
```
OOP_CORE_API_URL=http://CORE-IP-ADDRESS:PORT
OOP_CORE_TOKEN=foobar
OOP_BLACKLIST_SOCKET_ADDRESS=http://CORE-IP-ADDRESS
OOP_CORE_BLACKLIST_ENTRY_UPDATE_EXCHANGE=oop.core.blacklist_entries
```

To build the blacklist, run `go build` and once built, run `go run cmd/oop-blacklist/main.go` to start blacklist running.

### oop-core-interface

The interface should be installed last, once core and all microservices are installed and running, and a dummy user has been created.

Clone the repo - `git clone http://github.com/open-interop/oop-core-interface.git`

`cd oop-core-interface`

Install all dependencies - `yarn install`

Create a file .env:

##### .env
```
# For development
PROXY=http://CORE-IP-ADDRESS:PORT
```

Once core and all microservices are running, use `yarn run start` to launch the interface, this will open up a window in your browser to access the interface.

## via Docker Hub

_Ensure you have read the [Prerequisites](/installation#Prerequisites) section._

The advantage of installing Open Interop via Docker is that it will run anywhere that you can run the Docker engine. This includes; Windows, Linux, and Mac OS. Containerisation is a valid method for both development and production systems. Here we will describe how to spin up a working and complete Open Interop backend with [docker-compose](https://docs.docker.com/compose/) for use in development and prototyping.

### Prerequisites for Docker

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
