# hello-kube project

The original hello project has been generated by the lagom/lagom-scala.g8 template. Kubernetes configuration has been added to show how a Lagom project can be deployed in a kubernetes cluster.

## Running: Kubernetes

Follow the steps below to install Lagom application in your own local [Kubernetes](https://kubernetes.io/) environment.

## Setup

You'll need to ensure that the following software is installed:

* [Docker](https://www.docker.com/) v19.03.2 (Verify with `docker version`)
* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl) Client Version: v1.16.0, Server Version: v1.13.3 (Verify with `kubectl version --short`)
* [Minikube](https://github.com/kubernetes/minikube) v1.3.1 (Verify with `minikube version`)

## Operations Workflow

This workflow matches how an application would be deployed in a production environment. In this method,
the images are built by SBT. The steps below focus on Minikube, but could be used on
a real Kubernetes cluster given access to a Docker registry and configured `kubectl`.

> Want a fresh Minikube? Run `$ minikube delete` before the steps below.

### Start Minikube and setup Docker to point to Minikube's Docker Engine.<br>
```shell
$ (minikube delete || true) &>/dev/null && minikube start --cpus 4 --memory 8192

 --extra-config=apiserver.authorization-mode=RBAC


$ minikube addons enable ingress
$ kubectl apply -f kubernetes/discovery-role.yaml
$ eval $(minikube docker-env)
```
To check if ingress controller enabled:<br>
```bash
$ kubectl get pods -n kube-system | grep ingress
nginx-ingress-controller-57bf9855c8-pt7rc   1/1     Running   0          4m48s
```

> Make sure you run `$ eval $(minikube docker-env)` before following the steps below. When you need your old environment, you can get it back with `eval $(minikube docker-env -u)`.

### Create and publish the Docker container with the services.
You can use your preferred method to do that, by example using the sbt docker plugin included in [sbt-native-packager](https://www.scala-sbt.org/sbt-native-packager/formats/docker.html).<br>
```shell
$ sbt docker:publishLocal
```

### Change the ip's for Kafka and Cassandra services and deploy them.<br>
```shell
$ kubectl apply -f kubernetes/cassandra-service.yaml
$ kubectl apply -f kubernetes/kafka-service.yaml
```

### Create the secret with play secret.<br>
```shell
$ kubectl create secret generic hello-application-secret --from-literal=secret="$(openssl rand -base64 48)"
```

### Deploy the microservices.<br>
```shell
$ kubectl apply -f kubernetes/hello-deployment.yaml
```

### Create Ingress.<br>
```shell
$ kubectl apply -f kubernetes/ingress.yaml
```