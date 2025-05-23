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
# Автокомплит Helm в Zsh
# Добавляем .zshrc
# source <(kubectl completion zsh)
# source <(helm completion zsh)
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

# Создаем каркас чарта
helm create short-service

# Установка релиза
helm install short-service-release short-service 

# Release - информация о релизе
# Values - переменные из values.yml
# Chart - данные из Chart.yml
# Files - доступ к любым файлам
# Capabilities - информация о кластере
# Tempalte - информация о текущем шаблоне

# Инфо о релизах
helm ls

# Обновление релиза
# Дебаг с просмотром полученных yaml манифестов
helm install --debug --dry-run short-service-release ./short-service 

# Переопределение
helm install --debug --dry-run short-service-release ./short-service --set name=Vasia

# Обновление
helm upgrade short-service-release ./short-service

# Упаковываем чарт и индексируем
helm package ../kube-config/short-service/
helm repo index .

# Использование репозитория
helm repo add superhero86 https://raw.githubusercontent.com/superhero86rus/helm-repo/refs/heads/main

# Сопровождение релиза
# Удаляем все из k8s
helm uninstall short-service-release 

# Установка из репозитория (не забыть обновить его локально)
helm install short-service-release superhero86/short-service

# Установка определенной версии
helm install short-service-release superhero86/short-service --version="1.0.2"

# История
helm history short-service-release

# Откат до первой ревизии
helm rollback short-service-release 1

# Тесты
helm lint short-service
# Превью как при --debug --dry-run
helm template test ../kube-config/short-service
# Полный рендер
helm get all short-service-release

# Тесты
# Создаем отдельный под, который будет делать запросы на наш api
# Создаем отдельную папку tests внутри templates нашего чарта
# Обновляем чарт
helm upgrade short-service-release ./short-service

helm test short-service-release

# Пушим обновление в GitHub, обновляем репозитории и устанавливаем
helm repo update
```

### Секреты
```bash
# Используем плагин helm-secrets
helm plugin install https://github.com/jkroepke/helm-secrets

# Генерируем приватный ключ, для подписи секретов
gpg --gen-key

# Проверяем
gpg --list-keys

# Устанавливаем Mozilla Sops
wget https://github.com/mozilla/sops/releases/download/v3.7.1/sops_3.7.1_amd64.deb
sudo apt install ./sops_3.7.1_amd64.deb

# Создаем secrets.yml (открывать файл с паролем который мы задали при создании файла)
sops -p <GPG_KEY> secrets.yml
helm secrets decrypt
helm secrets edit

# Преобразование файла
helm secrets decrypt -i secrets.yml
helm secrets encrypt -i secrets.yml

# Чтобы шифрование и расшифровка работали без проблем - нужно создать sops файл и вставить в него ключ gpg (.sops.yaml)
# Расширение yaml (не yml)
helm secrets upgrade short-service-release ./short-service -f secrets.yml
kubectl get secrets short-api-secret --template={{.data.database-url}} | base64 -d

```