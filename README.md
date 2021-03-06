## Apache Log Analysis in 5 Minutes with ELK & Docker

(ELK: Elasticsearch, Logstash & Kibana)

A simple demo to show how docker & docker-compose make it easy to run useful services.

In this demo we are going to spin up the following applications:

* logstash
* elasticsearch
* kibana
* drupal

with only a 12-line yaml file and one command!

#### Dependencies
* docker
* netcat, or one of it's ilk (nc, ncat, socat)

Not going to go into details here on docker installation, though.

### 1. Clone this repo

1. `git https://github.com/fmbento/apache-elk-in-five-minutes.git`
1. `cd apache-elk-in-five-minutes`

### 2. Make & activate a virtualenv (optional but recommended)

1. `virtualenv venv`
1. `source venv/bin/activate`

### 3. Install docker-compose

Either `pip install -r requirements.txt` or `pip install docker-compose`

### 4. Create a .env file

Create a file called `.env` with the following contents:

```
LOGSTASH_CONFIG_URL=https://raw.githubusercontent.com/fmbento/apache-elk-in-five-minutes/master/logstash.conf
```

### 5. Run the container

    %> docker run -d --name elkapache --env-file=.env -p "5200:9200" -p "5601:9292" -p "3333:3333" pblittle/docker-logstash

If you want just test it with a ad-hoc service, you could try drupal with:

    %> docker run -d -p "80:8080" drupal:latest

If your going to analyse big logs, better to disable logstash logging when creating and running the container (flag: --log-driver=none):

    docker run -d --name elkapache --log-driver=none --env-file=.env -p "8200:9200" -p "5601:9292" -p "3333:3333" pblittle/docker-logstash

#### 5a. Adjust ES port, Kibana config, if you are already using another ES (port 9200):

    docker exec -it elkapache /bin/bash
    root@21afd475079d:/opt/logstash# sed -i 's/9200/8200/g' ./vendor/kibana/config.js
    root@21afd475079d:/opt/logstash# exit
    docker restart elkapache

### 6. Check that the services are running

`docker ps` will give you a list of running containers. You should see 2.

Browse to...

* elasticsearch: [http://localhost:5600]()
* kibana: [http://localhost:5601]()
* drupal: [http://localhost:8080]()

### 7a. Just testing? Lets pipe our drupal apache logs into ELK!

1. get the drupal container id from `docker ps`.
2. run `docker logs -f [container_id] 2>&1 | nc localhost 3333`

### 7b. Or pipe an apache log file from somewhere else into logstash

`cat /var/log/apache2/access.log | nc localhost 3333`

### 8. Kibana

You should now be able to go back and forth between [drupal](http://localhost:8080) and [kibana](http://localhost:5601) and see the drupal apache log events populating the default dashboard.

[Logstach dashboard](http://localhost:5601/index.html#/dashboard/file/logstash.json)

### 9. Maintaince and used space

If you haven't disabled logging when launching the container:

    docker inspect --format='{{.LogPath}}' elkapache | xargs sudo ls -lash

and if too big, prune it with

    docker inspect --format='{{.LogPath}}' elkapache | xargs sudo tee



