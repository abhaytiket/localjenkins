# Monorepo CD/CD setup

## Requirements

As I'm already using docker desktop for macOS and it has in built support for kubernetes hence, I'll be using it for deploying the setup. In this initial setup we are going to use 
* a sample golang monorepo on GitHub
* a GitHub app for integrating Jenkins controller with GitHub
* Jenkins controller for continuous integration
* a linux agent with bazel installed on it 
* Argo CD for continuous deployment
* a local kubernetes cluster using docker desktop  

## Argo CD setup instructions

1. Enable kubernetes on docker desktop and wait for the cluster to be ready.
2. Install `kubectl` using homebrew command
```
$ brew install kubectl
```
3. Once the status of cluster on docker desktop dashboard has turned green then go to your terminal and change context using following commands 
```
$ kubectl config get-contexts
$ kubectl config use-context docker-desktop
```
4. Install Argo CD in your k8s cluster using following commands
```
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
``` 
This will create a new namespace `argocd`` where all Argo CD services and application resources will live.

5. To use Argo CD cli install it using homebrew by using following command
```
$ brew install argocd
```
6. To access the `argocd-server` I have changed the service type to `NodePort` by using following command
```
$ kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```
7. To get the http port for argocd use following command
```
$ kubectl get svc -n argocd
``` 
The output for this command will be quite similar to following
```
NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   10.109.176.37    <none>        7000/TCP,8080/TCP            47h
argocd-dex-server                         ClusterIP   10.111.41.167    <none>        5556/TCP,5557/TCP,5558/TCP   47h
argocd-metrics                            ClusterIP   10.101.45.255    <none>        8082/TCP                     47h
argocd-notifications-controller-metrics   ClusterIP   10.101.140.74    <none>        9001/TCP                     47h
argocd-redis                              ClusterIP   10.109.91.132    <none>        6379/TCP                     47h
argocd-repo-server                        ClusterIP   10.98.84.232     <none>        8081/TCP,8084/TCP            47h
argocd-server                             NodePort    10.104.114.205   <none>        80:31329/TCP,443:30966/TCP   47h
argocd-server-metrics                     ClusterIP   10.109.46.229    <none>        8083/TCP                     47h
```
So based on the output above we can use port `31329` for accessing Argo CD UI at `http://localhost:31329`

8. On opening Argo CD UI on your browser use username as `admin` and password can be found using following command on your terminal
```
$kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
Another way to get the initial password is by using CLI as follows
```
$ argocd admin initial-password -n argocd
```
9. You can also login using Argo CD CLI tool by following command
```
$ argocd login localhost:31329
```
10. You can also change the password by using following command
```
$ argocd account update-password
```
11. Now we need to register a cluster to deploy apps to. To do that first get the list of all cluster configs in your current kubeconfig using following command
```
$ kubectl config get-contexts -o name
```
Choose a context name from the list and supply it to following command
```
$ argocd cluster add CONTEXTNAME
```
As I'm going to use `docker-desktop` hence I have used 
```
$ argocd cluster add docker-desktop
```
The above command installs a ServiceAccount (`argocd-manager`), into the kube-system namespace of that kubectl context, and binds the service account to an admin-level ClusterRole. Argo CD uses this service account token to perform its management tasks (i.e. deploy/monitoring).

## Jenkins setup instructions
For setting up Jenkins on k8s we will do the following:
* Create a Namespace
* Create a service account with k8s admin permissions
* Create local persistent volume for persistent Jenkins data on Pod restarts
* Create a deployment YAML and deploy it
* Create a service YAML and deploy it

Please just follow the instructions given in [link](https://www.jenkins.io/doc/book/installing/kubernetes/)

