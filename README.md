# Infrastructure DevOps

### Docker-machine
- Для работы с GCP
```bash
export GOOGLE_PROJECT=devops-practice-test
```
- Создать VM для Postgres в GCE c docker-machine:
```bash
docker-machine create --driver google \
    --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
    --google-machine-type n1-standard-1 \
    --google-zone europe-west1-b \
    --google-open-port 5432/tcp \
    postgres
```
- Запустить докер контейнер Postgres на созданной VM
```bash
eval $(docker-machine env postgres)
docker run --name pg -p 5432:5432 \
    -v /root/pgdata:/var/lib/postgresql/data:Z \
    -e POSTGRES_DB=mydb -e POSTGRES_USER=dbowner \
    -e POSTGRES_PASSWORD=secret \
    -d postgres:alpine
```
- Создать VM с Kafka
```bash
docker-machine create --driver google \
    --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
    --google-machine-type n1-standard-1 \
    --google-zone europe-west1-b \
    --google-open-port 9092/tcp \
    --google-open-port 2181/tcp \
    kafka
eval $(docker-machine env kafka)
docker run -p 2181:2181 -p 9092:9092 \
    --env ADVERTISED_HOST=`docker-machine ip \`docker-machine active\`` \
    --env ADVERTISED_PORT=9092 \
    spotify/kafka
```
- Переключить docker-machine на локальный docker
```bash
eval $(docker-machine env --unset)
```

### Google Deployment Manager
- применить конфигурацию
```bash
gcloud deployment-manager deployments create postgres \
    --config postgres-vm.yml
```
- применить новые изменения
```bash
gcloud deployment-manager deployments update postgres
```

### GCP Console – gcloud
- create VM
```bash
gcloud compute instances create postgres-db \
    --boot-disk-size=20GB \
    --image-family ubuntu-1604-lts \
    --image-project=ubuntu-os-cloud \
    --machine-type=g1-small \
    --tags postgres-server \
    --restart-on-failure
gcloud compute instances create kafka \
    --boot-disk-size=20GB \
    --image-family ubuntu-1604-lts \
    --image-project=ubuntu-os-cloud \
    --machine-type=g1-small \
    --tags kafka-server \
    --restart-on-failure
```
- create firewall
```bash
gcloud compute firewall-rules create default-postgres-server \
    --allow=tcp:5432 \
    --source-ranges='0.0.0.0/0' \
    --target-tags postgres-server
gcloud compute firewall-rules create default-kafka-server \
    --allow=tcp:9092 \
    --source-ranges='0.0.0.0/0' \
    --target-tags kafka-server
```

### Ansible 
- install roles
```bash
ansible-galaxy install -r environments/qa/requirements.yml
```
- run playbook 
```bash
ansible-playbook site.yml --check
``` 

### Проверка работы
- Проверка сетевых настроек 
```bash
sudo lsof -i -P -n | grep LISTEN
```

- Проверка кафки
```bash
./kafka-console-producer \
    --broker-list 35.204.253.91:9092 \
    --topic sample-topic 
    
./kafka-console-consumer \
    --bootstrap-server 35.204.253.91:9092 \
    --topic sample-topic \
    --partition 0 \
    --from-beginning
```
