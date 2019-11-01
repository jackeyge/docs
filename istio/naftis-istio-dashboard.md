# naftis部署，通过istio gateway对外

### 部署

>  https://github.com/XiaoMi/naftis 



```shell
# 下载最新 release 文件和部署清单
wget -O - https://raw.githubusercontent.com/XiaoMi/naftis/master/tool/getlatest.sh | bash

# 创建 Naftis 命名空间
$ kubectl create namespace naftis
# 开启自动注入
kdctl label namespace naftis istio-injection=enabled

# 确认 Naftis 命名空间已创建
$ kubectl get namespace naftis
NAME           STATUS    AGE
naftis         Active    18m

# 部署 Naftis MySQL 服务（本地 Kuberenetes 集群）
$ kubectl apply -n naftis -f mysql.yaml
# 部署 Naftis MySQL 服务（云服务商提供的 Kuberenetes 集群）
$ kubectl apply -n naftis -f mysql-cloud.yaml

# 确认 MySQL 已部署
NAME                           READY     STATUS    RESTARTS   AGE
naftis-mysql-c78f99d6c-kblbq   0/1       Running   0          9s
naftis-mysql-test              1/1       Running   0          10s
#修改naftis.yaml
# Source: naftis/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: naftis-api
  namespace: naftis
  labels:
    heritage: Tiller
    chart: "naftis-0.1.1"
    app: naftis-api
    release: naftis
spec:
  selector:
    app: naftis-api
    release: naftis
  ports:
    - port: 50000
      #protocol: TCP
      name: http  #添加name
---
apiVersion: v1
kind: Service
metadata:
  name: naftis-ui
  namespace: naftis
  labels:
    heritage: Tiller
    chart: "naftis-0.1.1"
    app: naftis-ui
    release: naftis
spec:
  #type: LoadBalancer
  type: NodePort #修改type
  selector:
    app: naftis-ui
    release: naftis
  ports:
    - port: 80
      #protocol: TCP
      name: http  #添加name
---


# 部署 Naftis API 和 UI 服务
kubectl apply -n naftis -f naftis.yaml

# 确认 Naftis 所有的服务已经正确定义并正常运行中
kubectl get svc -n naftis
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
naftis-api     ClusterIP      10.233.3.144    <none>        50000/TCP      7s
naftis-mysql   ClusterIP      10.233.57.230   <none>        3306/TCP       55s
naftis-ui      LoadBalancer   10.233.18.125   <pending>     80:31286/TCP   6s

kubectl get pod -n naftis
NAME                           READY     STATUS    RESTARTS   AGE
naftis-api-0                   1/2       Running   0          19s
naftis-mysql-c78f99d6c-kblbq   1/1       Running   0          1m
naftis-mysql-test              1/1       Running   0          1m
naftis-ui-69f7d75f47-4jzwz     1/1       Running   0          19s

# 配置gateway
$ cat naftis-ui.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: naftis-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "naftis.xxx.com"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: naftis-ui
spec:
  hosts:
  - "naftis.xxx.com"
  gateways:
  - naftis-gateway
  http:
   - route:
     - destination:
         host: naftis-ui
         port:
           number: 80

$ kubectl apply -n naftis -f naftis-ui.yaml 

# 添加hosts解析

# 打开浏览器，访问 http://naftis.xxx.com 即可。默认用户名和密码分别为 admin、admin。
```