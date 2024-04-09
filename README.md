- 通过docker-compose  快速构建，项目所需要的一些组件，测试研发
- zookeeper
- redis
- elasticsearch
- kafka
- prometheus+grafana




- 查看所有容器
docker ps -a -q
docker update --restart=on-failure $(docker ps -a -q)
- 关闭容器自启动避免docker进程启动时容器全部启动
docker update --restart=no $(docker ps -a -q)