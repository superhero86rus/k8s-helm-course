# k8s-helm-course

### Основы
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

# ClusterIP - доступ к контейнерам из других обьектов
# NodePort - доступ к контейнерам извне по порту
# LoadBalancer - доступ к одному сервису извне
# Ingress (Nginx, AWS, GCE) - доступ к набору сервисов извне
 
 # Для того чтобы мы обращались по имени хоста к приложению, нужно сконфигурировать minikube и hosts файл
 minikube addons list

 minikube addons enable ingress

 sudo nano /etc/hosts
 # Прописываем туда minikube ip
 # 192.168.49.2    demo.test

kubectl apply -f ingress.yml
kubectl get ingress

# Port Forwarding из кластера на нашу машину
kubectl port-forward pods/postgres-deployment-5b645bb98c-4jqv8 5432:5432
docker pull dpage/pgadmin4

# Отключили форвард и перезапускаем
kubectl rollout restart deployment postgres-deployment

# Volume - связан с контейнером - выделенная область внутри пода
# Persistent Volume - отдельное хранилище (статические и динамические)
# Persistent Volume Claim - связка Persistent Volume и POD

kubectl get storageclasses.storage.k8s.io

# Создаем deployment, PVC и таблицу
kubectl get persistentvolumeclaims

kubectl apply -f kube-config/postgres-pvc.yml        
kubectl apply -f kube-config/postgres-deployment.yml

kubectl port-forward pod/postgres-deployment-7bb79b5cd-q8wxp 5432:5432
```
```sql
create table "Link" (
	"id" serial not null,
	"url" text not null,
	"hash" text not null,
	
	constraint "Link_pkey" primary key ("id") 
);
```
```bash
# Создание секрета
kubectl create secret generic pg-secret --from-literal PASSWORD=my_pass
kubectl get secrets
kubectl describe decret pg-secret

# Как декодировать секрет?
kubectl get secrets pg-secret --template={{.data.PASSWORD}} | base64 -d

# Кодируем секрет в base64
echo -n demo | base64

kubectl apply -f ./kube-config/postgres-secret.yml

echo -n postgresql://demo:demo@postgres-clusterip:5432/demo | base64

# Применяем API
kubectl apply -f ./kube-config/api-secret.yml
kubectl apply -f ./kube-config/api-service.yml 
kubectl apply -f ./kube-config/api-deployment.yml 

# Посмотреть логи kubectl logs pods/postgres-deployment-7bb79b5cd-q8wxp    

# Добавляем маршрут в ingress до api
kubectl apply -f ./kube-config/app-deployment.yml 
kubectl apply -f ./kube-config/ingress.yml
```

### Эксплуатация
```bash
# Проваливаемся в терминао контейнера
kubectl exec -it pod/short-api-deployment-5c74fd6c49-25kgd -- /bin/bash

# Смотрит историю ревизий и делаем откат
kubectl rollout history deployment short-api-deployment
kubectl rollout undo deployment --to-revision=2

# Следим за подами
kubectl get po --watch

# Health Check
# $? - показать результат выполнения предыдущей команды

# Автокомплит kubectl в Zsh
# Добавляем .zshrc
# source <(kubectl completion zsh)
omz reload
```

```bash
# HELM - менеджер пакетов для Kubernetes
# Первым делом добавляем стандартный репозиторий helm
helm repo add stable https://charts.helm.sh/stable
helm repo list
helm repo update
# Поиск чартов
helm search repo mysql

# Устанавливаем чарт
helm install stable/mysql --generate-name
# Смотрим что в составе чарта
helm show chart stable/mysql
helm show all stable/mysql
```