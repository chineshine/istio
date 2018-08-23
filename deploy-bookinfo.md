## 部署
### 要求
协议要求:部署的应用使用的 Http 协议必须是
```
  HTTP/1.1
  HTTP/2.0
  HTTP/1.0(不支持)
```

如果安装了 `Istio-sidecar-injector` ,可以直接使用 `kubectl apply` 命令部署应用    
`Istio-sidecar-injector` 会自动注入 `Envoy` 容器进入部署的应用 pods 中  
同时要求部署的应用的命名空间必须激活标签 `istio-injection=enabled`   
```
  kubectl label namespace <namespace> istio-injection=enabled
```
部署应用
```
  kubectl create -n <namespace> -f <your-app>.yaml
```

如果没有安装 `Istio-sidecar-injector`  
需要在部署应用之前使用命令 `istioctl kube-inject` 手动注入 'Envoy' 容器  
```
  istioctl kube-inject -f <your-app>.yaml | kubectl apply -f -
```

### 部署应用
官方例子:`bookinfo`
```
  cd /../istio-1.0.0
```
本例命名空间使用的是 `istio-test`
```
  kubectl create namespace istio-test
```

op1:如果使用手动的 `sidecar` 注入
```
  kubectl apply -f <(istioctl kube-inject -f @samples/bookinfo/platform/kube/bookinfo.yaml@)
```
op2: 如果集群内已经支持`Istio-sidecar-injector`
```
  kubectl label namespace istio-test istio-injection=enabled
```
部署
```
  kubectl -n istio-test apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```
上述 op1 或 op2 都会启动四个微服务  
包括 reviews 服务的 v1,v2,v3 所有三个版本  
实际部署应随着时间推移部署新版本,而不是一下部署所有版本  

安装 `istio-gateway` 用于外部访问
```
  kubectl -n istio-test apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```
确认网关已部署
```
  kubectl -n istio-test get gateway
```
