# Python Database Example
This example demonstrates how to use headless and clusterip services with StatefulSets

## Generate pyc file
```
python3 -m compileall app.py 
cp __pycache__/*.pyc app.pyc
```
## Dockerize the application
```
docker build -t kunchalavikram/python_todo_app:v2 .
docker push kunchalavikram/python_todo_app:v2
```

## Start Minikube Cluster
```
minikube start --driver=virtualbox --cpus=2 --memory=4G
```

## Demo 00 : Using Deployments
- Deploy the application and MySQL Deployment with 1 replica
```
kubectl apply -f mysql_deployment.yml
kubectl apply -f app_deployment.yml
```
- Scale the MySQL Deployment
```
kubectl scale deploy mysql --replicas=3
```
- Test the application from UI and also by connecting to each DB
```
kubectl get po -l "app=db" -o wide

kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --  mysql -h 172.17.0.3 -uroot -proot todo -e "SELECT * from todo;"

kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-normal -uroot -proot todo -e "SELECT * from todo;"

(-e, --execute=name -> Execute command and quit)
```

## Demo 01 : Using StatefulSets
- First test with MySQL ClusterIP service with One replica
- Increase the replicas
```
kubectl apply -f mysql_statefulset.yml
kubectl apply -f app_deployment.yml  
kubectl scale sts mysql --replicas=3
```
- Change to MySQL Headless service and check.
- Increase the replicas.
- Now change the URI to point to first instance of StatefulSets: Use DNS as mysql-0.mysql-headless.
- Check individual DBs.
```
kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-0.mysql-headless -uroot -proot todo -e "SELECT * from todo;"

```

## Demo 02: Using MySQL Replication
- Download Helm chart
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm pull bitnami/mysql --untar
```
C:\Users\kunch\Downloads\mysql
C:\Users\kunch\OneDrive\Study\DevOps\Kubernetes\Templates\my_manifests\statefulset\python_todo_app\kubernetes\02\mysql

- Modify PVC size, enable replication, reduce resources etc
- Add init db initdbScripts(https://docs.bitnami.com/kubernetes/infrastructure/mysql/configuration/customize-new-instance/)
```
initdbScripts:
  my_init_script.sh: |
    #!/bin/bash
    mysql -P 3306 -uroot -proot todo -e "
    CREATE DATABASE IF NOT EXISTS todo;
    USE todo;
    CREATE TABLE todo (id INT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY, title VARCHAR(100), complete BOOLEAN);"
```
- Install the Helm chart
```
vikram.a.kunchala$ helm install mysql mysql/
NAME: mysql
LAST DEPLOYED: 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mysql
CHART VERSION: 9.1.1
APP VERSION: 8.0.29

** Please be patient while the chart is being deployed **

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace default

Services:

  echo Primary: mysql-primary.default.svc.cluster.local:3306
  echo Secondary: mysql-secondary.default.svc.cluster.local:3306

Execute the following to get the administrator credentials:

  echo Username: root
  MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run mysql-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mysql:8.0.29-debian-10-r23 --namespace default --env MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD --command -- bash

  2. To connect to primary service (read/write):

      mysql -h mysql-primary.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"

  3. To connect to secondary service (read-only):

      mysql -h mysql-secondary.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"
```

- Check individual DBs.
```
kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-primary -uroot -proot todo -e "SELECT * from todo;"

kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-primary -uroot -proot todo -e "SHOW tables;"

kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-secondary -uroot -proot todo -e "SHOW tables;"

kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-secondary-0.mysql-secondary -uroot -proot todo -e "SHOW tables;"

kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-secondary-0.mysql-secondary -uroot -proot todo -e "SELECT * from todo;"

```


## Misc
```
docker run -d --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=todo mysql:latest
```