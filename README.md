# Cluster RabbitMQ :rabbit: with Consul

There are a lots of good options if you want to run a [RabbitMQ](https://hub.docker.com/_/rabbitmq/) cluster in [docker](http://docker.com/). Here's an solution that only rely on [docker official images](https://hub.docker.com/_/rabbitmq/) :tada:

The main benifit with this approach is that you can use [any version](https://hub.docker.com/r/library/rabbitmq/tags/) of RabbitMQ, which is maintaied by docker and will be up-to-date with future releases.

For information on how to achieve [clustering](https://www.rabbitmq.com/clustering.html) using peer discovery, refer to the rabbitmq official website.
  - [Cluster Formation and Peer Discovery](https://www.rabbitmq.com/cluster-formation.html)
  - [RabbitMQ Docker-compose Sample: Peer Discovery with Consul](https://github.com/rabbitmq/rabbitmq-server/tree/main/deps/rabbitmq_peer_discovery_consul/examples/compose_consul_haproxy)
  - Consul Deploy Sample
    - [Deploy a Consul datacenter(1 server and 1 client)](https://github.com/hashicorp/learn-consul-docker/tree/main/datacenter-deploy)
    - [Consul configuration (hcl format)](https://github.com/hashicorp/learn-consul-docker/blob/main/datacenter-deploy-observability/consul/config.hcl)
  - RabbitMQ Configuration Sample
    - [rabbitmq.conf.example (Github)](https://github.com/rabbitmq/rabbitmq-server/blob/main/deps/rabbit/docs/rabbitmq.conf.example)
    - [advanced.config.example (Github)](https://github.com/rabbitmq/rabbitmq-server/blob/main/deps/rabbit/docs/advanced.config.example)
    - [set_rabbitmq_policy.sh.example (Github)](https://github.com/rabbitmq/rabbitmq-server/blob/main/deps/rabbit/docs/set_rabbitmq_policy.sh.example)

## Changelog after forked (at 2023/04/10)

* Apply Auto-Clustering
  - Enable clustering plug-in; `rabbitmq_peer_discovery_consul`
  - Apply useful setting to rabbitmq.conf, from "peer discovery with consul sample"

* Change container image tag to specific version
  - Update image tag of RabbitMQ: `3-management` to `3-management-alpine`
  - Update image tag of HAProxy: `1.7` to `lts-alpine`
  - Add image tag of Consul: `latest` to `1-debian-11`

* Add some plugins for RabbitMQ
  - `rabbitmq_management_agent`
  - `rabbitmq_peer_discovery_consul`
  - `rabbitmq_tracing`
  - `rabbitmq_web_mqtt`
  - `rabbitmq_web_stomp`

* Add `depends_on` of containers; `Consul(consul) >> HAProxy(haproxy) >> RabbitMQ(rabbit)`

* Add `always` restart policy for all containers.

* Add container-shared network as `bridge mode`: `rabbitmq-network`

## Install

```
> # Clone repository
> git clone https://github.com/pseudojo/docker-rabbitmq-cluster.git
> cd docker-rabbitmq-cluster
>
> # Start development
> docker compose up
> 
> # Start long-options with daemon
> docker compose --env-file ./.env --file ./docker-compose.yml up -d
>
> # How to scale-up RabbitMQ : --scale rabbit=<NUMBER>
> docker compose up -d --scale rabbit=7
> # ... or 
> docker compose --env-file ./.env --file ./docker-compose.yml up -d --scale rabbit=7
```

Most things will be how you expect:

* The default username and password are `rabbitmq`/`changeme`
* The broker accepts connections on `localhost:5672`
  - AMQP : `localhost:5672`
  - MQTT : `localhost:1883`
  - STOMP : `localhost:15672`
* The Management interface is found at `localhost:15672`

## How to monitor/view for containers included RabbitMQ, HAProxy that.
```
> # check containers
> docker compose ps
>
> # trailing logs with latest 100 lines for all containers
> docker compose logs --tail=100 -f
```

## Customize

The `.env` file contains environment variables that can be used to change the default username, password and virtual host.

## HAProxy

This `docker-compose.yml` file comes with the latest version of [HAProxy](http://www.haproxy.org/), an open source software that provides a high availability load balancer and proxy server.

It should be fairly easy to add a [`port mapping`](https://docs.docker.com/compose/compose-file/#ports) for the individual containers if it is desired to connect to a specific broker node.

For monitoring, Connect to HAProxy Statistics Page. URL is `localhost:1936`.

## Uninstall

```
> cd docker-rabbitmq-cluster
>
> # Stop rabbitmq-cluster smoothly.
> docker compose down
>
> # Stop rabbitmq-cluster quickly. (3 seconds timeout)
> docker compose down --timeout 3 --remove-orphans
>
> # Purge all
> docker compose down --rmi all --remove-orphans --timeout 3
```

## Read more

I wrote [a blog post](http://fellowdeveloper.se/2017/05/24/cluster-rabbitmq-in-docker/) that explains some of the ideas behind this repo.

