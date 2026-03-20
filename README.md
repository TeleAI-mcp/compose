[![Docker Compose](https://github.com/docker/compose/blob/main/logo.png?raw=true)](https://www.docker.com/)

[![GitHub contributors](https://img.shields.io/github/contributors/docker/compose)](https://github.com/docker/compose/graphs/contributors)
[![GitHub stars](https://img.shields.io/github/stars/docker/compose)](https://github.com/docker/compose/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/docker/compose)](https://github.com/docker/compose/network)
[![Build Status](https://img.shields.io/github/actions/workflow/status/docker/compose-ci.svg)](https://github.com/docker/compose/actions)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://github.com/docker/compose/actions)
[![Docker Hub](https://img.shields.io/badge/docker-ready-blue.svg)](https://hub.docker.com/r/docker/compose/tags)
[![License](https://img.shields.io/github/license/docker/compose)](https://github.com/docker/compose/blob/main/LICENSE)
[![Twitter](https://img.shields.io/twitter/follow/dockercompose.svg?label=Follow&style=social)](https://twitter.com/compose)

# Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. 
With Compose, you use a YAML file to configure your application's services. 
Then, with a single command, you create and start all the services from your configuration.

## Using Compose is basically a three-step process:

1. Define your app's environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment:
   ```yaml
   services:
     web:
       build: .
       ports:
         - "5000:5000"
       volumes:
         - .:/code
       depends_on:
         - redis
     redis:
       image: redis
   ```
3. Run `docker compose up` from the project directory and the Docker Compose command starts and runs your entire app. 
   You can alternatively run `docker-compose up` with the hyphen.
   
   There is also the ability to scale services with the `--scale` flag:
   ```bash
   docker compose up --scale web=3 -d
   ```

## Installing Compose

### On macOS, Windows and Linux

You can install Docker Compose on:

- macOS: [Docker Desktop for Mac](https://docs.docker.com/desktop/install/mac-install/)
- Windows: [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)
- Linux: [Install Docker Compose](https://docs.docker.com/compose/install/linux/)

### Pre-release builds

If you're interested in pre-release builds, you can find them in the [nightly releases](https://github.com/docker/compose-nightly/releases) on GitHub.
Release candidates are published to Docker Hub with the `rc` suffix, for example:
```bash
docker pull docker/compose:rc
```

## Quick Start

Using Docker Compose is basically a 5 step process:

1. Create an empty directory project.
2. Create a file called `app.py` inside the directory.
   ```python
   import time
   
   import redis
   from flask import Flask
   
   app = Flask(__name__)
   cache = redis.Redis(host='redis', port=6379)
   
   
   def get_hit_count():
       retries = 5
       while True:
           try:
               return cache.incr('hits')
           except redis.exceptions.ConnectionError as exc:
               if retries == 0:
                   raise exc
               retries -= 1
               time.sleep(0.5)
   
   
   @app.route('/')
   def hello():
       count = get_hit_count()
       return f'Hello from Redis! I have been seen {count} times.\n'
   ```

3. Create a file called `requirements.txt` inside the directory.
   ```text
   flask
   redis
   ```

4. Create a file called `Dockerfile` inside the directory.
   ```dockerfile
   FROM python:3.7-alpine
   WORKDIR /code
   COPY requirements.txt requirements.txt
   RUN pip install -r requirements.txt
   COPY . .
   CMD ["python", "app.py"]
   ```

5. Create a file called `docker-compose.yml` inside the directory.
   ```yaml
   services:
     web:
       build: .
       ports:
         - "5000:5000"
     redis:
       image: "redis:alpine"
   ```

6. Run `docker compose up -d`.
   ```bash
   $ docker compose up -d
   
   Starting compose_redis_1 ... done
   Starting compose_web_1   ... done
   ```

That's it! Your application is now running on port 5000. In this example, Redis is running as a service.

- To stop the services, run `docker compose stop`. 
  You can use `docker compose down` to stop and remove containers, networks, volumes, and images created by `up`.

- To see logs, run `docker compose logs -f`.

- To build the images before starting the containers use `docker compose build --no-cache`.

- To get the containers IP addresses (useful for networking) run `docker inspect <container-name>`.

- To run a command within a service's container use `docker compose exec <service> <command>`.
  For example:
  ```bash
  docker compose exec redis redis-cli
  ```

## Useful Links

- [Docker Compose documentation](https://docs.docker.com/compose/)
- [Docker Compose file reference](https://docs.docker.com/compose/compose-file/)
- [Ask on the Docker Community Forums](https://discuss.docker.com/c/compose)
- [Join the Docker Community Slack](https://dockr.ly/slack) and then post to [#compose](https://dockercommunity.slack.com/archives/Compose)

## Contributing

Interested in contributing? See the [contributing guide](CONTRIBUTING.md).

## License

Docker Compose is licensed under the Apache License, Version 2.0.
See [LICENSE](LICENSE) for the full license text.

## Code of Conduct

Please read and follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## Releases

The release process is described in the [releases.md](releases.md) file.

## Building

Information about building the Compose CLI from source is described in the [BUILDING.md](BUILDING.md) file.

## Control Docker Compose behavior

You can specify override files (using `-f` flag) to control how Docker Compose behaves.

Docker Compose supports multiple configuration files, merged in the order they are specified in.

This allows people to customize their configuration based on environment, situation, and/or projects.

See the [Override file](#extending-services) section below.

### Prerequisites

Make sure you have already installed both [Docker Engine](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/), and that you have created the directory structure for your project from the [Quickstart](#quickstart) example.

### Understanding multiple compose files

#### Extending services

Docker Compose supports two methods for defining per-project override files, and also allows you to
extend the service under a different configuration.

The methods are:
1. Using [multiple compose files](https://docs.docker.com/compose/multiple-compose-files/), as described in
   the next section.
2. Using [`extends` keyword](https://docs.docker.com/compose/compose-file/#extends), that allows you to
   extend services across multiple compose files.

Using the `-f` flag also works with the `extends` keyword.

#### Multiple compose files

With multiple compose files, you can customize the `compose.yaml` for different environments or runtimes.

**Base compose file** (`compose.yaml`):
```yaml
services:
  web:
    image: myapp/web:latest
  db:
    image: mysql:latest
```

**Override compose file** (`composeOverride.yaml`):
```yaml
services:
  web:
    build: .
    environment:
      - DEBUG=1
  db:
    command: --default-authentication-plugin=mysql_native_password
```

If you run `docker compose up`, the base and the override file are merged together. If the same option is defined in both files, the value from the override file takes precedence.

To use multiple override files, or an override file with a different name, specify them individually using the `-f` flag:

```bash
docker compose -f compose.yaml -f composeOverride.yaml up -d
```

You can also use additional files that don't follow the `composeOverride.yaml` naming convention using the `-f` flag.
When files are merged, the settings from the override files always take precedence.

**Note**: For the following example, you should have created the `webapp` directory already from the [Quickstart](#quickstart) section.

For example, a development environment could be defined with:
* `compose.yaml` as the base file, which defines persistent configuration for development.
* `compose-dev.yaml` as the override file, which includes configuration for development (bind mounts, debug ports).

A production environment could be defined with:
* `compose.yaml` as the base file.
* `compose-prod.yaml` as the override file, which includes configuration for production (replicas, resource limits, SSL).

To deploy the application in the production environment:

```bash
docker compose -f compose.yaml -f compose-prod.yaml up -d
```

When stacking multiple files, define settings in the file that comes first. For example, if you want to use a different database in development and production, you could use the following:

**Base file (compose.yaml):**
```yaml
services:
  app:
    image: myapp
    # Persist the database
    volumes:
      - db-data:/opt/mounts
```

**Development override (compose-dev.yaml):**
```yaml
services:
  app:
    build: .
    volumes:
      - .:/code
      - /code/node_modules  # Persists the node_modules from the container's last build
  db:
    image: postgres-dev
```

**Production override (compose-prod.yaml):**
```yaml
services:
  app:
    image: myapp
    # No volumes, in production we use replicas instead
  db:
    image: postgres-prod
```

> **Note**: Because volumes are not cleared by setting `volumes` to an empty table in the service definition (as is the case in the example above), if you want to clear a volume you must explicitly declare the volume as empty in the overriding file.

For example:
```yaml
services:
  app:
    volumes: []
  db:
    image: postgres-prod
```

Alternatively, you can achieve the same behavior by defining volumes at the [top level](#volumes), which is useful when you want a persistent volume to be used by multiple services, and you want the volume to be created only once.

#### Using `profiles`

With profiles you can define a set of inactive services to be started in a specific runtime configuration using the `--profile` flag:

```yaml
# compose.yaml
services:
  frontend:
    image: frontend
    profiles: [frontend]

  backend:
    image: backend
    profiles: [backend]

  orchestrator:
    image: orchestrator
    profiles: [orchestrator]
```

```bash
# Starting specific profile
docker compose --profile frontend --profile backend up

# Starting all services for a profile
docker compose --profile frontend up
```

Using `docker compose up` will start services that have no profile defined. You can use `COMPOSE_PROFILES` environment variable to set the profiles for the Compose command line:

```bash
COMPOSE_PROFILES=frontend,backend docker compose up
```

#### Changing the project name

By default, the project name is the directory you run `docker compose` from.
You can change it using the `-p` flag or the `COMPOSE_PROJECT_NAME` environment variable.

```bash
docker compose -p my_project up
```

The `-p` flag also allows you to specify a project name to use for create, which uses this name for related resources (networks, volumes).

## Resource formatters

The `docker compose config` command can optionally format your compose file using the `--format` flag:

```bash
# Format the compose.yaml file to stdout using the 'json' format
docker compose config --format json

# Format to a file using the 'yaml' format
docker compose config --format yaml > compose.yaml
```

The available formats are `yaml` (default) and `json`.

The formatting works with any command that outputs valid compose files, such as `config`, `convert`, or `ls`.

## Using Prisma with Docker Compose

Prisma, the open-source ORM for Node.js and TypeScript, is a great way to manage your database schema and migrations.
Since Prisma can run inside a container, you can use it with Docker Compose to manage your database schema.

The following is an example of how to set up Prisma with Docker Compose:

```yaml
services:
  prisma:
    image: prismagraphql/prisma:1.34
    ports:
      - "4466:4466"
    environment:
      DATABASE_URL: file:./dev.db
    volumes:
      - ./prisma:/app/prisma
```

To run Prisma Studio, run:

```bash
docker compose run --rm prisma studio
```

## Using Swarm

Docker Compose and Docker Swarm are designed for different purposes. While Compose is for running multi-container applications on a single host, Swarm is designed for managing containers across multiple hosts.

You can use Compose to create containers, but not to deploy them to a Swarm cluster. The `compose` CLI is only a user-facing tool for running standalone Compose commands. 

Docker Compose V2 (and V1) does not have any built-in support for Docker Swarm. If you want to deploy your Compose application to a Swarm cluster, you should use `docker stack deploy`.

To learn more about using Docker Swarm with Docker Compose, see [Deploying to Docker Swarm](https://docs.docker.com/engine/swarm/stacks/).

## Cloud integration

Docker Compose can be used to deploy applications to the following cloud platforms:

- [Amazon ECS](https://docs.docker.com/cloud/ecs-integration/)
- [Microsoft Azure ACI](https://docs.docker.com/cloud/aci-integration/)
- [Google Cloud Run](https://docs.docker.com/cloud/gcp/)

You can use the `docker compose` command to deploy your application to these platforms, as well as to a Docker Swarm cluster.

## Contributing

We welcome contributions! Please read our [contributing guide](CONTRIBUTING.md) to get started.

## Security

For security issues, please see our [security policy](https://github.com/docker/compose/security/policy).

## License

See [LICENSE](LICENSE) for the full license text.

## Code of Conduct

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for the full code of conduct text.

## Acknowledgements

Docker Compose is built by many people across the world. See [AUTHORS](AUTHORS) for the full list of contributors.

---

<div align="center">

[![Docker Compose logo](logo.png)](https://www.docker.com/)

</div>

1. related project [kubernetes/kubernetes](https://github.com/kubernetes/kubernetes)
2. related project [helm/helm](https://github.com/helm/helm)