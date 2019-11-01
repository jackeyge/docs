#   实验03 故障注入

## 准备工作

### 进入工作目录

`istio-1.4.0-beta.0/samples/bookinfo/networking`

### 初始化路由配置

```shell
kubectl apply -f virtual-service-all-v1.yaml
kubectl apply -f virtual-service-reviews-test-v2.yaml
```

通过上面的配置，下面是请求的流程：

- `productpage` → `reviews:v2` → `ratings` (`jason` 用户)
- `productpage` → `reviews:v1` (其他用户)

## 故障注入测试

### 01 使用 HTTP 延迟进行故障注入

为了测试微服务应用程序 Bookinfo 的弹性，我们将为用户 `jason` 在 `reviews:v2` 和 `ratings` 服务之间注入一个 7 秒的延迟。 这个测试将会发现故意引入 Bookinfo 应用程序中的错误。

由于 `reviews:v2` 服务对其 ratings 服务的调用具有 10 秒的硬编码连接超时，比我们设置的 7s 延迟要大，因此我们期望端到端流程是正常的（没有任何错误）。

####  创建故障注入规则以延迟来自用户 `jason`的流量  

`kubectl apply -f virtual-service-ratings-test-delay.yaml`

*kubectl apply -f virtual-service-ratings-test-delay.yaml*

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:
      delay:
        percentage:
          value: 100.0
        fixedDelay: 7s
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

#### 测试

1. 通过浏览器打开 [Bookinfo](https://archive.istio.io/v1.2/zh/docs/examples/bookinfo) 应用。

2. 使用用户 `jason` 登陆到 `/productpage` 界面。

   你期望 Bookinfo 主页在大约 7 秒钟加载完成并且没有错误。但是，出现了一个问题，Reviews 部分显示了错误消息：

   ```plain
   Error fetching product reviews!
   Sorry, product reviews are currently unavailable for this book.
   ```

   

3. 查看页面的返回时间：

   1. 打开浏览器的 *开发工具* 菜单
   2. 打开 *网络* 标签
   3. 重新加载 `productpage` 页面，你会看到页面实际上用了大约 6s。

#### 原理

你发现了一个 bug。在微服务中有硬编码超时，导致 `reviews` 服务失败。

在 `productpage` 和 `reviews` 服务之间超时时间是 6s - 编码 3s + 1 次重试总共 6s ，`reviews` 和 `ratings` 服务之间的硬编码连接超时为 10s 。由于我们引入的延时，`/productpage` 提前超时并引发错误。

这些类型的错误可能发生在典型的企业应用程序中，其中不同的团队独立地开发不同的微服务。Istio 的故障注入规则可帮助您识别此类异常，而不会影响最终用户。

#### 错误修复

你通常会解决这样的问题：

1. 要么增加 `/productpage` 的超时或者减少 `reviews` 到 `ratings` 服务的超时
2. 终止并重启微服务
3. 确认 `productpage` 正常响应并且没有任何错误。

但是，我们已经在 `reviews` 服务的 v3 版中运行此修复程序，因此我们可以通过将所有流量迁移到 `reviews:v3` 来解决问题， 如[流量转移](https://archive.istio.io/v1.2/zh/docs/tasks/traffic-management/traffic-shifting/)中所述任务。

#### 练习

将延迟规则更改为使用 2.8 秒延迟，然后针对 v3 版本的 `reviews` 运行它。

`kubectl apply -f virtual-service-ratings-test-delay-2.yaml`

*virtual-service-ratings-test-delay-2.yaml*

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:
      delay:
        percentage:
          value: 100.0
        fixedDelay: 2.8s #缩短延时
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

### 02 使用 HTTP abort 进行故障注入

测试微服务弹性的另一种方法是引入 HTTP abort 故障。在这个任务中，在 `ratings` 微服务中引入 HTTP abort ，测试用户为 `jason` 。

在这个案例中，我们希望页面能够立即加载，同时显示 `Ratings service is currently unavailable` 这样的消息。

####  为用户 `jason` 创建故障注入规则发送 HTTP abort 

`kubectl apply -f virtual-service-ratings-test-abort.yaml`

*virtual-service-ratings-test-abort.yaml*

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:
      abort:
        percentage:
          value: 100.0
        httpStatus: 500
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1

```

#### 测试

1. 通过浏览器打开 [Bookinfo](https://archive.istio.io/v1.2/zh/docs/examples/bookinfo) 应用。

2. 使用用户 `jason` 登陆到 `/productpage` 界面。

   如果规则成功传播到所有的 pod，您应该能立即看到页面加载并看到 `Ratings service is currently unavailable` 消息。

3. 如果您注销用户 `jason` 或在匿名窗口（或其他浏览器）中打开 Bookinfo 应用程序， 您将看到 `/productpage` 为除 `jason` 以外用户调用了 `reviews:v1`（它根本不调用 `ratings`）。 因此，您不会看到任何错误消息。

### 清理

`kubectl delete -f virtual-service-all-v1.yaml`