# docker-solr

## Standalone Solr example

### 1. Start standalone Solr

```sh
$ docker run -d -p 8984:8983 --name solr \
    -e CORE_NAME=collection1 \
    -e CONFIG_SET=data_driven_schema_configs \
    -e DATA_DIR=data \
    mosuka/docker-solr:release-5.5
032dd48d12496c65a3405da483c4c16e4c9b26f3f7e22e0592717cfbd5830110
```

### 2. Check container ID

```sh
$ docker ps
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS              PORTS                                         NAMES
032dd48d1249        mosuka/docker-solr:release-5.5        "/usr/local/bin/docke"   15 seconds ago      Up 14 seconds       0.0.0.0:8984->8983/tcp                       solr
```

### 3. Get container IP

```sh
$ SOLR_CONTAINER_IP=$(docker inspect -f '{{ .NetworkSettings.IPAddress }}' solr)
$ echo ${SOLR_CONTAINER_IP}
172.17.0.2
```

### 4. Get host IP and port

```sh
$ SOLR_HOST_IP=$(docker-machine ip default)
$ echo ${SOLR_HOST_IP}
192.168.99.100

$ SOLR_HOST_PORT=$(docker inspect -f '{{ $port := index .NetworkSettings.Ports "8983/tcp" }}{{ range $port }}{{ .HostPort }}{{ end }}' solr)
$ echo ${SOLR_HOST_PORT}
8984
```

### 5. Open Solr Admin UI in a browser

```sh
$ SOLR_ADMIN_UI=http://${SOLR_HOST_IP}:${SOLR_HOST_PORT}/solr/#/
$ echo ${SOLR_ADMIN_UI}
http://192.168.99.100:8984/solr/#/
```

Open Solr Admin UI in a browser.



## SolrCloud (2 shards across 4 nodes with replication factor 2) example

### 1. Start Zookeeper ensemble

Run ZooKeeper ensemble.

