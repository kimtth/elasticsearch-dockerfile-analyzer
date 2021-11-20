
## Initialize ElasticSearch (Single Cluster) and Kibana

`sudo docker-compose start -d`

This command will build and execute `docker-compose.yml`.

## CSV Data

The Kibana can import and create ElasticSearch index by csv.

`Kibana > Machine Learning > Data Visualizer > Import` 

## Docker Command

- Docker 

```bash
docker run [ID] # Run a command in a new container
docker start  [ID] # Start one or more stopped containers

docker container ls
docker container ls -a

--all, -a : Show all containers (default only running containers)

docker ps -a
docker images
docker rm [CONTAINER_ID]
docker rmi [IMAGE_ID]

sudo docker run -t -i [ID] /bin/bash # Access a console in a docker container.
sudo docker container rm $(sudo docker ps -a -q) # Delete all containers
```

- Docker Compose

```bash
sudo docker-compose down # Remove docker-compose
sudo docker-compose up # Run a command  with build from Dockerfile
sudo docker-compose start # Start one or more stopped composed containers

sudo docker-compose ps -a # List docker-compose
sudo docker-compose logs # Log docker-compose
sudo docker-compose build # Build when Dockerfile changed
 
sudo docker system prune # Error response from daemon: unable to remove volume: remove XXX: volume is in use
sudo docker volume rm $(sudo docker volume ls -q) # Remove all volume. When facing, elasticsearch version downgrade not supported
sudo docker system prune -a --volumes # Remove all unused containers, volumes, networks and images
```

## Docker-Compose Completed Message

```bash
$ sudo docker-compose build
kib01 uses an image, skipping
Building es01
Step 1/4 : FROM docker.elastic.co/elasticsearch/elasticsearch:7.10.2
7.10.2: Pulling from elasticsearch/elasticsearch
ddf49b9115d7: Already exists
e736878e27ad: Pull complete
7487c9dcefbe: Pull complete
...
Digest: sha256:d528cec81720266974fdfe7a0f12fee928dc02e5a2c754b45b9a84c84695bfd9
Status: Downloaded newer image for docker.elastic.co/elasticsearch/elasticsearch:7.10.2
 ---> 0b58e1cea500
Step 2/4 : USER root
 ---> Running in 35078a2200c4
Removing intermediate container 35078a2200c4
 ---> f8402dc51696
Step 3/4 : RUN bin/elasticsearch-plugin install analysis-kuromoji
 ---> Running in bec38be2e6f8
-> Installing analysis-kuromoji
-> Downloading analysis-kuromoji from elastic
[=================================================] 100%?? 
-> Installed analysis-kuromoji
Removing intermediate container bec38be2e6f8
 ---> 7e5130d4d9f1
Step 4/4 : RUN bin/elasticsearch-plugin install analysis-icu
 ---> Running in 345b282c0b8d
-> Installing analysis-icu
-> Downloading analysis-icu from elastic
[=================================================] 100%?? 
-> Installed analysis-icu
Removing intermediate container 345b282c0b8d
 ---> c0fb72f2dc4f
Successfully built c0fb72f2dc4f
Successfully tagged elastic-search-react_es01:latest
WARNING: Image for service es01 was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
```

## Elasticsearch Custom Analyazer [Japanese 日本語] Configuration

In following cases, the index name is [your_index_name].

- Get current settings

```bash
GET /your_index_name/_settings
```

- Try testing with Configured Custom Analyzer.

```bash
GET /your_index_name/_analyze
{
  "text": "梅酒が好き", 
  "analyzer": "my_ja_analyzer"
}
```

- Before starting the setting, close the index.

```bash
POST /your_index_name/_close
```

- After update a setting, re-open [your_index_name] index.  

```bash
POST /your_index_name/_open
```

- Custom Analyzer and type mapping to a field only can be able to configure at the time of index creation. after creating an index, updating a setting is not allowed.

```bash
GET /your_index_name/_settings

>>Result
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "kuromoji": {
          "type": "kuromoji_tokenizer",
          "mode": "normal"
        }
      },
      "analyzer": {
        "my_jp": {
          "type": "custom",
          "tokenizer": "kuromoji",
          "mode": "search",
          "char_filter": [
            "icu_normalizer",
            "kuromoji_iteration_mark"
          ],
          "filter": [
            "cjk_width",
            "kuromoji_baseform",
            "kuromoji_part_of_speech",
            "ja_stop",
            "lowercase",
            "kuromoji_number",
            "kuromoji_stemmer"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "<your_property_name>": {
        "type": "text",
        "analyzer": "my_jp"
      }
    }
  }
}
```

## Ubuntu Swap Memory

- Create Swap Memory : Elastic requires at least 2GB Memory. 

```bash
~$ sudo fallocate -l 2G /swapfile
~$ ls -lh /swapfile
-rw-r--r-- 1 root root 2.0G Jul 14 05:51 /swapfile
~$ sudo chmod 600 /swapfile
~$ ls -lh /swapfile
-rw------- 1 root root 2.0G Jul 14 05:51 /swapfile

~$ sudo mkswap /swapfile
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)no label, UUID=b0ee7549-ccce-4876-bd8d-4dc8558288f2
~$ sudo swapon /swapfile
~$ sudo swapon --show
NAME TYPE SIZE USED PRIO/swapfile file 2G 0B -2
~$ free -h 
total used free shared buff/cache availableMem: 
7.8Gi 3.9Gi 2.6Gi 165Mi 1.2Gi 3.4GiSwap: 2.0Gi 0B 2.0Gi

~$ sudo cp /etc/fstab /etc/fstab.bak
~$ echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab/swapfile none swap sw 0 0
~$ cat /proc/sys/vm/swappiness60
```

https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-20-04

## Japanese Tokenizer (How to directly install into Ubuntu, not through Docker)

```bash
$ git clone --depth 1 https://github.com/neologd/mecab-ipadic-neologd.git
$ sudo apt-get install -y mecab # Install MeCab when me-cab is not found.

$ echo `mecab-config --dicdir`"/mecab-ipadic-neologd" # check the 'mecab-config' directory path.
$ sudo apt install libmecab-dev # when 'mecab-config' is not found.
$ ./bin/install-mecab-ipadic-neologd -n # Standard Dictionary -n / All Dictionaries -a
```
