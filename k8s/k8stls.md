转自 https://blog.51cto.com/ylw6006/2167592 

> 在前文介绍的k8s master高可用实践方案中，需要对kube-apiserver的证书进行更新，加入VIP和从节点的IP，然后重新下发证书。回顾K8S集群整个搭建过程中，最容易让人懵圈的也就是配置证书环节，因此本文对K8S集群所用到的证书进行梳理一下。

# 一、根证书

> ca.pem 根证书公钥文件
> ca-key.pem 根证书私钥文件
> ca.csr 证书签名请求，用于交叉签名或重新签名
> ca-config.json 使用cfssl工具生成其他类型证书需要引用的配置文件
> ca.pem用于签发后续其他的证书文件，因此ca.pem文件需要分发到集群中的每台服务器上去。

证书生成命令，默认生成的证书有效期5年

```
# echo '{"CN":"CA","key":{"algo":"rsa","size":2048}}' | cfssl gencert -initca - | cfssljson -bare ca -
# echo '{"signing":{"default":{"expiry":"43800h","usages":["signing","key encipherment","server auth","client auth"]}}}' > ca-config.json
```

![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/832456b9c000b7f962860d718e8527e4.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

# 二、flannel证书

证书生成命令，默认生成的证书有效期5年

```
# cfssl gencert -ca=/etc/ssl/etcd/ca.pem  \
 -ca-key=/etc/ssl/etcd/ca-key.pem   \
-config=/etc/ssl/etcd/ca-config.json  \
 -profile=kubernetes flanneld-csr.json | cfssljson -bare flanneld
```

![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/8f1563c95c09bf9d3628e44e1c91f541.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
这里生成证书需要flanneld-csr.json文件

```
# cat flanneld-csr.json 
{
  "CN": "flanneld",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "FuZhou",
      "L": "FuZhou",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

flannel启动文件配置

```
# cat /usr/lib/systemd/system/flanneld.service 
[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service

[Service]
Type=notify
ExecStart=/usr/local/bin/flanneld \
  -etcd-cafile=/etc/ssl/etcd/ca.pem \
  -etcd-certfile=/etc/ssl/flanneld/flanneld.pem \
  -etcd-keyfile=/etc/ssl/flanneld/flanneld-key.pem \
  -etcd-endpoints=https://192.168.115.5,https://192.168.115.6:2379,https://192.168.115.7:2379 \
  -etcd-prefix=/kubernetes/network
ExecStartPost=/usr/local/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/docker
Restart=on-failure

[Install]
WantedBy=multi-user.target
RequiredBy=docker.service
```

# 三、etcd证书

1、服务端证书

> server.pem etcd服务端证书公钥文件
> server-key.pem etcd服务端证书私钥文件
> server.csr 证书签名请求

证书生成命令，默认生成的证书有效期5年

```
# export ADDRESS=192.168.115.5,192.168.115.6,192.168.115.7,vm1,vm2,vm3
# export NAME=server
# echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' |  \
cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem \
-hostname="$ADDRESS" - | cfssljson -bare $NAME
```

![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/1c2cd101d4b672d0e23435e96d0b1352.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
server.pem、server-key.pem文件用来etcd集群间通信加解密，因此所有的etcd服务器都需要有这两个文件

```
# tail -15 /usr/lib/systemd/system/etcd.service   
--initial-cluster-token=etcd-cluster-token \
--initial-cluster-state=new \
--cert-file=/etc/ssl/etcd/server.pem \
--key-file=/etc/ssl/etcd/server-key.pem \
--peer-cert-file=/etc/ssl/etcd/server.pem \
--peer-key-file=/etc/ssl/etcd/server-key.pem \
--trusted-ca-file=/etc/ssl/etcd/ca.pem \
--peer-trusted-ca-file=/etc/ssl/etcd/ca.pem \
--peer-client-cert-auth=true \
--client-cert-auth=true"
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

2、客户端证书

> client.pem etcd客户端证书公钥文件
> client-key.pem etcd客户端证书私钥文件
> client.csr 证书签名请求

证书生成命令，默认生成的证书有效期5年

```
# export ADDRESS=
# export NAME=client
# echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | \
 cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem \
-hostname="$ADDRESS" - | cfssljson -bare $NAME
```

![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/dd597dab4b2516b62078359dcfc9c8b3.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
client.pem、client-key.pem文件给etcdctl客户端用来和etcd服务器进行通信，可根据实际需要进行配置。

```
# export ETCDCTL_API=3
# etcdctl --write-out=table  \
--cert=/etc/ssl/etcd/client.pem \
--key=/etc/ssl/etcd/client-key.pem \
--cacert=/etc/ssl/etcd/ca.pem \
--endpoints=https://192.168.115.5:2379,https://192.168.115.6:2379,https://192.168.115.7:2379 \
member list
```

![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/53c9574f8edfb80cdef4f16b16663e4b.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

# 四、Master节点证书

> Kube-apiserver证书
> Kubernetes.pem kube-apiserver证书公钥文件
> Kubernetes-key.pem kube-apiserver证书私钥文件
> kuberentes.csr kube-apiserver证书签名请求

证书生成命令，默认生成的证书有效期5年

```
# cfssl gencert -ca=/etc/ssl/etcd/ca.pem \
  -ca-key=/etc/ssl/etcd/ca-key.pem \
  -config=/etc/ssl/etcd/ca-config.json \
  -profile=kubernetes k8s-csr.json | cfssljson -bare kubernetes
```

![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/8251ca6f60ba41219b793bac00781cf7.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
这里生成证书需要k8s-csr.json文件，其中定义了master节点的IP列表等信息

```
# cat k8s-csr.json 
{
  "CN": "kubernetes",
  "hosts": [
    "127.0.0.1",
    "192.168.115.4",
    "192.168.115.5",
    "192.168.115.6",
    "192.168.115.7",
    "10.254.0.1",
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "FuZhou",
      "L": "FuZhou",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

# 五、Node节点证书

1、kube-proxy证书

> Kube-proxy.pem kube-proxy证书公钥文件
> Kube-proxy-key.pem kube-proxy证书私钥文件
> Kube-proxy.csr kube-proxy证书签名请求

证书生成命令，默认生成的证书有效期5年

```
# cfssl gencert -ca=/etc/ssl/etcd/ca.pem \
  -ca-key=/etc/ssl/etcd/ca-key.pem \
  -config=/etc/ssl/etcd/ca-config.json \
  -profile=kubernetes  kube-proxy-csr.json | cfssljson -bare kube-proxy
```

![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/545da792f00324af2f4590e5a10c7cc5.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
这里生成证书需要kube-proxy-csr.json文件

```
# cat kube-proxy-csr.json 
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "FuZhou",
      "L": "FuZhou",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

2、Kubelet证书

> Kubelet-client.crt: kubectl客户端证书公钥文件
> Kubelet-client.key: kubectl客户端私钥文件
> Kubelet.crt：kubelet服务端证书公钥文件
> Kubelet.key：kubelet服务端证书私钥文件
>
> > kubelet-client.crt 文件在 kubelet 完成 TLS bootstrapping 后生成，有效期为 1 年。此证书是由 controller manager 签署的，此后 kubelet 将会加载该证书，用于与 apiserver 建立 TLS 通讯，同时使用该证书的 CN 字段作为用户名，O 字段作为用户组向 apiserver 发起其他请求。
> > kubelet.crt 文件在 kubelet 完成 TLS bootstrapping 后且没有配置 --feature-gates=RotateKubeletServerCertificate=true 时才会生成；该文件为一个独立于 apiserver CA 的自签 CA 证书，有效期为 1 年；被用作 kubelet 10250 api 端口

![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/ae06ef6c3d958209919072cdaaca45c2.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/f713c87203ea4feb24c68546c42f7290.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

关于kubelet首次启动 TLS bootstrapping的介绍（先有鸡还是先有蛋问题的解决方案）可参考文档，https://mritd.me/2018/01/07/kubernetes-tls-bootstrapping-note/

# 六、配置证书自动续期

> 默认签署kubectl客户端和kubelet服务端证书只有 1 年有效期，如果想要调整证书有效期可以通过设置 kube-controller-manager 的 --experimental-cluster-signing-duration 参数实现，该参数默认值为 8760h0m0s。下面我们来介绍一下如何实现证书到期的自动续签。这个问题如果处理不当，证书过期之后会出现所有的node节点连接不上的情况。

1、kcm服务，这里为了方便测试，过期时间修改为30分钟

```
# egrep 'feature|experimental' /usr/lib/systemd/system/kube-controller-manager.service 
  --feature-gates=RotateKubeletClientCertificate=true,RotateKubeletServerCertificate=true \
  --experimental-cluster-signing-duration=30m0s \
# systemctl  daemon-reload
# systemctl restart kube-controller-manager
```

2、kubelet服务
配置完成删掉Kubelet-client.crt、Kubelet-client.key、Kubelet.crt、Kubelet.key四个文件后重启kubelet服务。

```
# egrep 'feature|rotate' /usr/lib/systemd/system/kubelet.service 
  --feature-gates=RotateKubeletClientCertificate=true,RotateKubeletServerCertificate=true \
  --rotate-certificates=true \
# systemctl  daemon-reload
# systemctl restart kubelet
```

3、手工签发证书

```
# kubectl create clusterrolebinding kubelet-clinet \
--clusterrole=system:node  \
--user=system:anonymous
```

> 如果缺少对system:anonymous用户的授权，kubelet启动的时候会报错如下：
> error: failed to run Kubelet: cannot create certificate signing request: certificatesigningrequests.certificates.k8s.io is forbidden: User "system:anonymous" cannot create certificatesigningrequests.certificates.k8s.io at the cluster scope

```
# kubectl get csr
# kubectl certificate approve csr-fn946
# kubectl certificate approve csr-kwvg9 
```

![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/d6435f03139e8bccb5c46d48d98ff9ed.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

node节点将会重新生成kubectl客户端和kubelet服务端证书
![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/047419bafe2b5a7849cae91544537ecd.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

4、配置自动签发证书，在大规模集群下，这个配置是必须的

```
# cat rbac.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:certificates.k8s.io:certificatesigningrequests:selfnodeserver
rules:
- apiGroups:
  - certificates.k8s.io
  resources:
  - certificatesigningrequests/selfnodeserver
  verbs:
  - create
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubeadm:node-autoapprove-certificate-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:certificates.k8s.io:certificatesigningrequests:selfnodeserver
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:nodes

# kubectl create -f  rbac.yaml
# kubectl create clusterrolebinding node-client-auto-approve-csr  \
--clusterrole=system:certificates.k8s.io:certificatesigningrequests:nodeclient  \
--group=system:bootstrappers
# kubectl create clusterrolebinding node-client-auto-renew-crt \
--clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeclient \
--group=system:nodes
# kubectl create clusterrolebinding node-server-auto-renew-crt  \
--clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeserver  \
--group=system:nodes
```

![K8S集群tls证书管理](https://s1.51cto.com/images/blog/201808/31/b31c790a7402052bc6d186f1680183de.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

这里需要注意的是删除掉Kubelet-client.crt、Kubelet-client.key两个文件之后，启动kubelet服务，首先会生成一个Kubelet-client.key文件，我们需要对这个证书的crs请求进行approve，否则node节点无法正常启动。
其次，如果kubelet.kubeconfig文件中配置的client-certificate、client-key目录位置和kubelet的启动参数--cert-dir不一致，则kubelet.kubeconfig文件中的配置文件会被自动更新。