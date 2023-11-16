# 如果需要构建Linux amd64的包可以用如下指令 buildx
docker buildx build --platform=linux/amd64 -t [image-name] .
docker inspect [image-name] | grep Arch

# 或者在from 时指定  如：FROM --platform=linux/amd64 alpine:3.18.0  指定平台

--------------------



# docker build --rm -t <username>/ssh:centos7 .





docker run -itd -p 22 <username>/ssh:centos7
# docker run -itd --name centos7_1 -p 10000:22 cjw/ssh:centos7 /bin/bash


--------------------
## dockerfile 2

. 代表当前目录
# docker build --rm -t cjw/sshcentos:centos7 -f Dockerfile2 .
# docker run -itd --name centos7_2 -p 10000:22 cjw/sshcentos:centos7
# /usr/sbin/sshd -D &
# lsof -i:22
# passwd root
