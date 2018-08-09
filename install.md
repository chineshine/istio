## istio 安装

#### 下载
```
  curl -L https://git.io/getLatestIstio | sh -
```
反正没下下来,本作者将地址提取了一下
```
  wget https://github.com/istio/istio/releases/download/1.0.0/istio-1.0.0-linux.tar.gz
```
实在不行就网页下载,再传过去

#### 说明
```
  cd istio-1.0.0
```
目录说明
```
  install/ 用于kubernetes 安装的 .yaml 文件
  bin/ 运行文件  istioctl 客户端,用于手动注入 ENVOY 作为一个 sidecar 代理,并用于创建路由规则和策略
  istio.VERSION 配置文件
  samples/ 一些应用
```

#### 环境变量
```
  export PATH=$PWD/bin:$PATH
```

#### 使用 helm 进行 安装
先安装 helm,[helm 安装手册](https://github.com/chineshine/kubernetes/blob/master/helm/helm-install.md)   
安装 istio 自定义的 api-server
```
  kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
```
如果激活了 `certmanager`
```
  kubectl apply -f install/kubernetes/helm/istio/charts/certmanager/templates/crds.yaml
```
创建命名空间
```
  kubectl create namespace istio-system
```
以下两种方式任选一致  
本作者已经装了 `tiller`,所以选的是 `op2`  
op1:
```
  helm template install/kubernetes/helm/istio --name istio --namespace istio-system > $HOME/istio.yaml
  kubectl create -f $HOME/istio.yaml
```
op2:
```
  helm install install/kubernetes/helm/istio --name istio --namespace istio-system
```
istio 默认使用的 service 类型是 loadBlancer  
如果平台不支持 loadBlancer,请使用 nodePort    
在 `helm` 命令末尾加上
```
  --set gateways.istio-ingressgateway.type=NodePort --set gateways.istio-egressgateway.type=NodePort
```
在装完之后,如果 `istio-ingressgateway` 这个 service 是 pending 状态,则代表集群不支持 loadBlancer  

通信管理最小化设置(官方例子)
```
helm install install/kubernetes/helm/istio --name istio --namespace istio-system \
--set ingress.enabled=false \
--set gateways.istio-ingressgateway.enabled=false \
--set gateways.istio-egressgateway.enabled=false \
--set galley.enabled=false \
--set sidecarInjectorWebhook.enabled=false \
--set mixer.enabled=false \
--set prometheus.enabled=false \
--set global.proxy.envoyStatsd.enabled=false
```
确保 istio-pilot-*  istio-citadel-* 这两个 pod 起来了

#### 卸载
op1:
```
  kubectl delete -f $HOME/istio.yaml
```
op2:
```
  helm delete --purge istio
```
如果 `helm` 版本小于 2.9 ,需要手动删除一些 `job`
```
  kubectl -n istio-system delete job --all
```
删除 `istio` 自定义的 api
```
  kubectl delete -f install/kubernetes/helm/istio/templates/crds.yaml -n istio-system
```
