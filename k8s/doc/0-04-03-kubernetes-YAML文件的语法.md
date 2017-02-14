k8s yaml文件详解
===
k8中创建RC,pod.svc都通过.yaml文件创建，以redis为例：
``` shell
vi redis-master-controller.yaml
 
apiVersion: v1
kind: Replicationrroller                 #Replicationtroller 表示这是个RC
metadate:
  name: redis-master
  labels:
    name: redis-master
spec:
  replicas: 1                       #当pod数量小于replicas时，RC会根据sepc.template字段定义的pod模板来生成新的pod实例
  selector:                         #spec.selector是RC的pod选择器，即监控和管理拥有这些label的pod实例
    name: redis-master
  template:
    labels:
      name: redis-master            #label属性指定了该pod的标签，label必须匹配RC的spec.selector ;spec.image
  spec:
    containers:
    - name: master
        image: kubeguide/redis-master
        ports:
        - containerPort: 6379
```
创建RC
``` shell
kubectl create -f redis-master-controller.yaml
```
Service
---
``` shell
vi redis-master-service.yaml

apiVersion: v1
kind:service
metadata:
  name:redis-master
  levels:
    name: redis-master
spec:
  ports:
  - port:6379
    targetPort: 6379      #提供该服务的容器EXPOSE的端口号，具体的服务进程在容器的targetPort上提供服务，port属性定义Server的虚拟端口
  selector:
    name: redis-master
```
