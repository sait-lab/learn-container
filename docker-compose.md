## Docker Compose



### Table of Contents

   * [Docker Compose](#docker-compose)
      * [Table of Contents](#table-of-contents)
      * [What is Docker Compose](#what-is-docker-compose)
         * [Docker Compose Overview](#docker-compose-overview)
         * [Why Use Docker Compose](#why-use-docker-compose)
         * [Common Use Cases of Docker Compose](#common-use-cases-of-docker-compose)
            * [Development environments](#development-environments)
            * [Automated Testing Environments](#automated-testing-environments)
            * [Single Host Deployments](#single-host-deployments)
      * [How Compose works](#how-compose-works)
         * [The Compose File](#the-compose-file)
      * [Install Docker Compose](#install-docker-compose)
      * [Docker Compose Demo](#docker-compose-demo)
         * [Project Structure](#project-structure)
         * [compose.yaml](#composeyaml)
         * [Deploy with docker compose](#deploy-with-docker-compose)
         * [Expected Result](#expected-result)
         * [Testing the App](#testing-the-app)
         * [Stop and Remove the Containers](#stop-and-remove-the-containers)
      * [Compose Files](#compose-files)
         * [Name top-level element](#name-top-level-element)
         * [Services top-level elements](#services-top-level-elements)
            * [Attributes](#attributes)
               * [container_name](#container_name)
               * [depends_on](#depends_on)
               * [deploy](#deploy)
               * [dns](#dns)
               * [entrypoint](#entrypoint)
               * [env_file](#env_file)
               * [environment](#environment)
               * [expose](#expose)
               * [healthcheck](#healthcheck)
               * [image](#image)
               * [links](#links)
               * [ports](#ports)
               * [runtime](#runtime)
               * [volumes](#volumes)
      * [Run Docker Compose services with GPU access](#run-docker-compose-services-with-gpu-access)



---



### What is Docker Compose

#### Docker Compose Overview

When Docker was new, a company called Orchard Labs built a tool called [Fig](https://orchardup.github.io/fig.sh/) that made deploying and managing multi-container apps easy. It was a Python tool that ran on top of Docker and let you define complex multi-container microservices apps in a simple YAML file. You could even use the fig command-line tool to manage the entire application lifecycle.

In the background, Fig processed the YAML file and executed the necessary Docker commands to handle the deployment and management of the app.

Fig was so good that [Docker, Inc. acquired Orchard Labs](https://devops.com/docker-acquires-orchard/) and rebranded Fig as Docker Compose. They renamed the command-line tool from fig to `docker-compose`, and then, more recently, they folded it into the main docker CLI with its own compose sub-command. You can now run simple `docker compose` commands to easily manage multi-container microservices apps.

Excerpt from [Docker Compose overview | Docker Docs](https://docs.docker.com/compose/)

> Docker Compose is a tool for defining and running multi-container applications. It is the key to unlocking a streamlined and efficient development and deployment experience.
>
> Compose simplifies the control of your entire application stack, making it easy to manage services, networks, and volumes in a single, comprehensible YAML configuration file. Then, with a single command, you create and start all the services from your configuration file.
>
> Compose works in all environments; production, staging, development, testing, as well as CI workflows. It also has commands for managing the whole lifecycle of your application:
>
> - Start, stop, and rebuild services
> - View the status of running services
> - Stream the log output of running services
> - Run a one-off command on a service

#### Why Use Docker Compose

Excerpt from https://docs.docker.com/compose/intro/features-uses/

>Using Docker Compose offers several benefits that streamline the development, deployment, and management of containerized applications:
>
>- Simplified control: Docker Compose allows you to define and manage multi-container applications in a single YAML file. This simplifies the complex task of orchestrating and coordinating various services, making it easier to manage and replicate your application environment.
>- Efficient collaboration: Docker Compose configuration files are easy to share, facilitating collaboration among developers, operations teams, and other stakeholders. This collaborative approach leads to smoother workflows, faster issue resolution, and increased overall efficiency.
>- Rapid application development: Compose caches the configuration used to create a container. When you restart a service that has not changed, Compose re-uses the existing containers. Re-using containers means that you can make changes to your environment very quickly.
>- Portability across environments: Compose supports variables in the Compose file. You can use these variables to customize your composition for different environments, or different users.
>- Extensive community and support: Docker Compose benefits from a vibrant and active community, which means abundant resources, tutorials, and support. This community-driven ecosystem contributes to the continuous improvement of Docker Compose and helps users troubleshoot issues effectively.

#### Common Use Cases of Docker Compose

##### Development environments

>When you're developing software, the ability to run an application in an isolated environment and interact with it is crucial. The Compose command line tool can be used to create the environment and interact with it.
>
>The [Compose file](https://docs.docker.com/compose/compose-file/) provides a way to document and configure all of the application's service dependencies (databases, queues, caches, web service APIs, etc). Using the Compose command line tool you can create and start one or more containers for each dependency with a single command (`docker compose up`).
>
>Together, these features provide a convenient way for you to get started on a project. Compose can reduce a multi-page "developer getting started guide" to a single machine-readable Compose file and a few commands.

##### Automated Testing Environments

> An important part of any Continuous Deployment or Continuous Integration process is the automated test suite. Automated end-to-end testing requires an environment in which to run tests. Compose provides a convenient way to create and destroy isolated testing environments for your test suite. By defining the full environment in a [Compose file](https://docs.docker.com/compose/compose-file/), you can create and destroy these environments in just a few commands:
>
> ```
> docker compose up -d
> ./run_tests
> docker compose down
> ```

##### Single Host Deployments

> Compose has traditionally been focused on development and testing workflows, but with each release we're making progress on more production-oriented features.



---



### How Compose works

Excerpt from https://docs.docker.com/compose/compose-application-model/

>Docker Compose relies on a YAML configuration file, usually named `compose.yaml`.
>
>The `compose.yaml` file follows the rules provided by the [Compose Specification](https://docs.docker.com/compose/compose-file/) in how to define multi-container applications. This is the Docker Compose implementation of the formal [Compose Specification](https://github.com/compose-spec/compose-spec).

#### The Compose File

> The default path for a Compose file is `compose.yaml` (preferred) or `compose.yml` that is placed in the working directory. Compose also supports `docker-compose.yaml` and `docker-compose.yml` for backwards compatibility of earlier versions. If both files exist, Compose prefers the canonical `compose.yaml`.



---



### Install Docker Compose

https://docs.docker.com/compose/install/linux/ 

https://docs.docker.com/compose/install/standalone/



---



### Docker Compose Demo

https://docs.docker.com/compose/gettingstarted/

Download example code from [Sample Node.js application with Nginx proxy and a Redis database.](https://github.com/docker/awesome-compose/tree/master/nginx-nodejs-redis)

#### Project Structure

```
.
├── README.md
├── compose.yaml
├── nginx
│   ├── Dockerfile
│   └── nginx.conf
└── web
    ├── Dockerfile
    ├── package.json
    └── server.js

2 directories, 7 files
```

#### compose.yaml

```yaml

services:
  redis:
    image: 'redislabs/redismod'
    ports:
      - '6379:6379'
  web1:
    restart: on-failure
    build: ./web
    hostname: web1
    ports:
      - '81:5000'
  web2:
    restart: on-failure
    build: ./web
    hostname: web2
    ports:
      - '82:5000'
  nginx:
    build: ./nginx
    ports:
    - '80:80'
    depends_on:
    - web1
    - web2
```

The compose file defines an application with four services `redis`, `nginx`, `web1` and `web2`. When deploying the application, docker compose maps port 80 of the nginx service container to port 80 of the host as specified in the file.

Run the following commands to download the project to `~/docker-compose-demo`

```
mkdir -p ~/docker-compose-demo
cd ~/docker-compose-demo

wget https://raw.githubusercontent.com/docker/awesome-compose/master/nginx-nodejs-redis/compose.yaml

mkdir -p nginx web

wget https://raw.githubusercontent.com/docker/awesome-compose/master/nginx-nodejs-redis/nginx/Dockerfile -O nginx/Dockerfile
wget https://raw.githubusercontent.com/docker/awesome-compose/master/nginx-nodejs-redis/nginx/nginx.conf -O nginx/nginx.conf

echo 'node_modules' > web/.gitignore
wget https://raw.githubusercontent.com/docker/awesome-compose/master/nginx-nodejs-redis/web/Dockerfile -O web/Dockerfile
wget https://raw.githubusercontent.com/docker/awesome-compose/master/nginx-nodejs-redis/web/package-lock.json -O web/package-lock.json
wget https://raw.githubusercontent.com/docker/awesome-compose/master/nginx-nodejs-redis/web/package.json -O web/package.json
wget https://raw.githubusercontent.com/docker/awesome-compose/master/nginx-nodejs-redis/web/server.js -O web/server.js
```

#### Deploy with docker compose

`docker compose up` builds, (re)creates, starts, and attaches to containers for a service. `-d` runs containers in the background.

```shell
docker compose up -d
```

The output is similar to this:

```
[+] Running 24/24
 ✔ redis Pulled                                                                                                 28.8s
   ✔ 1fe172e4850f Pull complete                                                                                  3.7s
   ✔ b37ded7c5478 Pull complete                                                                                  3.7s
   ✔ 731f72c7a1a0 Pull complete                                                                                  3.8s
   ✔ 34cfdf5ff0b4 Pull complete                                                                                  3.8s
   ✔ f34dd688846e Pull complete                                                                                  5.1s
   ✔ 6c5b270e591b Pull complete                                                                                  5.1s
   ✔ 85e4511d5c0c Pull complete                                                                                  6.6s
   ✔ 8ae042d0aa02 Pull complete                                                                                  6.6s
   ✔ 8f36acf34341 Pull complete                                                                                  6.6s
   ✔ 4f4fb700ef54 Pull complete                                                                                  7.0s
   ✔ 6de9a4176384 Pull complete                                                                                  7.0s
   ✔ d3e57aaad013 Pull complete                                                                                  7.7s
   ✔ 8eb23391b30a Pull complete                                                                                  9.4s
   ✔ 93afd04540e1 Pull complete                                                                                  9.4s
   ✔ fbab70c08f99 Pull complete                                                                                  9.4s
   ✔ d5a580d6eb66 Pull complete                                                                                 19.0s
   ✔ a78faad883d5 Pull complete                                                                                 19.0s
   ✔ 414b4d29d83b Pull complete                                                                                 19.3s
   ✔ 41b72fc2f30c Pull complete                                                                                 19.3s
   ✔ ae9c2ce92fb1 Pull complete                                                                                 19.4s
   ✔ aeb01ec80daf Pull complete                                                                                 19.4s
   ✔ b90ccc8ea95b Pull complete                                                                                 19.4s
   ✔ 0e6204cd32ab Pull complete                                                                                 27.8s
[+] Building 14.5s (24/27)                                                                             docker:default
 => [web1 internal] load build definition from Dockerfile                                                        0.0s
 => => transferring dockerfile: 182B                                                                             0.0s
 => [web2 internal] load build definition from Dockerfile                                                        0.0s
 => => transferring dockerfile: 182B                                                                             0.0s
 => [web1 internal] load metadata for docker.io/library/node:14.17.3-alpine3.14                                  1.2s
 => [web1 auth] library/node:pull token for registry-1.docker.io                                                 0.0s
 => [web1 internal] load .dockerignore                                                                           0.0s
 => => transferring context: 2B                                                                                  0.0s
 => [web2 internal] load .dockerignore                                                                           0.0s
 => => transferring context: 2B                                                                                  0.0s
 => [web2 1/5] FROM docker.io/library/node:14.17.3-alpine3.14@sha256:2fbaecb39357dec8af7afacaba214bba1ecb1da55b  4.3s
 => => resolve docker.io/library/node:14.17.3-alpine3.14@sha256:2fbaecb39357dec8af7afacaba214bba1ecb1da55ba8163  0.0s
 => => sha256:2fbaecb39357dec8af7afacaba214bba1ecb1da55ba8163f6afe531534787f80 1.22kB / 1.22kB                   0.0s
 => => sha256:289a5e686ccfd79d2cf6e3cded8a5baae90eeb655e46c9c9213ac9dcd67ce3a0 1.16kB / 1.16kB                   0.0s
 => => sha256:4c6ef524cd6d471188cc0f1caea69b7ecca4b8c7f04e731587a0e332929c46ee 6.53kB / 6.53kB                   0.0s
 => => sha256:5843afab387455b37944e709ee8c78d7520df80f8d01cf7f861aae63beeddb6b 2.81MB / 2.81MB                   0.8s
 => => sha256:a37d1565f85984511b93184a508aa421957073350176058f625546b42c2f2513 36.35MB / 36.35MB                 1.5s
 => => sha256:55161235e29f33c7780cd469b11203f5737c26738178dab68aea61e64f89f26b 2.36MB / 2.36MB                   0.6s
 => => sha256:b62f94589dc39388edce717712642533f1734bca9d1c5ffb0f88baa42bb9f9f3 281B / 281B                       0.7s
 => => extracting sha256:5843afab387455b37944e709ee8c78d7520df80f8d01cf7f861aae63beeddb6b                        0.1s
 => => extracting sha256:a37d1565f85984511b93184a508aa421957073350176058f625546b42c2f2513                        2.5s
 => => extracting sha256:55161235e29f33c7780cd469b11203f5737c26738178dab68aea61e64f89f26b                        0.1s
 => => extracting sha256:b62f94589dc39388edce717712642533f1734bca9d1c5ffb0f88baa42bb9f9f3                        0.0s
 => [web1 internal] load build context                                                                           0.0s
 => => transferring context: 16.86kB                                                                             0.0s
 => [web2 internal] load build context                                                                           0.0s
 => => transferring context: 16.86kB                                                                             0.0s
 => [web2 2/5] WORKDIR /usr/src/app                                                                              0.6s
 => [web1 3/5] COPY package.json package-lock.json ./                                                            0.0s
 => [web1 4/5] RUN npm ci                                                                                        2.2s
 => [web1 5/5] COPY ./server.js ./                                                                               0.0s
 => [web1] exporting to image                                                                                    0.1s
 => => exporting layers                                                                                          0.1s
 => => writing image sha256:9d2ff731bf4a4d226813bfa625b24c18af2c4cf07ba29cd067c197aea3c7d0fe                     0.0s
 => => naming to docker.io/library/docker-compose-demo-web1                                                      0.0s
 => [web2] exporting to image                                                                                    0.1s
 => => exporting layers                                                                                          0.1s
 => => writing image sha256:0de7da9e68e34a626c2c8db127387611031dfd4e88ea3b33513ba7325860743f                     0.0s
 => => naming to docker.io/library/docker-compose-demo-web2                                                      0.0s
 => [nginx internal] load build definition from Dockerfile                                                       0.0s
 => => transferring dockerfile: 140B                                                                             0.0s
 => [nginx internal] load metadata for docker.io/library/nginx:1.21.6                                            1.0s
 => [nginx auth] library/nginx:pull token for registry-1.docker.io                                               0.0s
 => [nginx internal] load .dockerignore                                                                          0.0s
 => => transferring context: 2B                                                                                  0.0s
 => [nginx 1/3] FROM docker.io/library/nginx:1.21.6@sha256:2bcabc23b45489fb0885d69a06ba1d648aeda973fae7bb981baf  4.4s
 => => resolve docker.io/library/nginx:1.21.6@sha256:2bcabc23b45489fb0885d69a06ba1d648aeda973fae7bb981bafbb8841  0.0s
 => => sha256:25dedae0aceb6b4fe5837a0acbacc6580453717f126a095aa05a3c6fcea14dd4 1.57kB / 1.57kB                   0.0s
 => => sha256:42c077c10790d51b6f75c4eb895cbd4da37558f7215b39cbf64c46b288f89bda 31.38MB / 31.38MB                 1.3s
 => => sha256:2bcabc23b45489fb0885d69a06ba1d648aeda973fae7bb981bafbb884165e514 1.86kB / 1.86kB                   0.0s
 => => sha256:0e901e68141fd02f237cf63eb842529f8a9500636a9419e3cf4fb986b8fe3d5d 7.66kB / 7.66kB                   0.0s
 => => sha256:62c70f376f6a97b1b1f970100583b01740ee4d0f1305226880d7f1624e425b9b 25.35MB / 25.35MB                 1.7s
 => => sha256:915cc9bd79c2262c322fb536ab56f19e551e71044aa2f80ab964cb15ea5e3ed4 601B / 601B                       0.6s
 => => sha256:75a963e94de04fe56dda9d3e3235bddbb34ea47d8f426acebf260ac24ef91f81 893B / 893B                       0.8s
 => => sha256:7b1fab684d70a138987d1539434eaa1d46f5e1b07cc8ee363cb31d251e048187 667B / 667B                       1.4s
 => => extracting sha256:42c077c10790d51b6f75c4eb895cbd4da37558f7215b39cbf64c46b288f89bda                        2.1s
 => => sha256:db24d06d5af41a56ab5e579ad26c71b7c0e35c6b11fd36015cb5e98df881d025 1.40kB / 1.40kB                   1.5s
 => => extracting sha256:62c70f376f6a97b1b1f970100583b01740ee4d0f1305226880d7f1624e425b9b                        0.8s
 => => extracting sha256:915cc9bd79c2262c322fb536ab56f19e551e71044aa2f80ab964cb15ea5e3ed4                        0.0s
 => => extracting sha256:75a963e94de04fe56dda9d3e3235bddbb34ea47d8f426acebf260ac24ef91f81                        0.0s
 => => extracting sha256:7b1fab684d70a138987d1539434eaa1d46f5e1b07cc8ee363cb31d251e048187                        0.0s
 => => extracting sha256:db24d06d5af41a56ab5e579ad26c71b7c0e35c6b11fd36015cb5e98df881d025                        0.0s
 => [nginx internal] load build context                                                                          0.0s
 => => transferring context: 210B                                                                                0.0s
 => [nginx 2/3] RUN rm /etc/nginx/conf.d/default.conf                                                            0.4s
 => [nginx 3/3] COPY nginx.conf /etc/nginx/conf.d/default.conf                                                   0.0s
 => [nginx] exporting to image                                                                                   0.0s
 => => exporting layers                                                                                          0.0s
 => => writing image sha256:bf47ef3599520cef52438da54458ac1fdebd3ce5f2527dcd90465d337939440f                     0.0s
 => => naming to docker.io/library/docker-compose-demo-nginx                                                     0.0s
[+] Running 5/5
 ✔ Network docker-compose-demo_default    Created                                                                0.1s
 ✔ Container docker-compose-demo-web2-1   Started                                                                0.5s
 ✔ Container docker-compose-demo-redis-1  Started                                                                0.5s
 ✔ Container docker-compose-demo-web1-1   Started                                                                0.5s
 ✔ Container docker-compose-demo-nginx-1  Started                                                                0.9s
```

#### Expected Result

Listing containers must show four containers running and the port mapping as below:

```shell
docker compose ps
```

```
NAME                          IMAGE                       COMMAND                  SERVICE   CREATED              STATUS              PORTS
docker-compose-demo-nginx-1   docker-compose-demo-nginx   "/docker-entrypoint.…"   nginx     About a minute ago   Up About a minute   0.0.0.0:80->80/tcp, :::80->80/tcp
docker-compose-demo-redis-1   redislabs/redismod          "redis-server --load…"   redis     About a minute ago   Up About a minute   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp
docker-compose-demo-web1-1    docker-compose-demo-web1    "docker-entrypoint.s…"   web1      About a minute ago   Up About a minute   0.0.0.0:81->5000/tcp, :::81->5000/tcp
docker-compose-demo-web2-1    docker-compose-demo-web2    "docker-entrypoint.s…"   web2      About a minute ago   Up About a minute   0.0.0.0:82->5000/tcp, :::82->5000/tcp
```

#### Testing the App

After the application starts, run `curl` multiple times to observer visit count and load balancing.

```shell
curl localhost:80
curl localhost:80
curl localhost:80
curl localhost:80
```

![test-app-curl](./docker-compose.assets/test-app-curl.png) 

Or navigate to `http://<IP_ADDRESS_OF_HOST>:80` in your web browser, refresh the web page and check the visit count.

![test-app-browser](./docker-compose.assets/test-app-browser.png) 

#### Stop and Remove the Containers

`docker compose down` stops containers and removes containers, networks, volumes, and images created by `up`.

By default, the only things removed are:

- Containers for services defined in the Compose file.
- Networks defined in the networks section of the Compose file.
- The default network, if one is used.

Networks and volumes defined as external are never removed.

```shell
docker compose down
```

For more Docker Compose examples: [Samples of Docker Compose applications with multiple integrated services](https://github.com/docker/awesome-compose#samples-of-docker-compose-applications-with-multiple-integrated-services)



---



### Compose Files

https://docs.docker.com/reference/compose-file/

Compose uses YAML files to define microservices applications. We call them *Compose files*, and Compose expects you to name them **`compose.yaml`** or **`compose.yml`**. However, you can specify a different filename with the **`-f`** flag.

#### Name top-level element

The top-level `name` property is defined by the Compose Specification as the project name to be used if you don't set one explicitly. Compose offers a way for you to override this name, and sets a default project name to be used if the top-level `name` element is not set.

```yaml
name: myapp       # top-level name property

services:
  foo:
    image: busybox
    command: echo "I'm running ${COMPOSE_PROJECT_NAME}"
```

#### Services top-level elements

> A service is an abstract definition of a computing resource within an application which can be scaled or replaced independently from other components. Services are backed by a set of containers, run by the platform according to replication requirements and placement constraints. As services are backed by containers, they are defined by a Docker image and set of runtime arguments. All containers within a service are identically created with these arguments.
>
> A Compose file must declare a `services` top-level element as a map whose keys are string representations of service names, and whose values are service definitions. A service definition contains the configuration that is applied to each service container.
>
> Each service may also include a `build` section, which defines how to create the Docker image for the service. Compose supports building Docker images using this service definition. If not used, the `build` section is ignored and the Compose file is still considered valid. Build support is an optional aspect of the Compose Specification.

The following example demonstrates how to define two simple services, set their images, map ports, and configure basic environment variables using Docker Compose.

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: example
      POSTGRES_DB: exampledb
```

In the following example, the `proxy` service uses the Nginx image, mounts a local Nginx configuration file into the container, exposes port `80` and depends on the `backend` service.

The `backend` service builds an image from the Dockerfile located in the `backend` directory that is set to build at stage `builder`.

```yaml
services:
  proxy:
    image: nginx
    volumes:
      - type: bind
        source: ./proxy/nginx.conf
        target: /etc/nginx/conf.d/default.conf
        read_only: true
    ports:
      - 80:80
    depends_on:
      - backend

  backend:
    build:
      context: backend
      target: builder
```

##### Attributes

[Full list of services top-level elements attributes](https://docs.docker.com/reference/compose-file/services/#attributes)

###### `container_name`

`container_name` is a string that specifies a custom container name, rather than a name generated by default.

```yml
container_name: my-web-container
```

Compose does not scale a service beyond one container if the Compose file specifies a `container_name`. Attempting to do so results in an error.

###### `depends_on`

With the `depends_on` attribute, you can control the order of service startup and shutdown. It is useful if services are closely coupled, and the startup sequence impacts the application's functionality.

The *short syntax* variant only specifies service names of the dependencies. Service dependencies cause the following behaviors:

- Compose creates services in dependency order. In the following example, `db` and `redis` are created before `web`.
- Compose removes services in dependency order. In the following example, `web` is removed before `db` and `redis`.

```yaml
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

Compose guarantees dependency services have been started before starting a dependent service. Compose waits for dependency services to be "ready" before starting a dependent service.

The *long form syntax* enables the configuration of additional fields that can't be expressed in the short form.

- `restart`: When set to `true` Compose restarts this service after it updates the dependency service. This applies to an explicit restart controlled by a Compose operation, and excludes automated restart by the container runtime after the container dies. Introduced in Docker Compose version [2.17.0](https://docs.docker.com/compose/releases/release-notes/#2170).
- `condition`: Sets the condition under which dependency is considered satisfied
  - `service_started`: An equivalent of the short syntax described previously
  - `service_healthy`: Specifies that a dependency is expected to be "healthy" (as indicated by [`healthcheck`](https://docs.docker.com/reference/compose-file/services/#healthcheck)) before starting a dependent service.
  - `service_completed_successfully`: Specifies that a dependency is expected to run to successful completion before starting a dependent service.
- `required`: When set to `false` Compose only warns you when the dependency service isn't started or available. If it's not defined the default value of `required` is `true`. Introduced in Docker Compose version [2.20.0](https://docs.docker.com/compose/releases/release-notes/#2200).

Service dependencies cause the following behaviors:

- Compose creates services in dependency order. In the following example, `db` and `redis` are created before `web`.
- Compose waits for healthchecks to pass on dependencies marked with `service_healthy`. In the following example, `db` is expected to be "healthy" before `web` is created.
- Compose removes services in dependency order. In the following example, `web` is removed before `db` and `redis`.

```yaml
services:
  web:
    build: .
    depends_on:
      db:
        condition: service_healthy
        restart: true
      redis:
        condition: service_started
  redis:
    image: redis
  db:
    image: postgres
```

Example:

https://immich.app/docs/install/docker-compose

https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml

###### `deploy`

`deploy` specifies the configuration for the deployment and lifecycle of services, as defined [in the Compose Deploy Specification](https://docs.docker.com/reference/compose-file/deploy/).

###### `dns`

`dns` defines custom DNS servers to set on the container network interface configuration. It can be a single value or a list.

```yaml
dns:
  - 8.8.8.8
  - 9.9.9.9
```

###### `entrypoint`

`entrypoint` declares the default entrypoint for the service container. This overrides the `ENTRYPOINT` instruction from the service's Dockerfile.

If `entrypoint` is non-null, Compose ignores any default command from the image, for example the `CMD` instruction in the Dockerfile.

See also [`command`](https://docs.docker.com/reference/compose-file/services/#command) to set or override the default command to be executed by the entrypoint process.

In its short form, the value can be defined as a string:

```yml
entrypoint: /code/entrypoint.sh
```

Alternatively, the value can also be a list, in a manner similar to the [Dockerfile](https://docs.docker.com/reference/dockerfile/#cmd):

```yml
entrypoint:
  - php
  - -d
  - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
  - -d
  - memory_limit=-1
  - vendor/bin/phpunit
```

If the value is `null`, the default entrypoint from the image is used.

If the value is `[]` (empty list) or `''` (empty string), the default entrypoint declared by the image is ignored, or in other words, overridden to be empty.

###### `env_file`

The `env_file` attribute is used to specify one or more files that contain environment variables to be passed to the containers.

```yml
env_file: .env
```

Relative paths are resolved from the Compose file's parent folder. As absolute paths prevent the Compose file from being portable, Compose warns you when such a path is used to set `env_file`.

Environment variables declared in the [`environment`](https://docs.docker.com/reference/compose-file/services/#environment) section override these values. This holds true even if those values are empty or undefined.

`env_file` can also be a list. The files in the list are processed from the top down. For the same variable specified in two environment files, the value from the last file in the list stands.

```yml
env_file:
  - ./a.env
  - ./b.env
```

List elements can also be declared as a mapping, which then lets you set additional attributes.

Example:

https://linkding.link/installation/

https://github.com/sissbruecker/linkding/blob/master/docker-compose.yml

###### `environment`

The `environment` attribute defines environment variables set in the container. `environment` can use either an array or a map. Any boolean values; true, false, yes, no, should be enclosed in quotes to ensure they are not converted to True or False by the YAML parser.

Environment variables can be declared by a single key (no value to equals sign). In this case Compose relies on you to resolve the value. If the value is not resolved, the variable is unset and is removed from the service container environment.

Map syntax:

```yml
environment:
  RACK_ENV: development
  SHOW: "true"
  USER_INPUT:
```

Array syntax:

```yml
environment:
  - RACK_ENV=development
  - SHOW=true
  - USER_INPUT
```

When both `env_file` and `environment` are set for a service, values set by `environment` have precedence.

Example:

https://docs.stirlingpdf.com/Installation/Docker%20Install

###### `expose`

`expose` defines the (incoming) port or a range of ports that Compose exposes from the container. These ports must be accessible to linked services and should not be published to the host machine. Only the internal container ports can be specified.

Syntax is `<portnum>/[<proto>]` or `<startport-endport>/[<proto>]` for a port range. When not explicitly set, `tcp` protocol is used.

```yml
expose:
  - "3000"
  - "8000"
  - "8080-8085/tcp"
```

###### `healthcheck`

The `healthcheck` attribute declares a check that's run to determine whether or not the service containers are "healthy". It works in the same way, and has the same default values, as the HEALTHCHECK Dockerfile instruction set by the service's Docker image. Your Compose file can override the values set in the Dockerfile.

For more information on `HEALTHCHECK`, see the [Dockerfile reference](https://docs.docker.com/reference/dockerfile/#healthcheck).

```yml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
  start_interval: 5s
```

`interval`, `timeout`, `start_period`, and `start_interval` are [specified as durations](https://docs.docker.com/reference/compose-file/extension/#specifying-durations). Introduced in Docker Compose version [2.20.2](https://docs.docker.com/compose/releases/release-notes/#2202)

`test` defines the command Compose runs to check container health. It can be either a string or a list. If it's a list, the first item must be either `NONE`, `CMD` or `CMD-SHELL`. If it's a string, it's equivalent to specifying `CMD-SHELL` followed by that string.

```yml
# Hit the local web app
test: ["CMD", "curl", "-f", "http://localhost"]
```

Using `CMD-SHELL` runs the command configured as a string using the container's default shell (`/bin/sh` for Linux). Both of the following forms are equivalent:

```yml
test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]
```

```yml
test: curl -f https://localhost || exit 1
```

`NONE` disables the healthcheck, and is mostly useful to disable the Healthcheck Dockerfile instruction set by the service's Docker image. Alternatively, the healthcheck set by the image can be disabled by setting `disable: true`:

```yml
healthcheck:
  disable: true
```

###### `image`

`image` specifies the image to start the container from. `image` must follow the Open Container Specification [addressable image format](https://github.com/opencontainers/org/blob/master/docs/docs/introduction/digests.md), as `[<registry>/][<project>/]<image>[:<tag>|@<digest>]`.

```yml
    image: redis
    image: redis:5
    image: redis@sha256:0ed5d5928d4737458944eb604cc8509e245c3e19d02ad83935398bc4b991aac7
    image: library/redis
    image: docker.io/library/redis
    image: my_private.registry:5000/redis
```

If the image does not exist on the platform, Compose attempts to pull it based on the `pull_policy`. If you are also using the [Compose Build Specification](https://docs.docker.com/reference/compose-file/build/), there are alternative options for controlling the precedence of pull over building the image from source, however pulling the image is the default behavior.

`image` may be omitted from a Compose file as long as a `build` section is declared. If you are not using the Compose Build Specification, Compose won't work if `image` is missing from the Compose file.

###### `links`

`links` defines a network link to containers in another service. Either specify both the service name and a link alias (`SERVICE:ALIAS`), or just the service name.

```yml
web:
  links:
    - db
    - db:database
    - redis
```

Containers for the linked service are reachable at a hostname identical to the alias, or the service name if no alias is specified.

Links are not required to enable services to communicate. When no specific network configuration is set, any service is able to reach any other service at that service’s name on the `default` network. If services specify the networks they are attached to, `links` does not override the network configuration. Services that are not connected to a shared network are not be able to communicate with each other. Compose doesn't warn you about a configuration mismatch.

Links also express implicit dependency between services in the same way as [`depends_on`](https://docs.docker.com/reference/compose-file/services/#depends_on), so they determine the order of service startup.

###### `ports`

The `ports` is used to define the port mappings between the host machine and the containers. This is crucial for allowing external access to services running inside containers. It can be defined using short syntax for simple port mapping or long syntax, which includes additional options like protocol type and network mode.

The short syntax is a colon-separated string to set the host IP, host port, and container port in the form:

`[HOST:]CONTAINER[/PROTOCOL]` where:

- `HOST` is `[IP:](port | range)` (optional). If it is not set, it binds to all network interfaces (`0.0.0.0`).
- `CONTAINER` is `port | range`.
- `PROTOCOL` restricts ports to a specified protocol either `tcp` or `udp`(optional). Default is `tcp`.

Ports can be either a single value or a range. `HOST` and `CONTAINER` must use equivalent ranges.

You can either specify both ports (`HOST:CONTAINER`), or just the container port. In the latter case, the container runtime automatically allocates any unassigned port of the host.

`HOST:CONTAINER` should always be specified as a (quoted) string, to avoid conflicts with [YAML base-60 float](https://yaml.org/type/float.html).

IPv6 addresses can be enclosed in square brackets.

Examples:

```yml
ports:
  - "3000"
  - "3000-3005"
  - "8000:8000"
  - "9090-9091:8080-8081"
  - "49100:22"
  - "8000-9000:80"
  - "127.0.0.1:8001:8001"
  - "127.0.0.1:5000-5010:5000-5010"  
  - "::1:6000:6000"   
  - "[::1]:6001:6001" 
  - "6060:6060/udp"   
```

The long form syntax lets you configure additional fields that can't be expressed in the short form.

- `target`: The container port.
- `published`: The publicly exposed port. It is defined as a string and can be set as a range using syntax `start-end`. It means the actual port is assigned a remaining available port, within the set range.
- `host_ip`: The host IP mapping. If it is not set, it binds to all network interfaces (`0.0.0.0`).
- `protocol`: The port protocol (`tcp` or `udp`). Defaults to `tcp`.
- `app_protocol`: The application protocol (TCP/IP level 4 / OSI level 7) this port is used for. This is optional and can be used as a hint for Compose to offer richer behavior for protocols that it understands. Introduced in Docker Compose version [2.26.0](https://docs.docker.com/compose/releases/release-notes/#2260).
- `mode`: Specifies how the port is published in a Swarm setup. If set to `host`, it publishes the port on every node in Swarm. If set to `ingress`, it allows load balancing across the nodes in Swarm. Defaults to `ingress`.
- `name`: A human-readable name for the port, used to document it's usage within the service.

```yml
ports:
  - name: web
    target: 80
    host_ip: 127.0.0.1
    published: "8080"
    protocol: tcp
    app_protocol: http
    mode: host

  - name: web-secured
    target: 443
    host_ip: 127.0.0.1
    published: "8083-9000"
    protocol: tcp
    app_protocol: https
    mode: host
```

###### `runtime`

`runtime` specifies which runtime to use for the service’s containers.

For example, `runtime` can be the name of [an implementation of OCI Runtime Spec](https://github.com/opencontainers/runtime-spec/blob/master/implementations.md), such as "runc".

```yml
web:
  image: busybox:latest
  command: true
  runtime: runc
```

The default is `runc`. To use a different runtime, see [Alternative runtimes](https://docs.docker.com/engine/daemon/alternative-runtimes/).

Read

https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#configuring-docker

https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/sample-workload.html#running-a-sample-workload

###### `volumes`

The `volumes` attribute define mount host paths or named volumes that are accessible by service containers. You can use `volumes` to define multiple types of mounts; `volume`, `bind`, `tmpfs`, or `npipe`.

If the mount is a host path and is only used by a single service, it can be declared as part of the service definition. To reuse a volume across multiple services, a named volume must be declared in the `volumes` top-level element.

The following example shows a named volume (`db-data`) being used by the `backend` service, and a bind mount defined for a single service.

```yml
services:
  backend:
    image: example/backend
    volumes:
      - type: volume
        source: db-data
        target: /data
        volume:
          nocopy: true
          subpath: sub
      - type: bind
        source: /var/run/postgres/postgres.sock
        target: /var/run/postgres/postgres.sock

volumes:
  db-data:
```

For more information about the `volumes` top-level element, see [Volumes](https://docs.docker.com/reference/compose-file/volumes/).



---



### Run Docker Compose services with GPU access

https://docs.docker.com/compose/how-tos/gpu-support/

> Compose services can define GPU device reservations if the Docker host contains such devices and the Docker Daemon is set accordingly.

GPUs are referenced in a `compose.yaml` file using the [device](https://docs.docker.com/reference/compose-file/deploy/#devices) attribute from the Compose Deploy specification, within your services that need them.

This provides more granular control over a GPU reservation as custom values can be set for the following device properties:

- `capabilities`. This value is specified as a list of strings. For example, `capabilities: [gpu]`. You must set this field in the Compose file. Otherwise, it returns an error on service deployment.
- `count`. Specified as an integer or the value `all`, represents the number of GPU devices that should be reserved (providing the host holds that number of GPUs). If `count` is set to `all` or not specified, all GPUs available on the host are used by default.
- `device_ids`. This value, specified as a list of strings, represents GPU device IDs from the host. You can find the device ID in the output of `nvidia-smi` on the host. If no `device_ids` are set, all GPUs available on the host are used by default.
- `driver`. Specified as a string, for example `driver: 'nvidia'`
- `options`. Key-value pairs representing driver specific options.

Example of a Compose file for running a service with access to 1 GPU device:

```yaml
services:
  test:
    image: nvidia/cuda:12.9.0-base-ubuntu22.04
    command: nvidia-smi
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

