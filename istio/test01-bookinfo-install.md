# istio实验01--bookinfo 安装

### 服务架构

- 服务自身架构

![](images/test01-noistio.svg)

- 加入istio

![](images/test01-withistio.svg)

 所有微服务都将与Envoy打包在一起，该Envoy边车拦截对服务的呼入和呼出，并通过Istio控制平面，路由，遥测收集和整个应用程序的策略实施提供外部控制所需的钩子。 



### 部署

1. 部署文件

进入istio安装包根目录

2. k8s defaute namespace 开启自动 Sidecar  注入

   ```shell
   $ kubectl label namespace default istio-injection=enabled
   ```

3. 安装

   ```shell
   $ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
   ```

4. 检查pod状态

   ```shell
   $ kubectl get services
   NAME                       CLUSTER-IP   EXTERNAL-IP   PORT(S)              AGE
   details                    10.0.0.31    <none>        9080/TCP             6m
   kubernetes                 10.0.0.1     <none>        443/TCP              7d
   productpage                10.0.0.120   <none>        9080/TCP             6m
   ratings                    10.0.0.15    <none>        9080/TCP             6m
   reviews                    10.0.0.170   <none>        9080/TCP             6m
   
   $ kubectl get pods
   NAME                                        READY     STATUS    RESTARTS   AGE
   details-v1-1520924117-48z17                 2/2       Running   0          6m
   productpage-v1-560495357-jk1lz              2/2       Running   0          6m
   ratings-v1-734492171-rnr5l                  2/2       Running   0          6m
   reviews-v1-874083890-f0qf0                  2/2       Running   0          6m
   reviews-v2-1343845940-b34q5                 2/2       Running   0          6m
   reviews-v3-1813607990-8ch52                 2/2       Running   0          6m
   ```



5. 验证服务运行状态

   ```shell
   $ kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
   <title>Simple Bookstore App</title>
   ```

6. 创建入口网关

   ```shell
   kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
   
   
   $ kubectl get gateway
   NAME               AGE
   bookinfo-gateway   32s
   ```

   

7. 测试外部访问

    ```shell
   # 查看公网IP xxx.xxx.xxx.xxx
   [root@tc-bj-test-node01 ~]# kubectl get svc istio-ingressgateway -n istio-system
   NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                                                                                                                      AGE
   istio-ingressgateway   LoadBalancer   172.18.255.150   xxx.xxx.xxx.xxx   15020:31873/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:30341/TCP,15030:30803/TCP,15031:31946/TCP,15032:30574/TCP,15443:30209/TCP   45m
   #访问
   [root@tc-bj-test-node01 ~]# curl -s xxx.xxx.xxx.xxx/productpage | grep -o "<title>.*</title>"
   <title>Simple Bookstore App</title>
   
    ```

8. 应用默认规则

   ```shell
   #未启用双向TLS
   $ kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
   #启用了双向TLS
   $ kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml
   #查看
   $ kubectl get destinationrules -o yaml
   ```

   

### 清理

```shell
#删除路由规则并终止应用程序容器

$ samples/bookinfo/platform/kube/cleanup.sh

#确认关机

$ kubectl get virtualservices   #-- there should be no virtual services
$ kubectl get destinationrules  #-- there should be no destination rules
$ kubectl get gateway           #-- there should be no gateway
$ kubectl get pods              #-- the Bookinfo pods should be deleted
```

