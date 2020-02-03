# Lab: Setup ELK

> **Difficulty**: Intermediate

> **Time**: Approximately 30 minutes

This lab focuses on understanding and deploying ELK for logging and monitoring.

The ELK Stack (Elasticsearch, Logstash and Kibana) can be installed on a variety of different operating systems and in various different setups. While the most common installation setup is Linux and other Unix-based systems, a less-discussed scenario is using Docker.


## Running our Dockerized ELK
There are various ways to install the stack with Docker. You can pull Elastic’s individual images and run the containers separately or use Docker Compose to build the stack from a variety of available images on the Docker Hub.

For this tutorial, I am using a ```https://github.com/deviantony/docker-elk``` - **Dockerized ELK Stack** that results in: three Docker containers running in parallel, for Elasticsearch, Logstash and Kibana, port forwarding set up, and a data volume for persisting Elasticsearch data.


```
git clone https://github.com/deviantony/docker-elk.git
cd /docker-elk
docker-compose up -d
```

### Verifying the installation
It might take a while before the entire stack is pulled, built and initialized. After a few minutes, you can begin to verify that everything is running as expected.

Start with listing your containers:

You’ll notice that ports on my localhost have been mapped to the default ports used by Elasticsearch (9200/9300), Kibana (5601) and Logstash (5000/5044).

```
Everything is already pre-configured with a privileged username and password:

user: elastic
password: changeme
```

You can now query Elasticsearch using:

```
curl http://localhost:9200/_security/_authenticate
{
  "name" : "VO32TCU",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "pFgIXMErShCm1R1cd3JgTg",
  "version" : {
    "number" : "6.1.0",
    "build_hash" : "c0c1ba0",
    "build_date" : "2017-12-12T12:32:54.550Z",
    "build_snapshot" : false,
    "lucene_version" : "7.1.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

And finally, access Kibana by entering: http://localhost:5601 in your browser.


