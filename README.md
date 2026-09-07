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

## references

- check out [aks hybrid edge](https://learn.microsoft.com/en-us/azure/aks-hybrid-edge/windows-server/deploy-windows-application)
- check out [api rest with dotnet core](https://medium.com/nbellocam-es/creando-una-api-rest-con-asp-net-core-desde-cero-fc58924395fd)
- check out [dotnet-8-app-to-azure-kubernetes](https://dev.to/kosisochukwu_ugochukwu_a2/deploy-a-net-8-app-to-azure-kubernetes-service-aks-tutorial-guide-423c)
