docker-compose -f kafka.yml  up -d




kafka-console-producer --broker-list localhost:9092 --topic testtopic


kafka-topics  --create  --replication-factor 2 --partitions 3 --topic cjw-test --bootstrap-server localhost:9092


kafka-topics  --create  --replication-factor 4 --partitions 3 --topic cjw-test5 --bootstrap-server localhost:9092

#端口号要看通讯的端口号
kafka-topics  --create  --replication-factor 4 --partitions 3 --topic cjw-test5 --bootstrap-server localhost:19092

kafka-topics --list --bootstrap-server localhost:9092

kafka-topics --describe cjw-test1 --bootstrap-server localhost:9092

#查看址发布到zookeeper供客户端使用的地址，即通讯端口号

get /brokers/ids/1