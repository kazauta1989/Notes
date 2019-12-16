# Network

k8s中常用的几种IP地址：
- Node IP：集群中一个主机节点的IP地址。
- Pod IP：一个Pod的IP地址。
- Service IP：不管一个应用运行了多少个Pod实例，都是暴露一个Service IP供用户访问。
  - 一个服务访问外部时，源地址是Pod IP；外部访问一个服务时，目的地址是Service IP。
- Ingress IP：服务通过该IP暴露到集群外，供集群外的主机访问。

k8s中主要研究的网络通信：
- 同一个Pod内，容器之间的通信
- 同一个服务内，Pod之间的通信
- 同一个集群内，Pod到服务的通信
- 集群内与集群外的通信

## Service

：提供服务发现、反向代理、负载均衡的功能。

Service的配置文件通常命名为service.yaml，内容示例如下：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: redis
spec:
  type: ClusterIP
  clusterIP: 10.124.0.1
  selector:
    app: redis
  ports:
  - name: redis
    port: 6379
    protocol: TCP
    targetPort: 6379
  - name: sentinel
    port: 26379
    protocol: TCP
    targetPort: 26379
```
- 该Service有一个clusterIP、两个port，供用户访问。
  - 用户可以通过“ServiceIP:Port”、“ServiceName:Port”的方式访问该服务。
  - 该Service通过selector选中符合条件的Pod（可能有多个），用Pod IP和port合并成EndPoint（比如10.124.0.1:6379，可能有多个），将用户的访问流量转发到EndPoint。
  - protocol默认为TCP，还可以填UDP。
  - targetPort是指流量被转发到的Pod端口。

### 主要分类

- `type: ClusterIP` ：默认类型，给Service分配一个集群内的虚拟IP，可以被集群内节点访问。
- `type: NodePort` ：从Node的30000~32767端口中随机选取或指定一个端口，供用户访问。访问 NodeIP:Port 的流量会被转发到EndPoint。如下：
    ```yaml
    spec:
      type: NodePort
      clusterIP: 10.124.0.1
      selector:
        app: redis
      ports:
      - name: redis
        nodePort: 31533
        port: 6379
        protocol: TCP
        targetPort: 6379
    ```
    - NodePort类型的Service可以被集群外同网段的主机访问。
- `type: LoadBalancer` ：给Service分配一个负载均衡IP，供集群外访问。访问 loadBalancerIP:Port 的流量会被转发到EndPoint。如下：
    ```yaml
    spec:
      type: LoadBalancer
      clusterIP: 10.124.0.1
      loadBalancerIP: 123.0.0.1
      selector:
        app: redis
      ports:
        - name: redis
          port: 6379
          protocol: TCP
          targetPort: 6379
    ```
- `ExternalName` ：将Service名解析到 spec.externalName 指定的域名（要被DNS解析，不能是IP地址）。如下：
    ```yaml
    spec:
      type: ExternalName
      externalName: redis.test.com
    ```
- externalIPs ：给Service分配集群外的IP，此时Service可以是任意type。如下：
    ```yaml
    spec:
      selector:
        app: redis
      ports:
      - name: redis
        port: 6379
        protocol: TCP
        targetPort: 6379
      externalIPs:
      - 123.0.0.1
    ```
- Headless Service：配置`clusterIP: None`。此时Service没有自己的IP，必须通过selector选中一个Pod，Service名会被解析到Pod IP。

## Ingress

：提供集群外部的访问入口，转发到集群内部的某个服务。
- 相当于第七层的反向代理。

配置示例：
```yaml
apiVersion: v1
kind: Ingress
metadata:
  name: test-ingress
spec:
  rules:                        # Ingress的入站规则列表
  - http:                       # 定义http协议的规则
      paths:
      - path: /login            # 将发往该URL的请求转发到后端（backend）
        backend:
           serviceName: nginx   # 后端的Service
           servicePort: 80
```

## 访问控制

- Service Account
- RBAC：基于角色的访问控制
- NetWorkPolicy ：管控Pod之间的网络流量，相当于第四层的ACL。