See [https://github.com/mosuka/docker-zookeeper/tree/master/3.5#zookeeper-ensemble-3-nodes-example](https://github.com/mosuka/docker-zookeeper/tree/master/3.5#zookeeper-ensemble-3-nodes-example).

### 2. Start Solr

```sh
$ docker run -d -p 8984:8983 --name=solr1 \
    -e ZK_HOST=${ZOOKEEPER_1_CONTAINER_IP}:2181,${ZOOKEEPER_2_CONTAINER_IP}:2181,${ZOOKEEPER_3_CONTAINER_IP}:2181/solr \
    -e COLLECTION_NAME=collection1 \
    -e NUM_SHARDS=2 \
    -e COLLECTION_CONFIG_NAME=data_driven_schema_configs \
    mosuka/docker-solr:release-5.5
aaef4999bd84f17387a0a868c864cf25154f743fd0519753172f20cac32d7334

$ docker run -d -p 8985:8983 --name=solr2 \
    -e ZK_HOST=${ZOOKEEPER_1_CONTAINER_IP}:2181,${ZOOKEEPER_2_CONTAINER_IP}:2181,${ZOOKEEPER_3_CONTAINER_IP}:2181/solr \
    -e COLLECTION_NAME=collection1 \
    -e NUM_SHARDS=2 \
    -e COLLECTION_CONFIG_NAME=data_driven_schema_configs \
    mosuka/docker-solr:release-5.5
30d83c26c131e65791a9f22eba2f1e7410bda156a634b27ef7593c07c7904753

$ docker run -d -p 8986:8983 --name=solr3 \
    -e ZK_HOST=${ZOOKEEPER_1_CONTAINER_IP}:2181,${ZOOKEEPER_2_CONTAINER_IP}:2181,${ZOOKEEPER_3_CONTAINER_IP}:2181/solr \
    -e COLLECTION_NAME=collection1 \
    -e NUM_SHARDS=2 \
    -e COLLECTION_CONFIG_NAME=data_driven_schema_configs \
    mosuka/docker-solr:release-5.5
18b80967aa730c11c49cd2ea5531044117469be7713de640b1fdc10cd3b8584b

$ docker run -d -p 8987:8983 --name=solr4 \
    -e ZK_HOST=${ZOOKEEPER_1_CONTAINER_IP}:2181,${ZOOKEEPER_2_CONTAINER_IP}:2181,${ZOOKEEPER_3_CONTAINER_IP}:2181/solr \
    -e COLLECTION_NAME=collection1 \
    -e NUM_SHARDS=2 \
    -e COLLECTION_CONFIG_NAME=data_driven_schema_configs \
    mosuka/docker-solr:release-5.5
7f6bf1fe58226942d9899a92c83377429381c93d4cd0075ee8a60c19e69d291b
```

### 5. Check container ID

```sh
$ docker ps
CONTAINER ID        IMAGE                                 COMMAND                  CREATED              STATUS              PORTS                                         NAMES
7f6bf1fe5822        mosuka/docker-solr:release-5.5        "/usr/local/bin/docke"   42 seconds ago       Up 41 seconds       7983/tcp, 0.0.0.0:8987->8983/tcp             solr4
18b80967aa73        mosuka/docker-solr:release-5.5        "/usr/local/bin/docke"   53 seconds ago       Up 52 seconds       7983/tcp, 0.0.0.0:8986->8983/tcp             solr3
30d83c26c131        mosuka/docker-solr:release-5.5        "/usr/local/bin/docke"   About a minute ago   Up About a minute   7983/tcp, 0.0.0.0:8985->8983/tcp             solr2
aaef4999bd84        mosuka/docker-solr:release-5.5        "/usr/local/bin/docke"   2 minutes ago        Up 2 minutes        7983/tcp, 0.0.0.0:8984->8983/tcp             solr1
1902ab480c57        mosuka/docker-zookeeper:release-3.4   "/usr/local/bin/docke"   15 hours ago         Up 15 hours         2888/tcp, 3888/tcp, 0.0.0.0:2184->2181/tcp   zookeeper3
182022f64ef7        mosuka/docker-zookeeper:release-3.4   "/usr/local/bin/docke"   15 hours ago         Up 15 hours         2888/tcp, 3888/tcp, 0.0.0.0:2183->2181/tcp   zookeeper2
a63962346037        mosuka/docker-zookeeper:release-3.4   "/usr/local/bin/docke"   15 hours ago         Up 15 hours         2888/tcp, 3888/tcp, 0.0.0.0:2182->2181/tcp   zookeeper1
```

### 6. Get container IP and port

```sh
$ SOLR_1_CONTAINER_IP=$(docker inspect -f '{{ .NetworkSettings.IPAddress }}' solr1)
$ echo ${SOLR_1_CONTAINER_IP}
172.17.0.5

$ SOLR_2_CONTAINER_IP=$(docker inspect -f '{{ .NetworkSettings.IPAddress }}' solr2)
$ echo ${SOLR_2_CONTAINER_IP}
172.17.0.6

$ SOLR_3_CONTAINER_IP=$(docker inspect -f '{{ .NetworkSettings.IPAddress }}' solr3)
$ echo ${SOLR_3_CONTAINER_IP}
172.17.0.7

$ SOLR_4_CONTAINER_IP=$(docker inspect -f '{{ .NetworkSettings.IPAddress }}' solr4)
$ echo ${SOLR_4_CONTAINER_IP}
172.17.0.8
```

### 10. Get host IP and port

```sh
$ SOLR_HOST_IP=$(docker-machine ip default)
$ echo ${SOLR_HOST_IP}
192.168.99.100

$ SOLR_1_HOST_PORT=$(docker inspect -f '{{ $port := index .NetworkSettings.Ports "8983/tcp" }}{{ range $port }}{{ .HostPort }}{{ end }}' solr1)
$ echo ${SOLR_1_HOST_PORT}
8984

$ SOLR_2_HOST_PORT=$(docker inspect -f '{{ $port := index .NetworkSettings.Ports "8983/tcp" }}{{ range $port }}{{ .HostPort }}{{ end }}' solr2)
$ echo ${SOLR_2_HOST_PORT}
8985

$ SOLR_3_HOST_PORT=$(docker inspect -f '{{ $port := index .NetworkSettings.Ports "8983/tcp" }}{{ range $port }}{{ .HostPort }}{{ end }}' solr3)
$ echo ${SOLR_3_HOST_PORT}
8986

$ SOLR_4_HOST_PORT=$(docker inspect -f '{{ $port := index .NetworkSettings.Ports "8983/tcp" }}{{ range $port }}{{ .HostPort }}{{ end }}' solr4)
$ echo ${SOLR_4_HOST_PORT}
8987
```

### 11. Open Solr Admin UI in a browser

```sh
$ SOLR_1_ADMIN_UI=http://${SOLR_HOST_IP}:${SOLR_1_HOST_PORT}/solr/#/
$ echo ${SOLR_1_ADMIN_UI}
http://192.168.99.100:8984/solr/#/

$ SOLR_2_ADMIN_UI=http://${SOLR_HOST_IP}:${SOLR_2_HOST_PORT}/solr/#/
$ echo ${SOLR_2_ADMIN_UI}
http://192.168.99.100:8985/solr/#/

$ SOLR_3_ADMIN_UI=http://${SOLR_HOST_IP}:${SOLR_3_HOST_PORT}/solr/#/
$ echo ${SOLR_3_ADMIN_UI}
http://192.168.99.100:8986/solr/#/

$ SOLR_4_ADMIN_UI=http://${SOLR_HOST_IP}:${SOLR_4_HOST_PORT}/solr/#/
$ echo ${SOLR_4_ADMIN_UI}
http://192.168.99.100:8987/solr/#/
```