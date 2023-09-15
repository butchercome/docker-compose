pwd
COMPOSE_PROJECT_NAME=zookeeper_cluster docker-compose -f zookeeper.yml  up -d

docker exec -it zookeeper_cluster-zoo1-1 /bin/bash

docker exec -it zookeeper_cluster-zoo2-1 /bin/bash

./zkCli.sh -server localhost:2181,localhost:2182,localhost:2183

./zkCli.sh -server localhost:2181,localhost:2182,localhost:2183