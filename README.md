# k8s
Kubernetes

**Kubernetes (K8s)** — это открытое программное обеспечение для автоматизации развёртывания, масштабирования и управления контейнеризированными приложениями.

**Kubernetes (k8s)** — фреймворк для запуска и управления приложениями в среде контейнеров.
- Объединяет несколько серверов в кластер с единым управлением и единым хранилищем конфигураций.
- Для запуска контейнеров использует docker (или другие).
- Основные принципы — это декларативный подход к описанию и идемпотентность.

Все объекты в kubernetes могут быть представлены в виде yaml или json файлов.

**Yaml — (Yet Another Markup Language)** язык разметки, совместимый с json. Использует отступы и отсюда считается более читабельным.

Основные параметры верхнего уровня:
- apiVersion версия апи,
- kind тип объекта,
- metadata метаданные для идентификации объекта,
- spec параметры объекта,
- status хранит текущий статус объекта (не указывается при создании).

Namespace — пространство имен. Используется для разделения объектов по неким критериям. Например, по окружению (prod, staging, ...) или по принадлежности к проекту (frontend, backend, ...)

Объекты условно можно разделить на:
- namespaced — привязанные к namespace,
- cluster-level — глобальные объекты.

**Labels** — лейблы. С их помощью реализован поиск объектов. И как следствие, их модификация. Если имя объекта динамическое и заранее невозможно его знать, то поиск таких объектов осуществляется с помощью лейблов.

**Примитивы Kubernetes**

**Pod** — минимальная единица рабочей нагрузки.
- В большинстве случаев: pod = контейнер
- Однако, возможно создание нескольких контейнеров в рамках одного пода.

**ReplicaSet** задает желаемую конфигурацию репликации и шаблон пода. Kubernetes будет стараться поддерживать это состояние и в случае удаления или остановки пода.

**Job** — разовый запуск пода. Обычно используется для бутстрапа или миграций.

**CronJob** — Запуск Job по расписанию.

**Deployment** задает шаблон replicaset и занимается выкаткой приложений. При изменении объекта создается новый replicaset, и одновременно со старым они начинают апскейлиться и даунскейлиться соответсвенно.

**DaemonSet** ведет себя аналогично деплойменту. Но вместо количества реплик daemonset поднимает по 1 поду на каждой ноде. Используется для приложений, которые необходимо держать на каждой ноде, например, node-exporter и.т.п.

**StatefulSet** — используется для деплоя кластерных приложений. Каждый под в нем имеет фиксированное имя, на которое можно сослаться в конфиге.

**ConfigMap** — используется для хранение конфигурации. Можно хранить как переменные окружения, так и файлы конфига целиком.

**Secret** — используется для хранения любой чувствительной информации (пароли, ключи и.т.п). В etcd хранится в зашифрованном виде. Может иметь один из нескольких типов:
- Opaque
- kubernetes.io/service-account-token
- kubernetes.io/dockercfg
- kubernetes.io/dockerconfigjson
- kubernetes.io/basic-auth
-  kubernetes.io/ssh-auth
- kubernetes.io/tlsdata
- bootstrap.kubernetes.io/token

**Service** — используется для направления трафика на под и балансировки в случае, если подов несколько. По умолчанию сетевой доступ будет обеспечен только внутри кластера. Имя сервиса будет использоваться в качестве DNS.

**Ingress** — объект, обрабатываемый ingress контроллером, реализует внешнюю балансировку (обычно по HTTP).
- Популярные контроллеры: nginx-ingress, traefik


**Minikube** — компактная версия k8s, созданная для тестирования и разработки в среде k8s. А также в учебных целях. Установка minikube зависит от операционной системы и системы виртуализации.

Полная инструкция доступна на официальном сайте:
```
https://minikube.sigs.k8s.io/docs/start/
```
Запуск: minikube start

**k3s** — аналог minikube, эмулирующий k8s. Запускается с помощью одного бинарного файла.

Установка возможна на linux с помощью bash скрипта:
```
curl -sfL https://get.k3s.io | sh -
```
Полная инструкция по установке доступна по ссылке:
```
https://rancher.com/docs/k3s/latest/en/quick-start/
```
Запуск: service k3s start

**Работа с кластером**

**Kubectl** — основная утилита для работы с k8s кластером. Использует конфигурацию, расположенную в файле ~/.kube/config Переопределить расположение этого файла можно с помощью переменной KUBECONFIG.
k3s имеет встроенный kubectl и может использоваться как:
```
k3s kubectl ...
```
Отдельно эту утилиту можно установить по инструкции:
```
https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/
```
Общий синтаксис:
```
kubectl действие объект флаги
```
Например:
```
kubectl get pods -n kube-system
```
**Основные действия:**
- get
- create
- edit
- delete
- describe
- apply
  
**Объекты:**
- pods
- deploy
- svc
- secrets
  
**Флаги** могут различаться для различных команд, из основных необходимо указывать namespace:
- -n
- --all-namespaces

Создаем файл: nginx.yaml

Деплоим с помощью:
```
kubectl apply -f nginx.yaml
```
Смотрим за статусом:
```
kubectl get po
```
Смотрим подробный статус пода:
```
kubectl describe po имя_пода
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Аналогом docker logs в k8s является команда kubectl logs
Пример:
```
kubectl logs --tail 100 имя_пода
```
Для того чтобы выполнить внутри пода какую-либо команду, используется:
```
kubectl exec -it имя_пода команда
```
Пример:
```
kubectl exec -it nginx-ans2l bash
```
Документация на русском:

https://kubernetes.io/ru/docs/home/

