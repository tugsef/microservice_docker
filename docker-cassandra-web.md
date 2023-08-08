- Dockerfile

```Dockerfile
FROM ruby:2.6.2-alpine
LABEL maintainer 'Doğukan Çağatay <dcagatay@gmail.com>'

ENV VERSION 0.5.0

RUN apk add --no-cache --update --virtual=build-dependencies build-base linux-headers gcc g++ \
    && gem install cassandra-web -v ${VERSION} \
    && apk del build-dependencies \
    && apk add --no-cache --update bash \
    && rm -rf /tmp/* /var/tmp/* /var/cache/apk/*

COPY run.sh /
RUN chmod +x /run.sh

CMD ["/run.sh"]
```
- docker-compose.yml
```yml
version: "3.7"
services:

  cassandra:
    image: bitnami/cassandra:3.11.10-debian-10-r13
    hostname: cassandra
    networks:
      default:
        ipv4_address: 10.6.0.11
    # ports:
    #   - 9042:9042 # CQL
    environment:
      CASSANDRA_SEEDS: 10.6.0.11
      CASSANDRA_USER: cassandra
      CASSANDRA_PASSWORD: cassandra
      CASSANDRA_PASSWORD_SEEDER: "yes"
    restart: unless-stopped

  cassandra-web:
    build: ./
    image: dcagatay/cassandra-web:latest
    depends_on:
      - cassandra
    ports:
      - 3000:3000
    environment:
      CASSANDRA_HOST_IPS: 10.6.0.11
      CASSANDRA_PORT: 9042
      CASSANDRA_USER: cassandra
      CASSANDRA_PASSWORD: cassandra
    restart: unless-stopped

networks:
  default:
    driver: bridge
    ipam:
      config:
        - subnet: 10.6.0.0/24
```

- run.sh
```sh
#!/bin/bash
set -eu

#HOST IPs
if [[ ! -v CASSANDRA_HOST_IPS ]]; then
  CASSANDRA_HOST_IPS="127.0.0.1"
else
  CASSANDRA_HOST_IPS="${CASSANDRA_HOST_IPS}"
fi

#PORT
if [[ ! -v CASSANDRA_PORT ]]; then
  CASSANDRA_PORT="9042"
else
  CASSANDRA_PORT="${CASSANDRA_PORT}"
fi

#USERNAME
if [[ ! -v CASSANDRA_USERNAME ]]; then
  CASSANDRA_USERNAME="cassandra"
else
  CASSANDRA_USERNAME="${CASSANDRA_USERNAME}"
fi

#PASSWORD
if [[ ! -v CASSANDRA_PASSWORD ]]; then
  CASSANDRA_PASSWORD="cassandra"
else
  CASSANDRA_PASSWORD="${CASSANDRA_PASSWORD}"
fi

COMMAND="cassandra-web --hosts $CASSANDRA_HOST_IPS --port $CASSANDRA_PORT --username $CASSANDRA_USERNAME --password $CASSANDRA_PASSWORD"

echo $COMMAND

exec $COMMAND
```
