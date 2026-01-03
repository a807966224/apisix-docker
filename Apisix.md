

1. 应用场景
   a. 南北、东西流量
2. 架构&同类产品对比
3. 技术底层&优势
   a. 内存更新配置, ng服务不在重启
   b. 多语言插件
   c. 路由匹配规则效率
4. 核心组件
   a. route
   b. upstream
   c. Admin api
5. 示例
   a. 流量分发
   b. 限流策略
   c. 安全保护
6. 生态支撑
   a. 服务发现
   b. 日志
   c. 监控


镜像加速
https://github.com/tech-shrimp/docker_installer

JDK镜像
https://docker.aityp.com/image/docker.io/openjdk:21

JVMS版本管理
https://jishuzhan.net/article/2002307339295195137

https://github.com/apache/apisix-java-plugin-runner.git
./mvnw package -DskipTests

https://nacos.io/docs/latest/quickstart/quick-start-docker/?spm=5238cd80.2ef5001f.0.0.3f613b7ctknj1j
启动nacos
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa

NACOS_AUTH_TOKEN=YWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFh
NACOS_AUTH_IDENTITY_KEY=nacos_key
NACOS_AUTH_IDENTITY_VALUE=nacos_value


docker run --name nacos-standalone-derby \
-e MODE=standalone \
-e NACOS_AUTH_TOKEN=YWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFh \
-e NACOS_AUTH_IDENTITY_KEY=nacos_key \
-e NACOS_AUTH_IDENTITY_VALUE=nacos_value \
-p 8080:8080 \
-p 8848:8848 \
-p 9848:9848 \
-d nacos/nacos-server:latest




curl --proxy http://127.0.0.1:7897 -LO https://dl.k8s.io/release/v1.30.0/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

kubectl version --client

curl --proxy http://192.168.0.109:7897 https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version


curl --proxy http://192.168.0.109:7897 -Lo kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/


kind create cluster --name demo
kubectl get nodes


helm repo add apisix https://charts.apiseven.com
helm repo update




FROM apache/apisix:3.14.1-debian


USER root
RUN apt-get update && \
apt-get install -y wget tar && \
wget https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz && \
tar -xzf jdk-21_linux-x64_bin.tar.gz -C /opt && \
ln -s /opt/jdk-21* /opt/jdk-21

ENV JAVA_HOME=/opt/jdk-21
ENV PATH=$JAVA_HOME/bin:$PATH

ADD apache-apisix-java-plugin-runner-0.6.0-bin.tar.gz /usr/local/

docker build -t apache/apisix:3.14.1-alpine-with-java-plugin .

