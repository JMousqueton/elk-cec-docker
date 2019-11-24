# ELK-CEC-DOCKER

[![Build Status](https://api.travis-ci.org/JMousqueton/elk-cec-docker.svg?branch=master)](https://travis-ci.org/JMousqueton/elk-cec-docker)

Latest version of the [Elastic stack](https://www.elastic.co/elk-stack) with Docker and Docker Compose.

Note: An other branche but depreciated is available with [cerebro](https://github.com/lmenezes/cerebro) (listen on port 9000)

## Requirements

### Host setup

1. Install [Docker](https://www.docker.com/community-edition#/download) version **17.05+**
2. Install [Docker Compose](https://docs.docker.com/compose/install/) version **1.6.0+**
3. Clone this repository

Based on the official Docker images from Elastic:

* [elasticsearch](https://github.com/elastic/elasticsearch-docker)
* [logstash](https://github.com/elastic/logstash-docker)
* [kibana](https://github.com/elastic/kibana-docker) 

By default, the stack exposes the following ports:

* 5000: Logstash TCP input
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana

## Configuration

**NOTE**: Configuration is not dynamically reloaded, you will need to restart the stack after any change in the
configuration of a component.

### How can I tune the Kibana configuration?

The Kibana default configuration is stored in `kibana/config/kibana.yml`.

It is also possible to map the entire `config` directory instead of a single file.

### How can I tune the Logstash configuration?

The Logstash configuration is stored in `logstash/config/logstash.yml`.

It is also possible to map the entire `config` directory instead of a single file, however you must be aware that
Logstash will be expecting a
[`log4j2.properties`](https://github.com/elastic/logstash-docker/tree/master/build/logstash/config) file for its own
logging.

### How can I tune the Elasticsearch configuration?

The Elasticsearch configuration is stored in `elasticsearch/config/elasticsearch.yml`.

You can also specify the options you want to override directly via environment variables:

```yml
elasticsearch:

  environment:
    network.host: "_non_loopback_"
    cluster.name: "my-cluster"
```


## Storage

### How can I persist Elasticsearch data?

The data stored in Elasticsearch will be persisted after container reboot but not after container removal.

In order to persist Elasticsearch data even after removing the Elasticsearch container, you'll have to mount a volume on
your Docker host. Update the `elasticsearch` service declaration to:

```yml
elasticsearch:

  volumes:
    - /path/to/storage:/usr/share/elasticsearch/data
```

This will store Elasticsearch data inside `/path/to/storage`.

**NOTE:** 
Beware of the [unprivileged `elasticsearch` user][esuser] is used within the Elasticsearch image, therefore the mounted data directory must be owned by the uid `1000`.

[esuser]: https://github.com/elastic/elasticsearch-docker/blob/016bcc9db1dd97ecd0ff60c1290e7fa9142f8ddd/templates/Dockerfile.j2#L22

## JVM tuning

By default, both Elasticsearch and Logstash start with [1/4 of the total host
memory](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size) allocated to
the JVM Heap Size.

The startup scripts for Elasticsearch and Logstash can append extra JVM options from the value of an environment
variable, allowing the user to adjust the amount of memory that can be used by each component:

| Service       | Environment variable |
|---------------|----------------------|
| Elasticsearch | ES_JAVA_OPTS         |
| Logstash      | LS_JAVA_OPTS         |

To accomodate environments where memory is scarce (Docker for Mac has only 2 GB available by default), the Heap Size
allocation is capped by default to 256MB per service in the `docker-compose.yml` file. If you want to override the
default JVM configuration, edit the matching environment variable(s) in the `docker-compose.yml` file.

For example, to increase the maximum JVM Heap Size for Logstash:

```yml
logstash:

  environment:
    LS_JAVA_OPTS: "-Xmx1g -Xms1g"
```

### Using a newer stack version

To use a different Elastic Stack version than the one currently available in the repository, simply change the version
number inside the `.env` file, and rebuild the stack with:

```console
$ docker-compose build
$ docker-compose up
```

**NOTE**: Always pay attention to the [upgrade instructions](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html)
for each individual component before performing a stack upgrade.
  

