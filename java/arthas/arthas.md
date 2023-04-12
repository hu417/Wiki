++++++ arthas ++++++

#java需要有jps命令
yum install -y java-1.8.0-openjdk-devel java-1.8.0-openjdk zip unzip
yum -y install java-11-openjdk-devel java-11-openjdk zip unzip

----- 单机版
// 在线安装

curl -O https://arthas.aliyun.com/arthas-boot.jar --output arthas-boot.jar

#启动arthas   注意：启动前已经要有java进程运行，否则无法进入

java -jar arthas-boot.jar --telnet-port 9998 --http-port -1
// 启动过程中会下载相关包到本地: /root/.arthas/目录下


----- 分布式
// 全版本
// 离线安装
wget https://github.com/alibaba/arthas/releases/download/arthas-all-3.6.7/arthas-bin.zip
unzip arthas-bin.zip -d arthas
cd arthas
./install-local.sh 
ls ~/.arthas/

// 服务端
wget https://github.com/alibaba/arthas/releases/download/arthas-all-3.6.7/arthas-tunnel-server-3.6.7-fatjar.jar
java -Xms256m -Xmx256m -Xmn512m -jar -Dserver.port=8081 arthas-tunnel-server-3.6.7-fatjar.jar  

// 客户端: ~/.arthas/
java -jar arthas-boot.jar --tunnel-server 'ws://10.0.0.91:7777/ws' --agent-id 10.0.0.91

#注意: 在线安装与离线安装不能使用同一个依赖(~/.arthas/), arthas-boot.jar启动会报错

----- 常用命令
参考: https://blog.csdn.net/qq_48721706/article/details/126696465

----- 停止服务

java -jar arthas-client.jar 127.0.0.1 9998 -c "stop"
或者界面连接执行: stop

----- 镜像制作
wget https://github.com/alibaba/arthas/releases/download/arthas-all-3.6.7/arthas-bin.zip

// jdk8
FROM openjdk:8-jdk-alpine 

ENV TZ=Asia/Shanghai \
    LANG=zh_CN.UTF-8 \
    JAVA_OPTS="-Xms128m -Xmx256m -Djava.security.egd=file:/dev/./urandom"

ARG JAR_FILE=halo.jar

WORKDIR /opt
COPY target/${JAR_FILE} app.jar
COPY ./arthas-bin.zip /tmp/

#修改源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories \
    && set -xe \
    && apk --no-cache add ttf-dejavu fontconfig tzdata curl tini && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    mkdir /usr/local/arthas && \
    unzip /tmp/arthas.zip -d /usr/local/arthas && \
    rm -rf /tmp/arthas.zip

EXPOSE 8080
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["sh", "-ec", "exec java ${JAVA_OPTS} -jar /opt/app.jar"]


// jdk11
FROM openjdk:11-jdk-slim

ENV TZ=Asia/Shanghai \
    LANG=zh_CN.UTF-8 \
    JAVA_OPTS="-Xms128m -Xmx256m -Djava.security.egd=file:/dev/./urandom"

ARG JAR_FILE=halo.jar

WORKDIR /opt
COPY target/${JAR_FILE} app.jar
COPY ./arthas-bin.zip /tmp/

#修改源
RUN sed -i 's#http://deb.debian.org#https://mirrors.ustc.edu.cn#g' /etc/apt/sources.list \
    && sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list \
    && set -xe \
    && apt-get -y update && apt-get upgrade -y \
    && apt-get install curl tini unzip procps -y \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && mkdir /usr/local/arthas \
    && unzip /tmp/arthas.zip -d /usr/local/arthas \
    && chmod +x /usr/local/arthas/*.jar \
    && rm -rf /tmp/arthas.zip

EXPOSE 8080
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["sh", "-ec", "exec java ${JAVA_OPTS} -jar /opt/app.jar"]
