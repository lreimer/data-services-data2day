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
```

### Building Block 2: Persistent Storage

We are going to use Cockroach DB as relational, persistent storage. Basically, follow the
instructions here: https://www.cockroachlabs.com/docs/stable/orchestrate-cockroachdb-with-kubernetes-insecure.html

```
$ kubectl create clusterrolebinding $USER-cluster-admin-binding --clusterrole=cluster-admin --user=mario-leander.reimer@qaware.de

$ kubectl create -f storage/cockroachdb-statefulset.yaml
$ kubectl get pods
$ kubectl get persistentvolumes

$ kubectl create -f storage/cluster-init.yaml
$ kubectl get job cluster-init
$ kubectl get pods
```

Test the cluster:
```
$ kubectl port-forward cockroachdb-0 8080

$ kubectl run cockroachdb -it --image=cockroachdb/cockroach --rm --restart=Never -- sql --insecure --host=cockroachdb-public
```
