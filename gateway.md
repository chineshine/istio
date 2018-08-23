##istio 网关
### 说明
kubernetes 采用 ingress 将服务暴露到集群外  
istio 采用了一个更好的方式--使用了一个不同的配置模型--也就是网关  
可同时运行在 kubernetes 环境和其他环境中  
包含了 监控 和 路由规则 等特性  

### 开始
确保已经 [安装 istio](install.md)  
进入 `istio` 目录
```
  cd /.../cd istio-1.0.0
```
#### 准备安装 `httpbin`  
本次使用命名空间为 `istio-test`
```
  kubectl create namespace istio-test
```
op1: `sidecar` 自动档
```
  kubectl -n istio-test apply -f samples/httpbin/httpbin.yaml
```
op2: `sidecar` 手动档
```
  kubectl -n istio-test apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml)
```

#### 定夺 `ingress` 的 ip 和 端口
```
  kubectl get svc istio-ingressgateway -n istio-system
```
查看 `EXTERNAL-IP` 这列是不是有值  
① 具体的 ip 地址,说明支持 `loadBlancer`,请根据下面 loadBlancer 继续配置  
② `none` 或 `pending`,不支持 `loadBlancer`,请使用 `nodePort` 设置重新安装,然后根据下面的 `nodePort` 接着配置  

#### `loadBlancer` 配置
设置 `ingress` 的 `IP` 和 `ports`
```
  export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```
```
  export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
```
```
  export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
```
注意: 某些环境中,`loadBalancer` 暴露出的是一个主机名称(hostname),而不是一个 IP地址,所设置的环境变量 `INGRESS_HOST ` 会失败, 使用下面的命令进行正确的设置:
```
  export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

####  `nodePort` 配置
设置端口
```
  export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
```
```
  export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
```
根据集群供应商设置 `ingress ip`
1) GKE
```
  export INGRESS_HOST=<workerNodeAddress>
```
为了 `ingressgateway` 服务端口能够通过 TCP 通信  
需要创建防火墙规则(firewall rules)  
```
   gcloud compute firewall-rules create allow-gateway-http --allow tcp:$INGRESS_PORT
```
```
  gcloud compute firewall-rules create allow-gateway-https --allow tcp:$SECURE_INGRESS_PORT
```
2) IBM Cloud Kubernetes Service Free Tier
```
  bx cs workers <cluster-name or id>
```
```
  export INGRESS_HOST=<public IP of one of the worker nodes>
```
3) Minikube
```
  export INGRESS_HOST=$(minikube ip)
```
4) 其他环境(IBM 私有云等)
```
  export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o 'jsonpath={.items[0].status.hostIP}')
```
