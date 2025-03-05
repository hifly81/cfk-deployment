Confluent For Kubernetes deployment with ArgoCD. 
Easily install and sync Confluent Platform components (kraft, kafka, schema registry, connect, ksql, control center) and resources (topics, acls, rbac, schemas, connector)

## Quickstart

### Requirements 

#### Docker Desktop 

You need _docker desktop_ installed on your host to run the quickstart: https://docs.docker.com/desktop

#### Minikube

You need _Minikube_ on your host. 

If you are on Linux follow instructions for ArchLinux (also tested with Fedora):
https://dev.to/xs/kubernetes-minikube-with-qemu-kvm-on-arch-312a

Minikube installation in Mac/Windows
https://minikube.sigs.k8s.io/docs/start/

#### argocd CLI

Install _argocd_ cli: 

```
$ brew install argocd
```

#### kubectl CLI

Install _kubectl_:

```
$ brew install kubectl
```

#### helm

Install _helm_:

```
$ brew install kubernetes-helm
$ helm init
```


### Start Minikube

Set _docker_ driver and reserve 16GB RAM and 4 CPUs to Minikube:

```
$ minikube delete
$ minikube config set driver docker

$ touch /tmp/config && export KUBECONFIG=/tmp/config
$ minikube start --memory 16384 --cpus 4
```

### Install ArgoCD Server in Minikube

Install ArgoCD on _argocd_ namespace and expose it on port 8080:

```
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

When state is _Running_, expose on port 8080:

```
$ kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Install CFK in Minikube

Create a namespace named _confluent_:

```
$ kubectl create namespace confluent
$ kubectl config set-context --current --namespace confluent
```

Add confluent repository to _helm_:

```
$ helm repo add confluentinc https://packages.confluent.io/helm
$ helm repo update
```

Install _confluent-for-kubernetes_ operator (latest version) from Confluent’s Helm repo:

```
$ helm upgrade --install confluent-operator confluentinc/confluent-for-kubernetes --set kRaftEnabled=true
```

### Deploy CP Platform using ArgoCD

Obtain Password to login to ArgoCD:

```
$ kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Login to ArgoCD:

```
argocd login localhost:8080 --username admin --password <your-password>
```

Deploy cp-platform-app on _confluent_ namespace for _dev_ environment:

```
argocd app create cp-platform-app \
--repo https://github.com/hifly81/cfk-deployment \
--path confluent-platform/environments/dev \
--dest-server https://kubernetes.default.svc \
--dest-namespace confluent \
--sync-policy manual
```

### Apply changes on Confluent Resources using ArgoCD

At the moment, these are the resources that can be managed:
 - Kafka Topics
 - Kafka Standard ACLs on Kraft

Obtain Password to login to ArgoCD:

```
$ kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Login to ArgoCD:

```
$ argocd login localhost:8080 --username admin --password <your-password>
```

Apply configuration-app on _confluent_ namespace for _dev_ environment:

```
$ argocd app create configuration-app \
--repo https://github.com/hifly81/cfk-deployment \
--path configuration/environments/dev \
--dest-server https://kubernetes.default.svc \
--dest-namespace confluent \
--sync-policy manual
```

### Teardown

```
$ minikube delete
```
