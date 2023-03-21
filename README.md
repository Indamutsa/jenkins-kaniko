This command create jennkins as a docker container \
`docker run --rm -d -u root -p 8080:8080 --name jenkins -v jenkins-data:/var/jenkins_home -v $(which docker):/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock -v "$HOME":/home jenkins/jenkins:lts`

This command create kubernetes using kind, with a worker node and api server address \

```
kind create cluster --name jenkins --config <(cat <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
apiServerAddress: "192.168.0.19" # replace with your API server IP address
apiServerPort: 8443
nodes:
- role: control-plane
- role: worker
EOF
)
```

---

Next we create a jenkins credentils using a secret a file where we choose config from .kube/config file

Let us make a docker secret to be able to pull images from docker hub

```
kubectl create secret docker-registry regcred -n jenkins \
  --docker-server=https://index.docker.io/v2 \
  --docker-username=indamutsa \
  --docker-password=arsene \
  --docker-email=arsichizy@gmail.com
```

Have the logs live in the console

```
k logs pod/jenkins-0  -n jenkins --follow
```

---

HOW TO INSTALL JENKINS IN KUBERNETES USING HELM

```
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
helm install jenkins --namespace jenkins --create-namespace jenkinsci/jenkins
```

How to install jenkins in Kubernetes using helm:

```
mkdir jenkins
cd jenkins
helm repo add jenkinsci https://charts.jenkins.io

helm repo update
gh repo view jenkinsci/helm-charts --web
```

Port fortward jenkins service to access it from browser

```
kubectl port-forward service/jenkins 8080:8080 --namespace jenkins
```

Access jenkins from browser

`http://localhost:8080`

Get admin password

```
kubectl get secret --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode; echo
```

The username is admin and the password is the one you got from the previous command.

You can also get the secret and then extract the password from it:

```
kubectl get secret --namespace jenkins jenkins -o yaml
```

Get the password from the output of the previous command and then decode it:

```
echo <password> | base64 --decode
```

Let us download the values file for jenkins

```
helm show values jenkinsci/jenkins > values.yaml
```

I added the following to the values.yaml file

```
  installPlugins:
    - kubernetes:3734.v562b_b_a_627ea_c
    - workflow-aggregator:590.v6a_d052e5a_a_b_5
    - git:4.13.0
    - configuration-as-code:1569.vb_72405b_80249
    - blueocean:1.27.3
    - ansicolor:1.0.2

```

I also uncommented random password generation to disable and i set it by my own!

And then we run it this way

```
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
helm install jenkins --values values.yaml --namespace jenkins --create-namespace jenkinsci/jenkins

```

Output

```
1. Get your 'arsene' user password by running:
  kubectl exec --namespace jenkins -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  echo http://127.0.0.1:8080
  kubectl --namespace jenkins port-forward svc/jenkins 8080:8080
```

Uninstall:

```
helm uninstall jenkins --namespace jenkins
```
