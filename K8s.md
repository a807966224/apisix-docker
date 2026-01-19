




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
kind create cluster --name k8s-lab --config kind-config.yaml
Creating cluster "k8s-lab" ...
• Ensuring node image (kindest/node:v1.35.0) 🖼  ...
PS C:\project\mine\apisix-docker\k8s> kind create cluster --name k8s-lab --config kind-config.yaml
Creating cluster "k8s-lab" ...
• Ensuring node image (kindest/node:v1.35.0) 🖼  ...
✓ Ensuring node image (kindest/node:v1.35.0) 🖼
• Preparing nodes 📦 📦 📦   ...
✓ Preparing nodes 📦 📦 📦
• Writing configuration 📜  ...
✓ Writing configuration 📜
• Starting control-plane 🕹️  ...
✓ Starting control-plane 🕹️
• Installing CNI 🔌  ...
✓ Installing CNI 🔌
• Installing StorageClass 💾  ...
✓ Installing StorageClass 💾
• Joining worker nodes 🚜  ...
✓ Joining worker nodes 🚜
Set kubectl context to "kind-k8s-lab"
You can now use your cluster with:

kubectl cluster-info --context kind-k8s-lab

Thanks for using kind! 😊


验证集群
kubectl get nodes

k8s可视化UI
https://www.rancher.cn/quick-start/

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

这个步骤在windows中一直无法启动成功，一定要在wsl中执行才可以
kubectl -n kube-system edit deployment metrics-server
在 args: 中加：
- --kubelet-insecure-tls

验证Pod是否正常
kubectl get pods -n kube-system
NAME                                            READY   STATUS    RESTARTS        AGE
coredns-7db6d8ff4d-jgn46                        1/1     Running   0               9m59s
coredns-7db6d8ff4d-q5xwf                        1/1     Running   0               9m59s
etcd-k8s-lab-control-plane                      1/1     Running   0               10m
kindnet-mw4md                                   1/1     Running   0               9m53s
kindnet-qf2q9                                   1/1     Running   0               9m54s
kindnet-tkv4h                                   1/1     Running   0               9m59s
kube-apiserver-k8s-lab-control-plane            1/1     Running   0               10m
kube-controller-manager-k8s-lab-control-plane   1/1     Running   1 (9m10s ago)   10m
kube-proxy-85bkp                                1/1     Running   0               9m54s
kube-proxy-tsgd7                                1/1     Running   0               9m53s
kube-proxy-wxcpt                                1/1     Running   0               9m59s
kube-scheduler-k8s-lab-control-plane            1/1     Running   1 (9m6s ago)    10m
metrics-server-7fbfbcc44c-cp7k6                 1/1     Running   0               2m16s

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
kubectl delete pod -l app=demo-app
kubectl get pod -w


验证应用是否正常的方式
方式一：端口转发
kubectl port-forward svc/demo-app 8080:80

方式二：集群内curl
kubectl run curl --image=curlimages/curl -it --rm -- sh
curl http://demo-app






这一段一直卡住，我没有在等，直接上手apisix了
先创建ingress-nginx-controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

等待启动陈工
kubectl -n ingress-nginx get pod -w


直接上apisix ingress controller
kubectl create namespace ingress-apisix

helm repo add apisix https://apache.github.io/apisix-helm-chart
helm repo update

helm install apisix \
--namespace ingress-apisix \
--create-namespace \
--set apisix.deployment.role=traditional \
--set apisix.deployment.role_traditional.config_provider=yaml \
--set etcd.enabled=false \
--set ingress-controller.enabled=true \
--set ingress-controller.config.provider.type=apisix-standalone \
--set ingress-controller.apisix.adminService.namespace=ingress-apisix \
--set ingress-controller.gatewayProxy.createDefault=true \
apisix/apisix


检查命名空间下的所有资源
kubectl -n ingress-apisix get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/apisix-6c77c9f5d4-p4bpt                     1/1     Running   0          3m48s
pod/apisix-ingress-controller-94c9d7f88-5lm2h   2/2     Running   0          3m48s

NAME                                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/apisix-admin                            ClusterIP   10.96.170.240   <none>        9180/TCP       3m48s
service/apisix-gateway                          NodePort    10.96.119.135   <none>        80:30551/TCP   3m48s
service/apisix-ingress-controller               ClusterIP   10.96.7.197     <none>        8080/TCP       3m48s
service/apisix-ingress-controller-webhook-svc   ClusterIP   10.96.42.174    <none>        443/TCP        3m48s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/apisix                      1/1     1            1           3m48s
deployment.apps/apisix-ingress-controller   1/1     1            1           3m48s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/apisix-6c77c9f5d4                     1         1         1       3m48s
replicaset.apps/apisix-ingress-controller-94c9d7f88   1         1         1       3m48s

