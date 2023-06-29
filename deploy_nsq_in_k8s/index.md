# 在 Kubernetes 中部署 NSQ


本文将介绍如何在 Kubernetes 中部署 NSQ。

## 什么是 NSQ？

NSQ是一个基于 Go 语言的分布式实时消息平台, 它具有分布式、去中心化的拓扑结构，支持无限水平扩展。无单点故障、故障容错、高可用性以及能够保证消息的可靠传递的特征。另外，NSQ 非常容易配置和部署, 且支持众多的消息协议。支持多种客户端，协议简单，如果有兴趣，可参照协议自已实现一个也可以。简单点说，NSQ 就是一个基于Go 语言实现的轻量级消息平台。

NSQ 包含如下组件
* nsqd: Nsqd 是接收、队列和向客户端传递消息的守护进程
* nsqlookupd: Nsqlookupd 是管理拓扑信息的守护进程。客户端查询 nsqlookupd 以发现特定主题的 nsqd producers，nsqd 节点广播 topic 和 channel
* nsqadmin: Nsqadmin 是一套 Web 用户界面，可实时查看集群的统计数据和执行相应的管理任务

## 部署步骤

在正式部署之前，先分享几个小技巧

* 可以通过如下命令生成相应的 API 对象模板
  ```bash
  # 生成 Deployment 配置模板
  kubectl create deploy nsqd-dep --image=nsqio/nsq --dry-run=client -o yaml

  # 生成 service 配置模板
  # 为名为 nsqd-dep 的 Deployment 对象生成对应的 service 配置模板
  # 并指定暴露出去的 port 和对应 Deployment 的 port
  kubectl expose deploy nsqd-dep --port=4151 --target-port=4151 --dry-run=client -o yaml 
  ```
* 可以通过命令 `kubectl explain` 查看参数的具体含义
  ```bash
  # 查看 Deployment 的配置的各项参数的含义
  kubectl explain deploy
  # 查看 Deployment spec.template.spec 的配置的各项参数的含义
  kubectl explain deploy.spec.template.spec
  ```

### 部署 nsqlookupd

基于 nsqlookupd 的职责，需要使其在集群内以固定的方式被 nsqd 节点发现，所以需要为 nsqlookupd 配备固定的域名。具体做法为部署其为 service，具体配置如下
<details><summary>nsqlookupd</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nsqlookupd-dep
  name: nsqlookupd-dep
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nsqlookupd-dep
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nsqlookupd-dep
    spec:
      containers:
      - image: nsqio/nsq
        name: nsq
        resources: {}
        imagePullPolicy: IfNotPresent
        command:
          - /nsqlookupd
        ports:
        - containerPort: 4161
        - containerPort: 4160
status: {}

---

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nsqlookupd-svr
  name: nsqlookupd-svr
spec:
  ports:
  - port: 4161
    name: http
    protocol: TCP
    targetPort: 4161
  - port: 4160
    name: tcp
    protocol: TCP
    targetPort: 4160
  selector:
    app: nsqlookupd-dep
status:
  loadBalancer: {}
```
</details>

### 部署 nsqd

部署 nsqd 需要注意两点：
1. 设置容器的启动参数 `-lookupd-tcp-address`，需要将 nsqlookupd 的域名配置为参数 `-lookupd-tcp-address` 的值。在 K8S 中，service 的固定域名格式为 `{service 对象名}.{namespace 名字}.svc.cluster.local`，但很多时候也可以省略后面的部分，直接写`{service 对象名}.{namespace 名字}`甚至`{service 对象名}`就足够了，默认会使用对象所在的名 namespace 名字。
2. 需要指定持久化存储，保证 pod 在被重建后依然能获取到数据（topic, channel 等）。下面是一个使用 NFS 做持久化的配置示例
   ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
        name: nfs-client-retain
    provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
    parameters:
        onDelete: "retain"

    ---

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
        name: nfs-dyn-10m-pvc

    spec:
        storageClassName: nfs-client-retain
        accessModes:
            - ReadWriteMany

    resources:
        requests:
            storage: 10Mi
   ```



具体配置如下
<details><summary>nsqd</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nsqd-dep
  name: nsqd-dep
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nsqd-dep
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nsqd-dep
    spec:
      volumes:
      - name: nfs-dyn-10m-vol
        persistentVolumeClaim:
          claimName: nfs-dyn-10m-pvc
      containers:
      - image: nsqio/nsq
        name: nsq
        resources: {}
        imagePullPolicy: IfNotPresent
        args:
          - -lookupd-tcp-address=nsqlookupd-svr:4160
          - -data-path=/data
        command:
          - /nsqd
        ports:
        - containerPort: 4150
        - containerPort: 4151
        volumeMounts:
        - name: nfs-dyn-10m-vol
          mountPath: /data
status: {}
```

</details>

### [可选]部署 nsqadmin

部署 nsqadmin 同样需要指定 nsqlookupd 的地址，配置如下

<details><summary>nsqadmin</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nsqadmin-dep
  name: nsqadmin-dep
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nsqadmin-dep
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nsqadmin-dep
    spec:
      containers:
      - image: nsqio/nsq
        name: nsq
        resources: {}
        imagePullPolicy: IfNotPresent
        args:
          - -lookupd-http-address=nsqlookupd-svr:4161
        command:
          - /nsqadmin
        ports:
        - containerPort: 4150
        - containerPort: 4151
status: {}

---

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nsqadmin-svr
  name: nsqadmin-svr
spec:
  ports:
  - port: 4161
    name: http
    protocol: TCP
    targetPort: 4171
  selector:
    app: nsqadmin-dep
status:
  loadBalancer: {}
```

</details>

## 参考文档
[github: nsqio/nsq](https://github.com/nsqio/nsq)
[nsq doc](https://nsq.io/overview/quick_start.html)

