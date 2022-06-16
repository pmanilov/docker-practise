Deployment:

```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: pmanilov/app:latest
        ports:
        - containerPort: 8000
```

```shell
kubectl get all -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS        AGE
default       pod/app-deployment-55bdb84c64-sxq6n    1/1     Running   0               3m34s
default       pod/app-deployment-55bdb84c64-xbzr4    1/1     Running   0               3m34s
kube-system   pod/coredns-64897985d-2rcrv            1/1     Running   1 (5d23h ago)   6d1h
kube-system   pod/etcd-minikube                      1/1     Running   1 (5d23h ago)   6d1h
kube-system   pod/kube-apiserver-minikube            1/1     Running   1 (5d23h ago)   6d1h
kube-system   pod/kube-controller-manager-minikube   1/1     Running   1 (15m ago)     6d1h
kube-system   pod/kube-proxy-wz42m                   1/1     Running   1 (15m ago)     6d1h
kube-system   pod/kube-scheduler-minikube            1/1     Running   1 (5d23h ago)   6d1h
kube-system   pod/storage-provisioner                1/1     Running   3 (14m ago)     6d1h

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  6d1h
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   6d1h

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   6d1h

NAMESPACE     NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/app-deployment   2/2     2            2           3m34s
kube-system   deployment.apps/coredns          1/1     1            1           6d1h

NAMESPACE     NAME                                        DESIRED   CURRENT   READY   AGE
default       replicaset.apps/app-deployment-55bdb84c64   2         2         2       3m34s
kube-system   replicaset.apps/coredns-64897985d           1         1         1       6d1h
```

Пробрасываю порты для каждой реплики:

```shell
$ kubectl port-forward pod/app-deployment-55bdb84c64-sxq6n 8080:8000
Forwarding from 127.0.0.1:8080 -> 8000
Forwarding from [::1]:8080 -> 8000

$ minikube kubectl -- port-forward pod/app-deployment-55bdb84c64-xbzr4 8081:8000
Forwarding from 127.0.0.1:8081 -> 8000
Forwarding from [::1]:8081 -> 8000
```

Проверяю что все работает:

```shell
[pavel@pavel-asus ~]$ curl http://127.0.0.1:8080/hello
<!DOCTYPE HTML>
<html>
<head>
    <title>Hello world</title>
</head>
<body>
<p>Hello world!!!</p>
<img src="docker.png" alt="Логотип Docker"/>
<p></p>
<img src="kuber.png" alt="Логотип Kubernetes"/>
</body>
</html>
[pavel@pavel-asus ~]$ curl http://127.0.0.1:8081/hello
<!DOCTYPE HTML>
<html>
<head>
    <title>Hello world</title>
</head>
<body>
<p>Hello world!!!</p>
<img src="docker.png" alt="Логотип Docker"/>
<p></p>
<img src="kuber.png" alt="Логотип Kubernetes"/>
</body>
```
