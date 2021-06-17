---
title: "对kustomize的resource进行明确指定"
date: 2021-06-17T15:49:36+08:00
draft: false
weight: 13
---

使用labelSelector等字段对kustomize管理的多种同类型resource进行更明确的指定。

<!--more-->

## 问题背景

base：
```bash
base
├── deployment.yaml
├── kustomization.yaml
├── monitor.yaml
├── namespace.yaml
├── prom_service.yaml
└── service.yaml
```

overlays:
```bash
overlays
├── container_lifecycle.yaml
├── container_liveness_probe.yaml
├── container_ports.yaml
├── container_readiness_probe.yaml
├── container_resources.yaml
├── kustomization.yaml
├── prom_service_ports.yaml
├── replica.yaml
└── service_ports.yaml
```

base/service.yaml:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: brickzzhang-test #<service.name>
  labels:
    app: brickzzhang-test #<service.name>
    app.kubernetes.io/type: non-prometheus
  namespace: default #<service.namespace>

spec:
  type: LoadBalancer
  selector:
    app: brickzzhang-test #<service.name>
```

base/prom_service.yaml:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: brickzzhang-test-prom #<service.name>
  labels:
    app: brickzzhang-test #<service.name>
    app.kubernetes.io/type: prometheus
  namespace: default #<service.namespace>

spec:
  type: ClusterIP
  selector:
    app: brickzzhang-test #<service.name>
```

overlays/service_ports.yaml:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: negligible
  labels:
    app.kubernetes.io/type: non-prometheus

spec:
  ports:
  - name: grpc-port
    port: 8080 #<api.grpc.port>
    protocol: TCP
    targetPort: 8080 #<api.grpc.port>
  - name: http-port
    port: 8081 #<api.http.port>
    protocol: TCP
    targetPort: 8081 #<api.http.port>
```

overlays/prom_service_ports.yaml:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: negligible
  labels:
    app.kubernetes.io/type: prometheus

spec:
  ports:
  - name: prom-port
    port: 1608 #<api.prometheus.port>
    protocol: TCP
    targetPort: 1608 #<api.prometheus.port>
```

overlays/kustomization.yaml:
```yaml
resources:
- ../base
patches:
- path: container_liveness_probe.yaml
  target:
    kind: Deployment
- path: container_readiness_probe.yaml
  target:
    kind: Deployment
- path: replica.yaml
  target:
    kind: Deployment
- path: container_ports.yaml
  target:
    kind: Deployment
- path: container_resources.yaml
  target:
    kind: Deployment
- path: container_lifecycle.yaml
  target:
    kind: Deployment
- path: service_ports.yaml
  target:
    labelSelector: app.kubernetes.io/type=non-prometheus
    kind: Service

- path: prom_service_ports.yaml
  target:
    labelSelector: app.kubernetes.io/type=prometheus
    kind: Service
```

项目需要将访问http使用的service对外暴露，prometheus则clusterIP类型的service将访问权限限定在本集群内。开始base和overlays目录下service yaml文件中没有定义相关labels，且overlays/kustomization.yaml中并未使用labelSelector对service target进行限定。
执行`kustomize build ./`只生成一个service，生成的service则包含网关和prometheus的ports，与预期不符。

## 解决方法

此种现象必然是由于kustomization.yaml中对service资源的指定不够详细导致。[kustomize官方](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/patchMultipleObjects.md)给出了明确指定patch target的方法。这里采用了`labelSelector`这种方式。

## 参考链接

[kustomize readme](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/patchMultipleObjects.md)
