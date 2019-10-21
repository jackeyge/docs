# k8s对外提供服务

[TOC]

## hostNetwork: true

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: influxdb
spec:
  hostNetwork: true
  containers:
    - name: influxdb
      image: influxdb
```

## hostPort

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: influxdb
spec:
  containers:
    - name: influxdb
      image: influxdb
      ports:
        - containerPort: 8086
          hostPort: 8086
```

## NodePort

```yaml
kind: Service
apiVersion: v1
metadata:
  name: influxdb
spec:
  type: NodePort
  ports:
    - port: 8086
      nodePort: 30000
  selector:
    name: influxdb
```

## LoadBalancer

```yaml
kind: Service
apiVersion: v1
metadata:
  name: influxdb
spec:
  type: LoadBalancer
  ports:
    - port: 8086
  selector:
    name: influxdb
```

## Ingress

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: influxdb
spec:
  rules:
    - host: influxdb.kube.example.com
      http:
        paths:
          - backend:
              serviceName: influxdb
              servicePort: 8086
```

