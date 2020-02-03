# Lab: Setup ELK

> **Difficulty**: Intermediate

> **Time**: Approximately 30 minutes

This lab focuses on understanding and deploying ELK for logging and monitoring.

The ELK Stack (Elasticsearch, Logstash and Kibana) can be installed on a variety of different operating systems and in various different setups. While the most common installation setup is Linux and other Unix-based systems, a less-discussed scenario is using Docker.


## Running our Dockerized ELK
There are various ways to install the stack with Docker. You can pull Elastic’s individual images and run the containers separately or use Docker Compose to build the stack from a variety of available images on the Docker Hub.

For this tutorial, I am using a ```https://github.com/deviantony/docker-elk``` - **Dockerized ELK Stack** that results in: three Docker containers running in parallel, for Elasticsearch, Logstash and Kibana, port forwarding set up, and a data volume for persisting Elasticsearch data.


```
cd ~
git clone https://github.com/deviantony/docker-elk.git
cd docker-elk
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

**You can query Elasticsearch in browser:**

`http://<public_ip>:9200`
`http://<public_ip>:9200/_security/_authenticate`
`http://<public_ip>:9200/_cat/indices`

**You can check Kibana in browser:**

`http://<public_ip>:5601`


