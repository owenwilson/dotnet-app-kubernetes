# dotnet-app-kubernetes

## setup

```sh
mkdir dotnet-app-kubernetes && cd dotnet-app-kubernetes
```

```sh
dotnet new globaljson --sdk-version 8.0.424
```

```sh
dotnet new mvc
```

## run project

```sh
dotnet run
```

- open browser [localhost:5110](http://localhost:5110)

## ci-cd

```sh
dotnet restore
```

- This flag prevents the dependencies from being restored again, assuming that `dotnet restore` was previously run.

```sh
dotnet build --configuration Release --no-restore
```

```sh
dotnet test --configuration Release --no-build --collect:"XPlat Code Coverage"
```

```sh
dotnet publish --configuration Release --no-build --output ./publish
```

## git remotes multiples

- default github

```sh
origin git@github.com:user/app-mvc.git (fetch)
origin git@github.com:user/app-mvc.git (push)
```

- add example configuration ssh alias

```sh
# azure devops ssh
Host ssh.dev.azure.com
    HostName ssh.dev.azure.com
    IdentityFile ~/.ssh/key_azure
    IdentitiesOnly yes
```

- add azure repos example

```sh
git remote add azure git@ssh.dev.azure.com:v3/user/app-mvc/app-mvc
```

```sh
git remote -v
```

- output

```sh
azure git@ssh.dev.azure.com:v3/user/app-mvc/app-mvc (fetch)
azure git@ssh.dev.azure.com:v3/user/app-mvc/app-mvc (push)
origin git@github.com:user/app-mvc.git (fetch)
origin git@github.com:user/app-mvc.git (push)
```

- test push azure

```sh
git push azure main
```

- git verbose

```sh
GIT_SSH_COMMAND="ssh -vvv" git push azure main
```

- git pull azure

```sh
git pull azure main
```

- check ou [use ssh keys to authenticate](https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops)

## aks configuration

```sh
az login
```

- get kube config
- **resource group:**`resource-name-cluster-kubernetes-azure-cloud`
- **cluster kubernetes name:**`name-cluster-kubernetes-azure-cloud`

```sh
az aks get-credentials --resource-group resource-name-cluster-kubernetes-azure-cloud --name name-cluster-kubernetes-azure-cloud
```

- output

```sh
Merged "name-cluster-kubernetes-azure-cloud" as current context in /home/user/.kube/config
```

- agentpool alert, need more resources
- migration agentpool to new agentpool
- use cordon and drain

```sh
kubectl get nodes
```

- output

```sh
kubectl get nodes
NAME                                STATUS                     ROLES    AGE   VERSION
aks-agentpool-XXXXXXXX-vmss000000   Ready   <none>   46h   v1.35.7
aks-agentpool2-XXXXXXXX-vms1        Ready                      <none>   16h   v1.35.7
```

- isolate pod, before migration and use kubectl cordon

```sh
kubectl cordon aks-agentpool-XXXXXXXX-vmss000000
```

- output

```sh
kubectl get nodes
NAME                                STATUS                     ROLES    AGE   VERSION
aks-agentpool-XXXXXXXX-vmss000000   Ready,SchedulingDisabled   <none>   46h   v1.35.7
aks-agentpool2-XXXXXXXX-vms1        Ready                      <none>   16h   v1.35.7
```

- use drain for migration complete

```sh
kubectl drain aks-agentpool-XXXXXXXX-vmss000000 --ignore-daemonsets --delete-emptydir-data
```

- Check it in the Azure web interface

- After running the “drain kubernetes” command, perform the migration to the agent pool that is in the “ready” state. The migration

## error limit cpu

- get all namespace

```sh
kubectl get all -n dotnet-app-cluster
NAME                          TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
service/dotnetappkubernetes   LoadBalancer   X.X.X.X        X.X.X.X   80:31854/TCP   23h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dotnetappkubernetes   0/1     0            0           23h

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/dotnetappkubernetes-XXXXXXXXXX   0         0         0       21h
replicaset.apps/dotnetappkubernetes-XXXXXXXXXX   1         0         0       8m32s
replicaset.apps/dotnetappkubernetes-XXXXXXXXXX   1         0         0       70m
replicaset.apps/dotnetappkubernetes-XXXXXXXXXX    0         0         0       22h
replicaset.apps/dotnetappkubernetes-XXXXXXXXXX   0         0         0       23h
replicaset.apps/dotnetappkubernetes-XXXXXXXXXX   0         0         0       20h
replicaset.apps/dotnetappkubernetes-XXXXXXXXXX    0         0         0       23h
```

- example get deployment dotnetappkubernetes

```sh
kubectl get deployment dotnetappkubernetes -n dotnet-app-cluster -o jsonpath='{.status.conditions[*].message}'
Deployment does not have minimum availability. ReplicaSet "dotnetappkubernetes-XXXXXXXX" has timed out progressing.
```

- example get deployment dotnetappkubernetes

```sh
kubectl get deployment dotnetappkubernetes -n dotnet-app-cluster -o jsonpath='{.status.conditions[*].message}'
Deployment does not have minimum availability. pods "dotnetappkubernetes-XXXXXXXX-hqkmz" is forbidden: failed quota: defaultresourcequota: must specify limits.cpu for: dotnetappkubernetes; limits.memory for: dotnetappkubernetes; requests.cpu for: dotnetappkubernetes; requests.memory for: dotnetappkubernetes ReplicaSet "dotnetappkubernetes-XXXXXXXXX" has timed out progressing.
```

```sh
kubectl get rs -n dotnet-app-cluster
NAME                             DESIRED   CURRENT   READY   AGE
dotnetappkubernetes-XXXXXXXXXX   0         0         0       21h
dotnetappkubernetes-XXXXXXXXXX   1         0         0       13m
dotnetappkubernetes-XXXXXXXXXX   1         0         0       75m
dotnetappkubernetes-XXXXXXXXXX    0         0         0       22h
dotnetappkubernetes-XXXXXXXXXX   0         0         0       23h
dotnetappkubernetes-XXXXXXXXXX   0         0         0       20h
dotnetappkubernetes-XXXXXXXXXX    0         0         0       23h
```

- get svc

```sh
kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.0.0.1     X.X.X.X        443/TCP   2d2h
```

- get events

```sh
kubectl get events -n dotnet-app-cluster --sort-by='.lastTimestamp' | tail -20
43m         Warning   FailedCreate           replicaset/dotnetappkubernetes-XXXXXXXXXX   Error creating: pods "dotnetappkubernetes-XXXXXXXXXX-XXXXX" is forbidden: failed quota: defaultresourcequota: must specify limits.cpu for: dotnetappkubernetes; limits.memory for: dotnetappkubernetes; requests.cpu for: dotnetappkubernetes; requests.memory for: dotnetappkubernetes
41m         Warning   FailedCreate           replicaset/dotnetappkubernetes-XXXXXXXXXX   (combined from similar events): Error creating: pods "dotnetappkubernetes-XXXXXXXXXX-XXXXX" is forbidden: failed quota: defaultresourcequota: must specify limits.cpu for: dotnetappkubernetes; limits.memory for: dotnetappkubernetes; requests.cpu for: dotnetappkubernetes; requests.memory for: dotnetappkubernetes
30m         Warning   FailedCreate           replicaset/dotnetappkubernetes-XXXXXXXXXX   Error creating: pods "dotnetappkubernetes-XXXXXXXXXX-XXXXX" is forbidden: failed quota: defaultresourcequota: must specify limits.cpu for: dotnetappkubernetes; limits.memory for: dotnetappkubernetes; requests.cpu for: dotnetappkubernetes; requests.memory for: dotnetappkubernetes
26m         Warning   FailedCreate           replicaset/dotnetappkubernetes-XXXXXXXXXX   Error creating: pods "dotnetappkubernetes-XXXXXXXXXX-XXXXX" is forbidden: failed quota: defaultresourcequota: must specify limits.cpu for: dotnetappkubernetes; limits.memory for: dotnetappkubernetes; requests.cpu for: dotnetappkubernetes; requests.memory for: dotnetappkubernetes
25m         Normal    ScalingReplicaSet      deployment/dotnetappkubernetes              Scaled up replica set dotnetappkubernetes-XXXXXXXXXX from 0 to 1
25m         Normal    ScalingReplicaSet      deployment/dotnetappkubernetes              Scaled down replica set dotnetappkubernetes-XXXXXXXXXX from 1 to 0
25m         Normal    SuccessfulCreate       replicaset/dotnetappkubernetes-XXXXXXXXXX    Created pod: dotnetappkubernetes-XXXXXXX-XXXXX
```

- describe pod

```sh
kubectl describe pod dotnetappkubernetes-XXXXXXXXXX-XXXXX -n dotnet-app-cluster
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  28m                  default-scheduler  0/1 nodes are available: 1 Insufficient cpu. no new claims to deallocate, preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.
  Warning  FailedScheduling  18m (x2 over 23m)    default-scheduler  0/1 nodes are available: 1 Insufficient cpu. no new claims to deallocate, preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.
```

## allocated resources

- get nodes

```sh
kubectl get nodes
NAME                          STATUS   ROLES    AGE   VERSION
aks-agentpool-XXXXXXXX-vms1   Ready    <none>   21h   v1.35.7
```

- describe node agentpool

```sh
kubectl describe node aks-agentpool-XXXXXXXX-vms1 | grep -A 10 "Allocated resources"
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                1893m (99%)   35342m (1860%)
  memory             3492Mi (48%)  63877408Ki (870%)
  ephemeral-storage  0 (0%)        0 (0%)
  hugepages-1Gi      0 (0%)        0 (0%)
  hugepages-2Mi      0 (0%)        0 (0%)
  hugepages-32Mi     0 (0%)        0 (0%)
  hugepages-64Ki     0 (0%)        0 (0%)
```

- get list usage for region

```sh
az vm list-usage --location 'Name Region' --output table
Name                                      CurrentValue    Limit
----------------------------------------  --------------  -------
Standard Bpsv2 Family vCPUs               2               4
Total Regional Low-priority vCPUs         0               3
Total Regional vCPUs                      2               4
Virtual Machines                          1               25000
Availability Sets                         0               2500
Virtual Machine Scale Sets                0               2500
Dedicated vCPUs                           0               0
Cloud Services                            0               2500
Basic A Family vCPUs                      0               4
Standard A0-A7 Family vCPUs               0               4
Standard A8-A11 Family vCPUs              0               4
Standard Av2 Family vCPUs                 0               4
Standard BS Family vCPUs                  0               4
Standard Basv2 Family vCPUs               0               4
Standard Bsv2 Family vCPUs                0               4
Standard D Family vCPUs                   0               4
```

## aks scale agentpool

- example scale nodepool (`name-resource-group`,`name-cluster-kubernetes`,`name-agentpool`)

```sh
az aks nodepool scale --resource-group name-resource-group --cluster-name name-cluster-kubernetes --name name-agentpool --node-count 2
```

## aks add new agentpool

- add new agentpool mode user (`name-resource-group`,`name-cluster-kubernetes`,`name-agentpool`)

```sh
az aks nodepool add --resource-group name-resource-group --cluster-name name-cluster-kubernetes --name agentpool3 --node-vm-size Standard_D2as_v7 --node-count 2 --mode User
```

- example scale agent

## references

- check out [aks hybrid edge](https://learn.microsoft.com/en-us/azure/aks-hybrid-edge/windows-server/deploy-windows-application)
- check out [api rest with dotnet core](https://medium.com/nbellocam-es/creando-una-api-rest-con-asp-net-core-desde-cero-fc58924395fd)
- check out [dotnet-8-app-to-azure-kubernetes](https://dev.to/kosisochukwu_ugochukwu_a2/deploy-a-net-8-app-to-azure-kubernetes-service-aks-tutorial-guide-423c)
- check out [kubernetes manifest azure pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/kubernetes-manifest-v1?view=azure-pipelines)
- check out [deploy kubernetes azure devops](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/deploy-kubernetes?view=azure-devops)
