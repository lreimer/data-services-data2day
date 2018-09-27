# Dataservices:<br>{Big,Fast,Smart} Data Processing with Microservices

This repository contains the showcase for my talk at data2day 2018 in Heidelberg.
For details see: https://www.data2day.de/veranstaltung-7266-dataservices%3A-%7Bbig%2Cfast%2Csmart%7D-data-processing-mit-microservices.html?id=7266

## Local Demo

It is possible to launch the complete demo locally using Docker Compose. Make sure Docker can use at least
4 GB of RAM. Then do:
```
$ docker-compose up -d
```

## Live Demo

### Building Block 1: Cloud-native Platform

- Install the gcloud SDK for Mac (or Windows)
- Make sure you have a project with billing activated as well as the container engine management API
- Make sure you have `kubectl` installed, either using `gcloud` or using `brew` et.al.

```
$ gcloud config list project
$ gcloud config set compute/zone europe-west1-b
$ gcloud config set container/use_client_certificate False

$ gcloud container clusters create data2day-services --num-nodes=5 --enable-autoscaling --min-nodes=5 --max-nodes=7

$ gcloud container clusters describe data2day-services

$ gcloud auth application-default login
$ kubectl cluster-info

$ kubectl config view -o jsonpath="{.users[?(@.name == \"$(kubectl config current-context)\")].user.auth-provider.config.access-token}"
$ gcloud config config-helper --format=json | jq .credential.access_token
$ open http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

### Building Block 2: Storage and Database

We are going to use Cockroach DB as relational, persistent storage. Basically, follow the
instructions here: https://www.cockroachlabs.com/docs/stable/orchestrate-cockroachdb-with-kubernetes-insecure.html

```
$ kubectl create clusterrolebinding $USER-cluster-admin-binding --clusterrole=cluster-admin --user=mario-leander.reimer@qaware.de

$ kubectl create -f database/cockroachdb/cockroachdb-statefulset.yaml
$ kubectl get pods
$ kubectl get persistentvolumes

$ kubectl create -f database/cockroachdb/cockroachdb-service.yaml
$ kubectl create -f database/cockroachdb/cluster-init.yaml
$ kubectl get job cluster-init
$ kubectl get pods
```

Test the cluster:
```
$ kubectl port-forward cockroachdb-0 8080

$ kubectl run cockroachdb -it --image=cockroachdb/cockroach --rm --restart=Never -- sql --insecure --host=cockroachdb-public
```

Alternatively, we will create a Mongo DB cluster and storage.

```
$ kubectl apply -f database/mongodb/gce-ssd-storageclass.yaml

$ gcloud compute disks create --size 10GB --type pd-ssd pd-ssd-disk-1
$ gcloud compute disks create --size 10GB --type pd-ssd pd-ssd-disk-2
$ gcloud compute disks create --size 10GB --type pd-ssd pd-ssd-disk-3
$ gcloud compute disks list

$ kubectl apply -f database/mongodb/gce-ssd-persistentvolumes.yaml
$ kubectl get persistentvolumes

$ TMPFILE=$(mktemp)
$ /usr/bin/openssl rand -base64 741 > $TMPFILE
$ kubectl create secret generic shared-bootstrap-data –from file=internal-auth-mongodb-keyfile=$TMPFILE
$ rm $TMPFILE

$ kubectl apply -f database/mongodb/mongodb-statefulset.yaml
$ kubectl get all

$ open http://pauldone.blogspot.com/2017/06/deploying-mongodb-on-kubernetes-gke25.html
```

### Building Block 3: Messaging Infrastructure

Next, we will deploy different messaging systems for our communication infrastructure. As possible
input source we will deploy Eclipse Mosquitto, and as central messaging fabric we are going to use
Apache Artemis.

```
$ kubectl apply -f messaging/mosquitto/
$ kubectl get pods

$ kubectl apply -f messaging/artemis/
$ kubectl get pods

$ kubectl port-forward message-queue-785589d777-gqszs 8161
$ open http://localhost:8161/
```

Also have a look at the Artemis Helm char found here: https://github.com/vromero/activemq-artemis-helm

### Building Block 4: Data Grid

We are going to use a Hazelcast IMDG embedded into a Payara Micro instance. To deploy the services
and pods issue the following commands:

```
$ kubectl apply -f datagrid/hazelcast/
$ kubectl get pods
$ kubectl scale deployment hazelcast-payara --replicas=5

$ open http://localhost:8001/api/v1/namespaces/default/services/http:hazelcast-payara:8080/proxy/api/application.wadl
```

### Building Block 5: Dataservices

Next we deploy our input, processor and output dataservices. The input here is
the OpenWeather API, the processor stores current weather in IMDG and forwards
the data to file and RDBMS.
```
$ kubectl apply -f dataservices/input/weather/
$ kubectl apply -f dataservices/processors/weather/
$ kubectl apply -f dataservices/sink/weather-file/
$ kubectl apply -f dataservices/sink/weather-rdbms/
```

These here simmulate car data flowing in from MQTT, database and CSV. The location
is processed and put in the IMDG.
```
$ kubectl apply -f dataservices/input/csv/
$ kubectl apply -f dataservices/input/database/
$ kubectl apply -f dataservices/input/mqtt/
$ kubectl apply -f dataservices/processors/location/
```

Once you are done, remember to delete the cluster again!
```
$ gcloud container clusters delete data2day-services
```
