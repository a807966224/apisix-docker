# APISIX-Starter

## 简介
    1. 快速搭建APISIX环境
    2. 如何集成K8S环境
    3. 如何集成Jenkisn环境
    4. 快速搭建A和B测试应用
    5. A和B服务的部署
    6. A和B服务之间通过APISIX网关调用
    7. A和B服务之间实现平滑扩缩容
    8. A和B服务之间实现平滑上下线


## 快速搭建APISIX环境
    1. git clone https://github.com/apache/apisix-docker.git
    2. cd apisix-docker/example
    3. docker-compose up -d
    4. 启动服务成功后，其中包含了nginx的两个测试服务，在宿主机器中访问
        简单测试下 curl http://localhost:9082/ 应返回 hello web2
        简单测试下 curl http://localhost:9081/ 应返回 hello web1
        如果上边没有通过，可以尝试进入容器内进行测试 
            docker exec -it -u root ${apisix_continer_id} bash
            apt install curl
            curl http://example-web1-1/ 应返回 hello web1
    5. 访问APISIX控制台 http://localhost:9180/ui
    6. 创建APISIX路由
        1. 在控制台中最小配置法创建路由，先名称
        2. 再配置访问地址
        3. 最后配置服务名称 + 端口 + 权重
        4. 保存后，验证路由是否正常，访问 curl http://localhost:9080/ng 按照权重返回相应内容
    7. 至此，测试环境搭建完成，至于复杂的配置，请参考官方文档，如果有额外需要可以联系我

## 如何集成K8S环境
    1. 创建K8S集群
        安装 kubectl（Windows）
        winget install Kubernetes.kubectl
        kubectl version --client
        
        安装 kind
        winget install Kubernetes.kind
        kind version
        
        安装 Helm（插件必备）
        winget install Helm.Helm
        helm version

        在 Windows 上创建 kind 集群
        创建集群配置文件 kind-config.yaml
        ````
            kind: Cluster
            apiVersion: kind.x-k8s.io/v1alpha4
            nodes:
            - role: control-plane
              extraPortMappings:
                - containerPort: 80
                  hostPort: 80
                  protocol: TCP
                - containerPort: 443
                  hostPort: 443
                  protocol: TCP
            - role: worker
            - role: worker
          ````
        创建集群：kind create cluster --name k8s-lab --config kind-config.yaml
        创建完成后，验证： kind get clusters
                        kubectl -n k8s-lab get nodes
        集群创建成功后，进入集群： kind export kubeconfig --name k8s-lab
            配置kubectl连接：将创建的kind集群的连接配置导出到本地的kubectl配置文件中
            建立认证信息：让kubectl能够识别并安全地连接到k8s-lab集群
            设置访问凭证：包含集群的API服务器地址、证书和认证令牌等必要信息
            将集群配置写入到 $HOME/.kube/config 文件中
            使得 kubectl 命令能够与kind集群通信
            不需要每次都指定 --kubeconfig 参数
        集群创建成功后，进入集群： kubectl config use-context kind-k8s-lab
            设置当前kubectl使用的上下文为目标集群
            确保后续的kubectl命令都发送到正确的集群
        
        部署 metrics-server（Windows 专用注意点）
            kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
            serviceaccount/metrics-server created
            clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
            clusterrole.rbac.authorization.k8s.io/system:metrics-server created
            rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
            clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
            clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
            service/metrics-server created
            deployment.apps/metrics-server created
            apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
            
            这个步骤在windows中一直无法启动成功，一定要在wsl中执行才可以，其实这里就是起了另外一个pod来监控节点状态
            kubectl -n kube-system edit deployment metrics-server
            在 args: 中加：
            - --kubelet-insecure-tls

            ````
            kubectl top nodes
            NAME                    CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
            k8s-lab-control-plane   94m          0%       680Mi           4%
            k8s-lab-worker          19m          0%       469Mi           2%
            k8s-lab-worker2         141m         0%       284Mi           1%
            ````

        查看日志
        kubectl -n kube-system logs metrics-server-7fbfbcc44c-cp7k6
        
        部署一个测试应用
        kubectl create deployment demo-app --image=nginx
        kubectl expose deployment demo-app --port=80 --type=ClusterIP
        
        
        验证
        kubectl get pod,svc
        NAME                            READY   STATUS    RESTARTS   AGE
        pod/demo-app-788fb7b895-ggbw7   1/1     Running   0          97s
        
        NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
        service/demo-app     ClusterIP   10.96.43.188   <none>        80/TCP    64s
        service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   13m
        
        删除应用查看是否会自动重建
        kubectl -n ingress-apisix delete pod -l app=demo-app
        kubectl -n ingress-apisix get pod -w
        
        验证应用是否正常的方式
        方式一：端口转发
        kubectl port-forward svc/demo-app 8080:80
        
        方式二：集群内curl
        kubectl run curl --image=curlimages/curl -it --rm -- sh
        curl http://demo-app


## 快速集成Jenkins环境
    


````#selectedCode
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
````

        




