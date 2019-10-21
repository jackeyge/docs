# k8s污点

>  Taint 和 toleration 相互配合，可以用来避免 pod 被分配到不合适的节点上。每个节点上都可以应用一个或多个 taint ，这表示对于那些不能容忍这些 taint 的 pod，是不会被该节点接受的。如果将 toleration 应用于 pod 上，则表示这些 pod 可以（但不要求）被调度到具有匹配 taint 的节点上。 

### 给节点增加一个 taint 

```shell
kubectl taint nodes node1 key=value:NoSchedule
```

### 删除一个 taint

```shell
kubectl taint nodes node1 key:NoSchedule-
```

### 使用

 您可以在PodSpec中为容器设定容忍标签。以下两个容忍标签都与上面的 `kubectl taint` 创建的污点“匹配”， 因此具有任一容忍标签的Pod都可以将其调度到“ node1”上： 

```yaml
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

```yaml
tolerations:
- key: "key"
  operator: "Exists"
  effect: "NoSchedule"
```

一个 toleration 和一个 taint 相“匹配”是指它们有一样的 key 和 effect ，并且：

- 如果 `operator` 是 `Exists` （此时 toleration 不能指定 `value`），或者
- 如果 `operator` 是 `Equal` ，则它们的 `value` 应该相等

 ### effect 取值

-  NoSchedule ： 只有拥有和这个 taint 相匹配的 toleration 的 pod 才能够被分配到 `node1` 这个节点 
-  PreferNoSchedule ： 系统会*尽量*避免将 pod 调度到存在其不能容忍 taint 的节点上，但这不是强制的。 
-  NoExecute ：  Kubernetes 不会将 pod 分配到该节点（如果 pod 还未在节点上运行），或者将 pod 从该节点驱逐（如果 pod 已经在节点上运行）。 

### 结合label创建专用节点

 如果您想将某些节点专门分配给特定的一组用户使用，您可以给这些节点添加一个 taint（即， `kubectl taint nodes nodename dedicated=groupName:NoSchedule`），然后给这组用户的 pod 添加一个相对应的 toleration（通过编写一个自定义的[admission controller](https://kubernetes.io/docs/admin/admission-controllers/)，很容易就能做到）。拥有上述 toleration 的 pod 就能够被分配到上述专用节点，同时也能够被分配到集群中的其它节点。如果您希望这些 pod 只能被分配到上述专用节点，那么您还需要给这些专用节点另外添加一个和上述 taint 类似的 label （例如：`dedicated=groupName`），同时 还要在上述 admission controller 中给 pod 增加节点亲和性要求上述 pod 只能被分配到添加了 `dedicated=groupName` 标签的节点上。 

```yaml
      tolerations:
        - key: "ingress"
          operator: "Equal"
          value: "true"
          effect: "NoSchedule"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: ingress
                operator: In
                values:
                - "true"
```

