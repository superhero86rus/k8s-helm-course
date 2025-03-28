# k8s-helm-course

```bash

# Устанавливаем minukube (кластер с одной нодой, для обучения)
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/

minikube start --driver=docker

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
```