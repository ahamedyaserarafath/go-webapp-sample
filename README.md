# go-webapp-sample

## Preface

Sample go webapp application forked from 

This repository is the sample of web application using golang forked from -> -> https://github.com/ybkuroki/go-webapp-sample

On top of that added the below items
1. Dockerfile to build the docker image and push it to dockerhub
2. Helm chart to deploy the postgres and redis database which inturn used for webserver.
3. Helm chart to deploy the web app application without the webserver 
4. Deploy the letzencrypt ssl with subdomain.
5. Added the terraform code to deploy the standalone k8s(v19.2.0) in EC2 instance with single master.



## Starting Without Web Server locally
1. Starting this web application by the following command.
    ```bash
    go run main.go
    ```
2. When startup is complete, the console shows the following message:
    ```
    http server started on [::]:8080
    ```
3. Access [http://localhost:8080](http://localhost:8080) in your browser.
4. Login with the following username and password.
    - username : ``test``
    - password : ``test``

### Dockerfile and steps to build postgres
Below are the steps to build docker image and push to docker hub
please replace the tagname w.r.t to your docker image

```
docker build -f DockerfilePostgres . --tag=ahamedyaserarafath/postgres-sample:latest
docker push ahamedyaserarafath/postgres-sample:latest
```

Or use the exsisting the docker image
```
https://hub.docker.com/repository/docker/ahamedyaserarafath/postgres-sample
```


### Dockerfile and steps to build webapp

Below are the steps to build docker image and push to docker hub
please replace the tagname w.r.t to your docker image

```
docker build . --tag=ahamedyaserarafath/go-webapp-sample:latest
docker push ahamedyaserarafath/go-webapp-sample:latest
```

Or use the exsisting the docker image
```
https://hub.docker.com/repository/docker/ahamedyaserarafath/go-webapp-sample
```

### Steps to deploy EC2 instance with terraform

Prerequisite -> Terraform needs to be installed 

1. Configure the machine with aws access using the below command
```
aws configure
```
2. Execute the below command to install the EC2 instance
```
cd infra/terraform/
./run.sh
```
Note: 
The above terraform code will create New VPC, EC2 Instance(t2.medium), EC2 SG and Install the k8s with single master 
SSH key and Kubeconfig will be found in the same folder.


### Steps to deploy Helm chart

Prerequisite -> Make sure kubectl is installed on your local machine and configure the kube config on your machine
Alternative you can login to the EC2 machine which created in the above steps and follow the below steps

1. Install the postgres and redis
```
cd infra/helm-charts
helm install postgres-database ./postgres-database
helm install redis ./redis
```

2. Install the go-webapp helm chart using the below command

```
cd infra/helm-charts
helm install go-webapp ./go-webapp
```

3. Confirm that the go webapp Pods have started:
```
kubectl get pods -w
```

4. Setting Up the Kubernetes Nginx Ingress Controller
```
cd infra/helm-charts/ingress-nginx/
kubectl apply -f ingress_controler.yaml
```
You should see the following output:

Output
```
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
```
Confirm that the Ingress Controller Pods have started:
```
kubectl get pods -n ingress-nginx \
  -l app.kubernetes.io/name=ingress-nginx --watch
```

5. Setting Up the Kubernetes Nginx Ingress Resource(HTTPS)
```
cd infra/helm-charts/ingress-nginx/
kubectl apply -f https_ingress_letzencrypt.yaml
```

6. Installing and Configuring Letzencrypt Cert-Manager
```
kubectl apply --validate=false -f cert-manager.yaml
```
To verify our installation, check the cert-manager Namespace for running pods:

```
kubectl get pods --namespace cert-manager
```

You can use kubectl describe to track the state of the Ingress changes you’ve just applied:

```
kubectl describe ingress
```

Once the certificate has been successfully created, you can run a describe on it to further confirm its successful creation:

```
kubectl describe certificate
```

## License
The License of this sample is *MIT License*.
