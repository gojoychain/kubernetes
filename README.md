# Deploying With Kubernetes

Gojoy Kubernetes/Helm charts

## Prerequisites

1. [Kubernetes kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
2. [Amazon eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)
3. [Helm](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)
4. [Tiller](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)

## Init Environment

### Start Tiller Locally

[Using Helm with Amazon EKS - Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)

As stated by the Amazon docs, it's safer to start Tiller locally instead. Run the following commands in your terminal:

```bash
$ export TILLER_NAMESPACE=tiller
$ tiller -listen=localhost:44134 -storage=secret -logtostderr
[main] 2019/08/28 10:06:56 Starting Tiller v2.14.2 (tls=false)
[main] 2019/08/28 10:06:56 GRPC listening on localhost:44134
[main] 2019/08/28 10:06:56 Probes listening on :44135
[main] 2019/08/28 10:06:56 Storage driver is Secret
[main] 2019/08/28 10:06:56 Max history per release is 0
```

### Start Helm Client

Next, open a new Terminal tab and run the following commands to start the Helm client.

```bash
$ export HELM_HOST=:44134
$ helm init --client-only
$HELM_HOME has been configured at /Users/deric/.helm.
Not installing Tiller due to 'client-only' flag having been set
```

## Secrets

Before deploying your Chart, some pods might require some [secrets](https://kubernetes.io/docs/concepts/configuration/secret/). Check the `deployment` or `statefulset` environment to see what is required.

Secrets need to be base64 encoded. Read [Creating a Secret Manually](https://kubernetes.io/docs/concepts/configuration/secret/#creating-a-secret-manually).

## Versioning

Chart `version` and `appVersion` should be updated to match the Docker image version.

## Charts

### Deploy New Chart

Deploying a Chart is easy. Here's an example of deploying a chart:

```bash
$ cd bodhi-pm-testnet
# helm install --name name-of-deployment --namespace your-namespace path/to/chart
$ helm install --name bodhi-pm-testnet --namespace default .
NAME:   bodhi-pm-testnet
LAST DEPLOYED: Wed Aug 28 10:33:12 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                     DATA  AGE
bodhi-pm-testnet-config  5     3s

==> v1/Deployment
NAME              READY  UP-TO-DATE  AVAILABLE  AGE
bodhi-pm-testnet  0/3    3           0          2s

==> v1/Pod(related)
NAME                               READY  STATUS             RESTARTS  AGE
bodhi-pm-testnet-7b7768b694-2md27  0/1    ContainerCreating  0         2s
bodhi-pm-testnet-7b7768b694-8q2qw  0/1    ContainerCreating  0         2s
bodhi-pm-testnet-7b7768b694-blrsk  0/1    ContainerCreating  0         2s

==> v1/Secret
NAME                   TYPE    DATA  AGE
bodhi-pm-testnet-auth  Opaque  3     3s

==> v1/Service
NAME              TYPE          CLUSTER-IP      EXTERNAL-IP       PORT(S)         AGE
bodhi-pm-testnet  LoadBalancer  10.100.217.144  a91395082c944...  5000:30163/TCP  2s
```

You can verify your pods are running after:

```bash
$ kubectl get pod
NAME                                             READY   STATUS    RESTARTS   AGE
bodhi-pm-testnet-6c4849558-cjb49                 1/1     Running   0          14s
bodhi-pm-testnet-6c4849558-cvcjt                 1/1     Running   0          14s
bodhi-pm-testnet-6c4849558-n9knj                 1/1     Running   0          14s
```

### Upgrade Chart

When you change the Chart and want to redeploy, you want to upgrade. Here's an example of an upgrade:

```bash
$ cd bodhi-pm-testnet
# helm upgrade name path/to/chart
$ helm upgrade bodhi-pm-testnet .
Release "bodhi-pm-testnet" has been upgraded.
LAST DEPLOYED: Thu Aug 29 17:10:13 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                     DATA  AGE
bodhi-pm-testnet-config  5     22h

==> v1/Deployment
NAME              READY  UP-TO-DATE  AVAILABLE  AGE
bodhi-pm-testnet  3/3    1           3          22h

==> v1/Pod(related)
NAME                               READY  STATUS             RESTARTS  AGE
bodhi-pm-testnet-575bc57d4-75x72   1/1    Running            3         22h
bodhi-pm-testnet-575bc57d4-ddd8x   1/1    Running            2         22h
bodhi-pm-testnet-575bc57d4-hdkng   1/1    Running            3         22h
bodhi-pm-testnet-856f58d9c5-f248x  0/1    ContainerCreating  0         2s

==> v1/Secret
NAME                   TYPE    DATA  AGE
bodhi-pm-testnet-auth  Opaque  3     22h

==> v1/Service
NAME              TYPE          CLUSTER-IP      EXTERNAL-IP       PORT(S)         AGE
bodhi-pm-testnet  LoadBalancer  10.100.170.230  a0bb2dd5dc987...  5000:31487/TCP  22h
```

### Delete Chart

Deleting your already deployed service is easy. Just use the following command with the name of the service.

**When you delete a Chart, you are deleting everything that comes with the Chart. If you are trying to just update the release/image. DO NOT DELETE, but [UPGRADE](#upgrade-chart).**

```bash
# helm del --purge deployment-name
$ helm del --purge bodhi-pm-testnet
release "bodhi-pm-testnet" deleted
```
