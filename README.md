# Containerised Google Cloud Pub/Sub Emulator

This repository contains the Docker configuration for a Google PubSub emulator. It's mainly the containerisation of 
https://github.com/thinkfluent/pubsubc 

Based on alpine linux, the image is around ~260MB in size (A *lot* smaller than a full-blown gcloud SDK image).

`fluentthinking/gcloud-pubsub-emulator:latest`

## Install & Run

A pre-built Docker container is available from Docker Hub:

```bash
docker run --rm -ti -p 8681:8681 fluentthinking/gcloud-pubsub-emulator:latest
```

Or, you can build this repository yourself:

```bash
docker build -t gcloud-pubsub-emulator:latest --platform=linux/amd64 .
docker run --rm -ti -p 8681:8681 gcloud-pubsub-emulator:latest
```

## Usage

Once the emulator is up and running, you should be able to use any app that has PubSub 
implemented and point it to your Docker container by specifying the `PUBSUB_EMULATOR_HOST` environment variable.

e.g. `PUBSUB_EMULATOR_HOST=localhost:8681`

### Automated Topic and Subscription Creation

This image can also automatically create topics and subscriptions on startup. Configuration via either (or both of)
* [Environment variables](https://github.com/thinkfluent/pubsubc?tab=readme-ov-file#environment-variables)
* [Docker labels](https://github.com/thinkfluent/pubsubc?tab=readme-ov-file#docker-labels) (for use with docker-compose etc.)


#### Environment Example
```dotenv
PUBSUB_PROJECT1=project-name,topic1,topic2:subscription1:subscription2
PUBSUB_PROJECT2=project-two,topicA,topicB:subscriptionX:subscriptionY
PUBSUB_PROJECT3=project-name,topic:push-subscription+myapp/endpoint
```

#### Docker Label Example
```yaml
services:
  # Your application service
  myapp:
    image: busybox:latest
    environment:
      - PUBSUB_EMULATOR_HOST=pubsub-emulator:8681
    labels:
      - 'pubsubc.config1=project-one,topic1,topic2:subscription1:subscription2'
      - 'pubsubc.config2=project-two,topic:push-subscription+endpoint'
```

### wait-for, wait-for-it
If you're using this Docker image in a docker-compose setup or something similar, you might have leveraged scripts like 
[wait-for](https://github.com/eficode/wait-for) or [wait-for-it](https://github.com/vishnubob/wait-for-it) to detect when the PubSub service comes up before starting a container that 
depends on it being up. 

If you're _not_ using the above-mentioned `PUBSUB_PROJECT` environment variable, you can simply 
check if port `8681` is available. If you _do_ depend on one or more `PUBSUB_PROJECT` environment variables, you should 
check for the availability of port `8682` as that one will become available once all the topics and subscriptions have 
been created.

### Port Configuration, Mapping
Whilst the default ports are 8681 for the Pub/Sub emulator and 8682 for the topic-ready check, you can change these by
setting the `PUBSUB_PORT` and `READY_PORT` environment variables respectively.

Or, just map them in your docker run / docker-compose setup:
```yaml
services:
  # Pub/Sub Emulator. We mount the docker socket so we can use the Docker API to fetch configuration labels
  pubsub-emulator:
    image: fluentthinking/gcloud-pubsub-emulator:latest
    ports:
      - '8085:8681'
      - '8086:8682'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

## Prior Art
Based on the work of:
- https://github.com/floatschedule/pubsubc
- https://github.com/marcelcorso/gcloud-pubsub-emulator