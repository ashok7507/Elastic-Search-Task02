# Elastic-Search
Step-01: Install docker apt repo 

    # Add Docker's official GPG key:
    sudo apt-get update -y
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    # Add the repository to Apt sources:
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update -y

Step-02: Install docker latest version

    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Step-03: Add Dockerfile

    # OPSNJDK use as a base image in these dockerfile
    FROM openjdk:17-slim

    # Set environment variables in dockerfile
    ENV ELASTIC_VERSION=7.17.5 \
        discovery.type=single-node \
        ES_JAVA_OPTS="-Xms512m -Xmx512m"

    # Install necessary dependencies required for installing elasticsearch
    RUN apt-get update && apt-get install -y \
        wget \
        curl \
        gnupg && \
        mkdir -p /usr/share/elasticsearch && \
        mkdir -p /usr/share/elasticsearch/config && \
        mkdir -p /usr/share/elasticsearch/data && \
        mkdir -p /usr/share/elasticsearch/logs

    # Download Elasticsearch
    RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-${ELASTIC_VERSION}-linux-x86_64.tar.gz && \
        tar -xzf elasticsearch-${ELASTIC_VERSION}-linux-x86_64.tar.gz -C /usr/share/elasticsearch --strip-components=1 && \
        rm elasticsearch-${ELASTIC_VERSION}-linux-x86_64.tar.gz

    # Add Elasticsearch as a user and set permissions to them like useradd and groupadd
    RUN groupadd elasticsearch && \
        useradd -g elasticsearch elasticsearch && \
        chown -R elasticsearch:elasticsearch /usr/share/elasticsearch

    # create elasticsearch.yml and copy these file from loacal to container
    COPY elasticsearch.yml /usr/share/elasticsearch/config/elasticsearch.yml

    # Change to the elasticsearch user
    USER elasticsearch

    # Expose port
    EXPOSE 9200 9300

    # Set up the working directory 
    WORKDIR /usr/share/elasticsearch

    # Command to start Elasticsearch
    CMD ["./bin/elasticsearch"]
    
Step-04: Add elasticsearch.yml

    cluster.name: "jonsnow-cluster"
    node.name: "nodeno-first"
    path.data: /usr/share/elasticsearch/data
    path.logs: /usr/share/elasticsearch/logs
    network.host: 0.0.0.0
    http.port: 9200
    discovery.type: single-node

- Building the Image
       
      docker build -t task-elasticsearch .
  
- Run the docker container

      docker run -d --name elasticsearch \
      -p 9200:9200 -p 9300:9300 \
      -v es_data:/usr/share/elasticsearch/data \
      -v es_logs:/usr/share/elasticsearch/logs \
      task-elasticelastic
    
Step-05: Add docker-compose file 
    
    version: '3.8'
    services:
      elasticsearch:
        build:
          context: .
          dockerfile: Dockerfile
        container_name: elasticsearch
        ports:
          - "9200:9200"
          - "9300:9300"
        environment:
          - discovery.type=single-node
          - ES_JAVA_OPTS=-Xms512m -Xmx512m
        volumes:
          - es_data:/usr/share/elasticsearch/data
          - es_logs:/usr/share/elasticsearch/logs
    volumes:
      es_data:
        driver: local
      es_logs:
        driver: local

Build the image from docker-compose file

    docker-compose build
    docker-compose up -d
    

    
    









         

   
