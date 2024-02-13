<!-- TRANSLATED by md-translate -->
# 后端协议

例如 "V1.0.0+ 版本中的实验频道"。

```
This concept is part of the Experimental Channel in `v1.0.0+`.
```

并非所有 gateway API 实现都支持自动协议选择。 在某些情况下，协议会在没有明确选择的情况下被禁用。

当路由的后端引用 Kubernetes 服务时，应用程序开发人员可以使用 `ServicePort` [`appProtocol`](https://kubernetes.io/docs/concepts/services-networking/service/#application-protocol) 字段指定协议。

例如，下面的 `store` Kubernetes 服务显示端口 `8080` 支持 HTTP/2 先验知识。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store
spec:
  selector:
    app: store
  ports:
  - protocol: TCP
    appProtocol: kubernetes.io/h2c
    port: 8080
    targetPort: 8080
```

目前，gateway API 已对以下方面进行了一致性测试：

* `kubernetes.io/h2c` - HTTP/2 预先知识
* `kubernetes.io/ws` - 通过 HTTP 的 WebSocket
