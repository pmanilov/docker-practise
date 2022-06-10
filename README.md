# Практика по Docker и Kubernetes

## Docker

Установил Docker и зарегистрировал аккаунт на Docker hub (имя - 'pmanilov')

Задачи:

1. Создать web-приложение, которое выводит содержимое папки app
2. Собрать его в виде Docker image
3. Запустить Docker container и проверить, что web-приложение работает.
4. Выложить image на Docker Hub.


## Шаг 1
Я создал веб-приложение с использованием Spring-boot, Spring MVC и Thymeleaf, которое выводит два html файла

index.html по адресу http://localhost:8080/:

```shell
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Welcome</title>
</head>
<body>
<p>Please open
    <a href="http://localhost:8080/hello">page</a>.
</p>
</body>
</html>
```

hello.html по адресу http://localhost:8080/hello:

```shell
<!DOCTYPE HTML>
<html xmlns:th="http://thymeleaf.org">
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
```

Затем я собрал это Maven-ом в исполняемый jar файл - app-1.0.0.jar.

Далее создал Dockerfile:

```shell
FROM bellsoft/liberica-openjdk-alpine:17.0.3.1
ARG USER=app
ARG UID=1001
ARG GID=1001
RUN addgroup --gid "$GID" "$USER" \
    && adduser \
    --disabled-password \
    --gecos "" \
    --home "$(pwd)" \
    --ingroup "$USER" \
    --no-create-home \
    --uid "$UID" \
    "$USER"
USER ${USER}
WORKDIR /app
COPY --chown=$USER:$USER target/app-1.0.jar /app.jar
EXPOSE 8000
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

## Шаг 2

Собрал Docker image командой:

```shell
$ sudo docker build -t pmanilov/app:1.0.0 -t pmanilov/web:latest .
Вывод терминала:

ending build context to Docker daemon  19.49MB
Step 1/10 : FROM bellsoft/liberica-openjdk-alpine:17.0.3.1
 ---> d1d0a945d9f0
Step 2/10 : ARG USER=app
 ---> Using cache
 ---> 068475f8165f
Step 3/10 : ARG UID=1001
 ---> Using cache
 ---> 2c34869ebd41
Step 4/10 : ARG GID=1001
 ---> Using cache
 ---> 9dbc991fda4c
Step 5/10 : RUN addgroup --gid "$GID" "$USER"     && adduser     --disabled-password     --gecos ""     --home "$(pwd)"     --ingroup "$USER"     --no-create-home     --uid "$UID"     "$USER"
 ---> Using cache
 ---> c64d29364105
Step 6/10 : USER ${USER}
 ---> Using cache
 ---> bf8e45623b3b
Step 7/10 : WORKDIR /app
 ---> Using cache
 ---> 2e5831104d95
Step 8/10 : COPY --chown=$USER:$USER target/app-1.0.jar /app.jar
 ---> Using cache
 ---> a0177173c39d
Step 9/10 : EXPOSE 8000
 ---> Using cache
 ---> 96d4932d043b
Step 10/10 : ENTRYPOINT ["java", "-jar", "/app.jar"]
 ---> Using cache
 ---> 0a4f7edbfb4b
Successfully built 0a4f7edbfb4b
Successfully tagged pmanilov/app:1.0.0
Successfully tagged pmanilov/app:latest
```

Команду выполнил не в первый раз, поэтому использовался кэш.
Посмотрим список image:

```shell
$ docker image ls
REPOSITORY                         TAG        IMAGE ID       CREATED        SIZE
pmanilov/app                       1.0.0      0a4f7edbfb4b   3 hours ago    143MB
pmanilov/app                       latest     0a4f7edbfb4b   3 hours ago    143MB
<none>                             <none>     7cce1d7049dc   3 hours ago    143MB
<none>                             <none>     c43e928922cd   16 hours ago   143MB
<none>                             <none>     bfd5a1c1da99   16 hours ago   143MB
bellsoft/liberica-openjdk-alpine   17.0.3.1   d1d0a945d9f0   7 days ago     124MB
gcr.io/k8s-minikube/kicbase        v0.0.30    1312ccd2422d   4 months ago   1.14GB
```

Как видим наш образ есть в списке.

## Шаг 3
Запустим наш контейнер из image 

```shell
docker run -ti --rm -p 8000:8000 --name app pmanilov/app:latest
Вывод терминала:


  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _' | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.7.0)

2022-06-10 16:27:38.167  INFO 1 --- [           main] com.manilov.app.AppApplication           : Starting AppApplication v1.0 using Java 17.0.3.1 on 26fa32f7ab18 with PID 1 (/app.jar started by app in /app)
2022-06-10 16:27:38.172  INFO 1 --- [           main] com.manilov.app.AppApplication           : No active profile set, falling back to 1 default profile: "default"
2022-06-10 16:27:40.182  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8000 (http)
2022-06-10 16:27:40.208  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2022-06-10 16:27:40.209  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.63]
2022-06-10 16:27:40.363  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2022-06-10 16:27:40.363  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 2078 ms
2022-06-10 16:27:41.012  INFO 1 --- [           main] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page: class path resource [static/index.html]
2022-06-10 16:27:41.399  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8000 (http) with context path ''
2022-06-10 16:27:41.428  INFO 1 --- [           main] com.manilov.app.AppApplication           : Started AppApplication in 4.247 seconds (JVM running for 5.422)

