# Домашнее задание к занятию «Helm»

## Задание 1. Подготовить Helm-чарт для приложения

1. Создадим структуру чарта:
```bash
helm create myapp
cd myapp
rm -rf templates/*  # Удаляем стандартные шаблоны
```

2. Создадим файлы чарта. Вот пример структуры:
```
myapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   └── _helpers.tpl
└── values.yaml
```

3. Пример `Chart.yaml`:
```yaml
apiVersion: v2
name: myapp
description: A Helm chart for my application
version: 0.1.0
appVersion: "1.0"
```

4. Пример `values.yaml`:
```yaml
global:
  namespace: default

frontend:
  replicaCount: 1
  image:
    repository: nginx
    tag: "1.23.4"
    pullPolicy: IfNotPresent
  service:
    type: ClusterIP
    port: 80

backend:
  replicaCount: 1
  image:
    repository: busybox
    tag: "1.36"
    pullPolicy: IfNotPresent
  service:
    type: ClusterIP
    port: 8080
```

5. Пример шаблона деплоймента (`templates/frontend-deployment.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-frontend
  namespace: {{ .Values.global.namespace }}
  labels:
    app: {{ .Release.Name }}-frontend
spec:
  replicas: {{ .Values.frontend.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-frontend
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-frontend
    spec:
      containers:
        - name: frontend
          image: "{{ .Values.frontend.image.repository }}:{{ .Values.frontend.image.tag }}"
          imagePullPolicy: {{ .Values.frontend.image.pullPolicy }}
          ports:
            - containerPort: 80
```

## Задание 2. Запустить две версии в разных неймспейсах

1. Создадим неймспейсы:
```bash
kubectl create namespace app1
kubectl create namespace app2
```

2. Установим первую версию в app1:
```bash
helm install app1-v1 ./myapp --namespace app1 \
  --set global.namespace=app1 \
  --set frontend.image.tag="1.23.4" \
  --set backend.image.tag="1.36"
```

3. Установим вторую версию в app1 (с другими параметрами):
```bash
helm install app1-v2 ./myapp --namespace app1 \
  --set global.namespace=app1 \
  --set frontend.image.tag="1.25.2" \
  --set backend.image.tag="latest" \
  --set frontend.replicaCount=2
```

4. Установим третью версию в app2:
```bash
helm install app2-v1 ./myapp --namespace app2 \
  --set global.namespace=app2 \
  --set frontend.image.tag="1.24.0" \
  --set backend.image.tag="1.35" \
  --set backend.replicaCount=3
```

5. Проверим установленные релизы:
```bash
helm list -A
```

6. Проверим поды в разных неймспейсах:
```bash
kubectl get pods -n app1
kubectl get pods -n app2
```

7. Пример вывода (может отличаться в вашем случае):
```
NAMESPACE  NAME                     READY  STATUS   RESTARTS  AGE
app1       app1-v1-frontend-xxxxx   1/1    Running  0         2m
app1       app1-v1-backend-xxxxx    1/1    Running  0         2m
app1       app1-v2-frontend-xxxxx   1/1    Running  0         1m
app1       app1-v2-frontend-xxxxx   1/1    Running  0         1m
app1       app1-v2-backend-xxxxx    1/1    Running  0         1m
app2       app2-v1-frontend-xxxxx   1/1    Running  0         30s
app2       app2-v1-backend-xxxxx    1/1    Running  0         30s
app2       app2-v1-backend-xxxxx    1/1    Running  0         30s
app2       app2-v1-backend-xxxxx    1/1    Running  0         30s
```

8. Для демонстрации различий можно проверить образы:
```bash
kubectl get deployments -n app1 -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.template.spec.containers[0].image}{"\n"}{end}'
kubectl get deployments -n app2 -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.template.spec.containers[0].image}{"\n"}{end}'
```

Таким образом, мы успешно развернули три разных версии приложения в двух неймспейсах с помощью Helm, продемонстрировав гибкость управления версиями и конфигурациями приложений в Kubernetes.
