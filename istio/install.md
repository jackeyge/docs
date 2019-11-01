# istio 部署 - helm-k8s

### 下载相应版本包

 https://github.com/istio/istio/releases 

### 安装选项

1. **default**：根据默认的[安装选项](https://preliminary.istio.io/docs/reference/config/installation-options/) （建议用于生产部署）启用组件。
2. **演示**：旨在展示Istio功能且资源需求适中的配置。适合运行[Bookinfo](https://preliminary.istio.io/docs/examples/bookinfo/)应用程序和相关任务。该配置与[快速入门](https://preliminary.istio.io/docs/setup/install/kubernetes/)说明中安装的配置相同，但是使用Helm，如果您以后希望探索更高级的任务，则可以更轻松地启用其他功能。
3. **minimal**：使用Istio的[流量管理](https://preliminary.istio.io/docs/tasks/traffic-management/)功能所需的最少组件集。
4. **sds-auth**：类似于**默认**配置文件，但也启用了Istio的[SDS（秘密发现服务）](https://preliminary.istio.io/docs/tasks/security/auth-sds)。此配置文件附带默认情况下启用的其他身份验证功能（严格双向TLS）。

标记为**X**的组件安装在每个配置文件中：

|                          | 默认          | 演示                     | 最小的                      | sds                          |
| ------------------------ | ------------- | ------------------------ | --------------------------- | ---------------------------- |
| 个人资料档案名称         | `values.yaml` | `values-istio-demo.yaml` | `values-istio-minimal.yaml` | `values-istio-sds-auth.yaml` |
| 核心组成                 |               |                          |                             |                              |
| `istio-citadel`          | X             | X                        |                             | X                            |
| `istio-egressgateway`    |               | X                        |                             |                              |
| `istio-galley`           | X             | X                        |                             | X                            |
| `istio-ingressgateway`   | X             | X                        |                             | X                            |
| `istio-nodeagent`        |               |                          |                             | X                            |
| `istio-pilot`            | X             | X                        | X                           | X                            |
| `istio-policy`           | X             | X                        |                             | X                            |
| `istio-sidecar-injector` | X             | X                        |                             | X                            |
| `istio-telemetry`        | X             | X                        |                             | X                            |
| 插件                     |               |                          |                             |                              |
| `grafana`                |               | X                        |                             |                              |
| `istio-tracing`          |               | X                        |                             |                              |
| `kiali`                  |               | X                        |                             |                              |
| `prometheus`             | X             | X                        |                             | X                            |

要进一步自定义Istio和安装插件，可以在安装Istio时使用`--set =`的`helm template`or `helm install`命令中添加一个或多个选项。在[安装选项](https://preliminary.istio.io/docs/reference/config/installation-options/)列出了一整套支持安装密钥和值对。

本次 部署选择 values-istio-demo.yaml 



### 使用 helm template 安装

1. 为`istio-system`组件创建一个名称空间：

   ```shell
   $ kubectl create namespace istio-system
   ```

   

2. 使用以下命令安装所有Istio [自定义资源定义](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions) （CRD）`kubectl apply`：

   ```shell
   $ helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -
   ```

   

3. 等待所有Istio CRD创建：

   ```shell
   $ kubectl -n istio-system wait --for=condition=complete job --all
   ```

![](images\install-1.png)

4.选择一个配置文件部署:

```shell
#defaute
$ helm template install/kubernetes/helm/istio --name istio --namespace istio-system | kubectl apply -f -
#demo(本次部署使用)
$ helm template install/kubernetes/helm/istio --name istio --namespace istio-system \
    --values install/kubernetes/helm/istio/values-istio-demo.yaml | kubectl apply -f -
#minimal
$ helm template install/kubernetes/helm/istio --name istio --namespace istio-system \
    --values install/kubernetes/helm/istio/values-istio-minimal.yaml | kubectl apply -f -
#sds
$ helm template install/kubernetes/helm/istio --name istio --namespace istio-system \
    --values install/kubernetes/helm/istio/values-istio-sds-auth.yaml | kubectl apply -f -

```

![](images\install-2.png)





###  卸载

```shell
$ helm template install/kubernetes/helm/istio --name istio --namespace istio-system \
    --values install/kubernetes/helm/istio/values-istio-demo.yaml | kubectl delete -f -
$ kubectl delete namespace istio-system
```



[官网]: https://preliminary.istio.io/docs/setup/install/helm/

