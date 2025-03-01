# Notice

Repository is in read-only and [archive](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/about-archiving-repositories) mode

## Introduction

[![Build Status](https://dev.azure.com/azure/helm-vsts-agent/_apis/build/status/Azure.helm-vsts-agent?branchName=master)](https://dev.azure.com/azure/helm-vsts-agent/_build/latest?definitionId=12?branchName=master)

This chart bootstraps a [Visual Studio Team Services agent pool](https://github.com/Microsoft/vsts-agent) on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites
 - Kubernetes 1.8+ or newer

## Configuration

The following tables lists the configurable parameters of the `vsts-agent` chart and their default values.

| Parameter                         | Description                           | Default                                                   |
| --------------------------------- | ------------------------------------- | --------------------------------------------------------- |
| `image.repository`                | vsts-agent image                      | `microsoft/vsts-agent`                                    |
| `image.tag`                       | Specify image tag                     | `latest`                                                  |
| `image.pullSecrets`               | Specify image pull secrets            | `nil` (does not add image pull secrets to deployed pods)  |
| `image.pullPolicy`                | Image pull policy                     | `Always`                                                  |
| `replicas`                        | Number of vsts-agent instaces started | `3`                                                       |
| `resources.disk`                  | Size of the disk attached to the agent| `50Gi`                                                    |
| `resources.storageclass`          | Specify storageclass used in kubernetes| `default`                                                    |
| `adoAccount`                      | AZP account name                      | `nil` (must be provided during installation)              |
| `adoToken`                        | AZP personal access token             | `nil` (must be provided during installation)              |
| `adoPool`                         | AZP agent pool name                   | `kubernetes-ado-agents`                                  |
| `adoAgentName`                    | AZP agent name                        | `$HOSTNAME`                                               |
| `adoWorkspace`                    | AZP agent workspace                   | `/workspace`                                              |
| `extraEnv`                   | Extra environment variables on the vsts-agent container                  | `nil`                                              |
| `cleanRun`                   | Kill and restart vsts-agent container on completion of a build (completely resets the environment)                  | `false`                                              |
| `volumes`                   | An array of custom volumes to attach to the vsts-agent pod                  | `docker-socket` to mount /var/run/docker.sock (if you still need this volume when defining addition ones, please ensure you reference it again in your list)                                             |
| `volumeMounts`                   | volumeMounts to the vsts-agent container as referenced in `volumes`                  | A read-only `docker-socket` to mount as /var/run/docker.sock in vsts-agent container (as in `volumes` please reference this again if you still need it in your custom list)                                               |
| `extraContainers`                   | Array of additional sidecar containers to add into the vsts-agent pod                  | `nil`                                              |

## Configure your AZP instance

Before starting the chart installation, you have to configure your AZP instance as follows:

1. Create a personal access token with the authorized scope **Agent Pools(read, manage)**  following these [instructions](https://docs.microsoft.com/en-us/vsts/git/_shared/personal-access-tokens). You will have to provide later the base64 encoded value of this token to the `adoToken` value of the chart.

2. Create a new queue and agent pool with the name `kubernetes-ado-agents`. You can find more details [here](https://docs.microsoft.com/en-us/vsts/build-release/concepts/agents/pools-queues#creating-agent-pools-and-queues).

## Installing the Chart

The chart can be installed with the following command:

```bash
export AZP_TOKEN=$(echo -n '<AZP TOKEN>' | base64)

helm install --namespace <NAMESPACE> --set adoToken=${AZP_TOKEN} --set adoAccount=<AZP ACCOUNT> --set adoPool=<AZP POOL> -f values.yaml vsts-agent .
```

Your deployment should look like this if everything works fine:

```bash
kubectl get pods --namespace <NAMESPACE>
NAME           READY     STATUS    RESTARTS   AGE
vsts-agent-0   1/1       Running   0          1m
vsts-agent-1   1/1       Running   0          1m
vsts-agent-2   1/1       Running   0          1m
```
## Upgrading a deployed Chart

A deployed chart can be upgraded as follows:

```bash
export AZP_TOKEN=$(echo -n '<AZP TOKEN>' | base64)

helm upgrade --timeout 900 --namespace <NAMESPACE> --set adoToken=${AZP_TOKEN} --set adoAccount=<AZP ACCOUNT> --set adoPool=<AZP POOL> -f values.yaml vsts-agent . --wait
```

This command will upgrade the exiting chart and wait until the deployment is completed.

## Uninstalling the Chart

The chart can be uninstalled/deleted as follows:

```bash
helm delete --purge vsts-agent
```

This command removes all the Kubernetes resources associated with the chart and deletes the helm release.

## Validate the Chart

You can validate the chart during development by using the `helm template` command.

```bash
helm template .
```

## Scale up the number of AZP agents

The number of AZP agents can be easily increased to `10` by using the following command:

```bash
kubectl scale --namespace <NAMESPACE> statefulset/vsts-agent --replicas 10
```
## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