Проверим, что контейнер действительно запущен
$ docker ps
CONTAINER ID   IMAGE                                 COMMAND                  CREATED             STATUS             PORTS                                                                                                                                  NAMES
4ded18fe1b17   pmanilov/app:latest                   "java -jar /app.jar"     32 seconds ago      Up 31 seconds      0.0.0.0:8000->8000/tcp, :::8000->8000/tcp                                                                                              app
```

## Шаг 4
Я залогинился без токена:

$ docker login

Далее потребовало логин и пароль.

Отправил Docker image в Registry Docker Hub

```shell
$ docker push pmanilov/app:1.0.0

docker push pmanilov/app:latest
The push refers to repository [docker.io/pmanilov/app]
b1861d063df6: Pushed 
a2d87f6e2bef: Pushed 
7e548929bc71: Pushed 
ea55d676b374: Pushed 
34378ea9a6af: Pushed 
24302eb7d908: Pushed 
latest: digest: sha256:7d0e7e371e2d41574c3a8d30c7b4cd3dd1a17d3d37e1a8a052458b2ea4222359 size: 1578
```
На сайте image появился

Скачаем docker image из Docker Hub

```shell
$ docker pull pmanilov/app:1.0.0
1.0.0: Pulling from pmanilov/app
Digest: sha256:7d0e7e371e2d41574c3a8d30c7b4cd3dd1a17d3d37e1a8a052458b2ea4222359
Status: Image is up to date for pmanilov/app:1.0.0
docker.io/pmanilov/app:1.0.0
```

Такой Image уже есть.

## Kubernetes

Установил minikube и через него же и установил kubectl и прописал $ alias kubectl="minikube kubectl --" для того, чтобы не писать всегда minikube kubectl


Задачи:

1. Запустить кластер.
2. Создать Kubernetes Deployment manifest, запускающий container из созданного image.
3. Установить manifest в кластер Kubernetes.
4. Обеспечить доступ к web-приложению внутри кластера и проверить его работу

## Шаг 1.

Запустил кластер:

```shell
$ minikube start --embed-certs
😄  minikube v1.25.2 на Arch 21.2.6
✨  Используется драйвер docker на основе существующего профиля
👍  Запускается control plane узел minikube в кластере minikube
🚜  Скачивается базовый образ ...
🏃  Обновляется работающий docker "minikube" container ...
🐳  Подготавливается Kubernetes v1.23.3 на Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
🔎  Компоненты Kubernetes проверяются ...
    ▪ Используется образ gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Включенные дополнения: storage-provisioner, default-storageclass
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Готово! kubectl настроен для использования кластера "minikube" и "default" пространства имён по умолчанию
```

Пoсмотрел статус кластера: 

```shell
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Просмотр конфигурационного файла Kubernetes kubeconfig:

```shell
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    extensions:
    - extension:
        last-update: Fri, 10 Jun 2022 18:34:08 MSK
        provider: minikube.sigs.k8s.io
        version: v1.25.2
      name: cluster_info
    server: https://192.168.49.2:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Fri, 10 Jun 2022 18:34:08 MSK
        provider: minikube.sigs.k8s.io
        version: v1.25.2
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

Информация о кластере:

```shell
[pavel@pavel-asus app]$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

Получил список объектов кластера:

```shell
[pavel@pavel-asus app]$ kubectl get all -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS      AGE
kube-system   pod/coredns-64897985d-2rcrv            1/1     Running   0             27m
kube-system   pod/etcd-minikube                      1/1     Running   0             28m
kube-system   pod/kube-apiserver-minikube            1/1     Running   0             28m
kube-system   pod/kube-controller-manager-minikube   1/1     Running   0             28m
kube-system   pod/kube-proxy-wz42m                   1/1     Running   0             27m
kube-system   pod/kube-scheduler-minikube            1/1     Running   0             28m
kube-system   pod/storage-provisioner                1/1     Running   1 (27m ago)   27m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  28m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   28m

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   28m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   1/1     1            1           28m

NAMESPACE     NAME                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-64897985d   1         1         1       27m
```

## Шаг 2
Создаю свой Deployment:

```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 1
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

## Шаг 3
Применяю написанный мной манифест:

```shell
$ kubectl apply -f deployment.yaml -n default
deployment.apps/app-deployment created
```

Посмотрим как все прошло:

```shell
$ kubectl describe deployment app-deployment -n default
Name:                   app-deployment
Namespace:              default
CreationTimestamp:      Fri, 10 Jun 2022 18:54:04 +0300
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=app
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=app
  Containers:
   app:
    Image:        pmanilov/app:latest
    Port:         8000/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   app-deployment-55bdb84c64 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  118s  deployment-controller  Scaled up replica set app-deployment-55bdb84c64 to 1
```

## Шаг 4
Пробросим порт на нашу локальную машину:

```shell
$ kubectl port-forward deployments/app-deployment 8080:8000
Forwarding from 127.0.0.1:8080 -> 8000
Forwarding from [::1]:8080 -> 8000
Handling connection for 8080
```

В браузере все выводится и через команду curl результат тоже виден:

```shell
curl http://127.0.0.1:8080/hello
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

Удалим deployment с web-приложением:

```shell
$ kubectl delete deployment app-deployment
deployment.apps "app-deployment" deleted
```

Остановим minikube:

```shell
[pavel@pavel-asus ~]$ minikube stop
✋  Узел "minikube" останавливается ...
🛑  Выключается "minikube" через SSH ...
🛑  Остановлено узлов: 1.
```