kubectl -n ingress-apisix get pods
NAME                                        READY   STATUS    RESTARTS   AGE
apisix-6c77c9f5d4-p4bpt                     1/1     Running   0          3m44s
apisix-ingress-controller-94c9d7f88-5lm2h   2/2     Running   0          3m44s


验证启动是否成功
kubectl port-forward svc/apisix-gateway 9080:80 &

curl -sI "http://127.0.0.1:9080" | grep Server
Handling connection for 9080
Server: APISIX/3.14.1


创建路由
kubectl -n ingress-apisix apply -f httpbin-route.yaml
apisixroute.apisix.apache.org/getting-started-ip created

┌──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ kubectl get pods -w
NAME                        READY   STATUS    RESTARTS      AGE
demo-app-788fb7b895-s752h   1/1     Running   1 (61m ago)   13h
^C
┌──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ kubectl -n ingress-apisix get pods -w
NAME                                        READY   STATUS    RESTARTS   AGE
apisix-6c77c9f5d4-p4bpt                     1/1     Running   0          51m
apisix-ingress-controller-94c9d7f88-5lm2h   2/2     Running   0          51m
httpbin-deployment-74bfc85c95-d8zd7         1/1     Running   0          21m

┌──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ kubectl -n ingress-apisix get pods -w
NAME                                        READY   STATUS    RESTARTS   AGE
apisix-6c77c9f5d4-p4bpt                     1/1     Running   0          51m
apisix-ingress-controller-94c9d7f88-5lm2h   2/2     Running   0          51m
httpbin-deployment-74bfc85c95-d8zd7         1/1     Running   0          21m
^C
┌──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ kubectl -n ingress-apisix port-forward svc/apisix-gateway 9080:80
Unable to listen on port 9080: Listeners failed to create with the following errors: [unable to create listener: Error listen tcp4 127.0.0.1:9080: bind: address already in use unable to create listener: Error listen tcp6 [::1]:9080: bind: address already in use]
error: unable to listen on any of the requested ports: [{9080 9080}]

┌──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ sudo netstat -tulpn | grep :9080
[sudo] password for scott:
tcp        0      0 127.0.0.1:9080          0.0.0.0:*               LISTEN      1068/kubectl
tcp6       0      0 ::1:9080                :::*                    LISTEN      1068/kubectl

┌──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ kill -15 1068

[1]+  Terminated                 kubectl -n ingress-apisix port-forward svc/apisix-gateway 9080:80  (wd: /mnt/c/Users/zxy80)
(wd now: /mnt/c/project/mine/apisix-docker/k8s)
┌──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ sudo netstat -tulpn | grep :9080

┌──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ kubectl -n ingress-apisix port-forward svc/apisix-gateway 9080:80
Forwarding from 127.0.0.1:9080 -> 9080
Forwarding from [::1]:9080 -> 9080
Handling connection for 9080
Handling connection for 9080
Handling connection for 9080
Handling connection for 9080


┌──(scott㉿X688)-[/mnt/c/Users/zxy80]
└─$ curl "http://127.0.0.1:9080/ip"
{
"origin": "127.0.0.1"
}


创建负载
──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ kubectl -n ingress-apisix apply -f lb-route.yaml
apisixupstream.apisix.apache.org/httpbin-external-domain created
apisixupstream.apisix.apache.org/mockapi7-external-domain created
apisixroute.apisix.apache.org/lb-route created

┌──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ kubectl port-forward svc/apisix-gateway 9080:80
Error from server (NotFound): services "apisix-gateway" not found

┌──(scott㉿X688)-[/mnt/c/project/mine/apisix-docker/k8s]
└─$ kubectl -n ingress-apisix port-forward svc/apisix-gateway 9080:80
Forwarding from 127.0.0.1:9080 -> 9080
Forwarding from [::1]:9080 -> 9080


┌──(scott㉿X688)-[/mnt/c/Users/zxy80]
└─$ resp=$(seq 50 | xargs -I{} curl "http://127.0.0.1:9080/headers" -sL) && \
>   count_httpbin=$(echo "$resp" | grep "httpbin.org" | wc -l) && \
count_>   count_mockapi7=$(echo "$resp" | grep "mock.api7.ai" | wc -l) && \
>   echo httpbin.org: $count_httpbin, mock.api7.ai: $count_mockapi7
httpbin.org: 25, mock.api7.ai: 25


暴露service到外部
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext