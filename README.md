# Dataservices:<br>{Big,Fast,Smart} Data Processing with Microservices

This repository contains the showcase for my talk at data2day 2018 in Heidelberg.
For details see: https://www.data2day.de/veranstaltung-7266-dataservices%3A-%7Bbig%2Cfast%2Csmart%7D-data-processing-mit-microservices.html?id=7266

## Demo

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

$ kubectl create -f database/cockroachdb-statefulset.yaml
$ kubectl get pods
$ kubectl get persistentvolumes

$ kubectl create -f database/cluster-init.yaml
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
$ kubectl apply -f database/gce-ssd-storageclass.yaml

$ gcloud compute disks create --size 10GB --type pd-ssd pd-ssd-disk-1
$ gcloud compute disks create --size 10GB --type pd-ssd pd-ssd-disk-2
$ gcloud compute disks create --size 10GB --type pd-ssd pd-ssd-disk-3
$ gcloud compute disks list

$ kubectl apply -f database/gce-ssd-persistentvolumes.yaml
$ kubectl get persistentvolumes

$ TMPFILE=$(mktemp)
$ /usr/bin/openssl rand -base64 741 > $TMPFILE
$ kubectl create secret generic shared-bootstrap-data –from file=internal-auth-mongodb-keyfile=$TMPFILE
$ rm $TMPFILE

$ kubectl apply -f database/mongodb-statefulset.yaml
$ kubectl get all

$ open http://pauldone.blogspot.com/2017/06/deploying-mongodb-on-kubernetes-gke25.html
```
