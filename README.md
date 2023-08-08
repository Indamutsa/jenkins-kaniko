# Running Jenkins inside of kubernetes: ADVANCED WAY

Let us create a cluster with kind utility \
`kind create cluster --name jenkins`

Let us create a namespace for jenkins \
`kubectl create namespace jenkins`

Install jenkins using local-jenkins folder \
`kubectl apply -f local-jenkins/`

Inside here we install:

- The persistence volumes
- The persistence volume claims
- The jenkins deployment
- The jenkins service
- Jenkins role based access control, which has a service account, role and role binding

After that we create a secret for docker registry \

```
kubectl create secret docker-registry regcred -n jenkins \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=****** \
  --docker-password=****** \
  --docker-email=***********
```

Let us expose the jenkins service to be able to access it from browser by port forwarding \
`kubectl port-forward service/jenkins 8080:8080 --namespace jenkins`

Access jenkins from browser \
`http://localhost:8080`

Get the jenkins pod name \
`kubectl get pods -n jenkins`

Get admin password \
`kubectl exec -it jenkins-pod-name -n jenkins -- cat /var/jenkins_home/secrets/initialAdminPassword`

Once inside:

- Install suggested plugins
- Create admin user and fill the form
- Click on `Start using Jenkins`

## Install kubernetes plugin

1. Head to Manage Jenkins > Manage Plugins > Available
2. Search for `Kubernetes`
3. Install `Kubernetes plugin`
4. Head to Manage Jenkins > Configure System
5. Scroll down to `Cloud` section
6. Click on `Add a new cloud` > `Kubernetes`
7. Click on details and fill the form

- Name: `kubernetes`
- Kubernetes URL: `https://kubernetes.default:443` or `https://kubernetes.default.svc.cluster.local`
- Kubernetes Namespace: `jenkins`
- Jenkins URL: `http://jenkins:8080`
- Jenkins Tunnel: `jenkins:50000`
- Disable SSL verification: `checked`
- Add authorization using a service account

8. Click on `Test Connection` and make sure it is successful

## Install blue ocean plugin

Go back and install blue ocean plugin \
`Manage Jenkins > Manage Plugins > Available > Search for blue ocean > Install`

## Create kubeconfig secret

1. System configuration > Global credentials > Add credentials
2. Kind: `Secret file`
3. Scope: `Global, Jenkins, and so on`
4. Secret file: `~/.kube/config`
5. ID: `kubeconfig`
6. Description: `kubeconfig`
7. Click on `OK`

## Create a pipeline

1. Click on `New Item`
2. Enter an item name: `pipeline`
3. Select `Pipeline`
4. Click on `OK`
5. Scroll down to `Pipeline` section
6. Select `Pipeline script from SCM`
7. Select `Git`
8. Repository URL: `https://github.com/Indamutsa/jenkins-kaniko.git`

Make sure the branch is set to `*/master` and jenkinsfile is set to `Jenkinsfile` at the first level in the folder.

9. Click on `Save`

## Run the pipeline

1. Click on `Build Now`
2. Click on build `#1`
3. Click on `Console Output` to see the logs

In the meantime build executor is running, we can see the logs in the console. We can set the number of executors in the jenkins dashboard as well.
