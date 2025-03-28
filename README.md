# k8s-helm-course

```bash

# Устанавливаем minukube (кластер с одной нодой, для обучения)
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/

minikube start --driver=docker

# Дашборд minikube
minikube dashboard

# Сервисы: ClusterIp, NodePort, LoadBalancer, Ingress - обеспечение доступов к контейнеру
# Применить все файлы в папке
kubectl apply -f .

kubectl apply -f pod.yml
kubectl apply -f node-port.yml

kubectl get pods -owide
kubectl get services

# Узнаем ip кластера
minikube ip

# Получаем инфу о поде или сервисе
kubectl describe pods short-app
kubectl describe service short-app-port

# Деплоймент
# Обновление без даунтайма выполняется, если тег приложения будет отличаться
# Rollout - выкатка нового обраба
kubectl rollout history deployment short-app-deployment

# Обновляем императивным путем (меняем образ у деплоймента)
kubectl set image deployment.apps/short-app-deployment short-app=antonlarichev/short-app:latest
kubectl rollout restart deployment short-app-deployment
```