# Windows Containers on Google Kubernetes Engine (GKE)

Steps to deploy a Windows container on Google Kubernetes Engine (GKE)

## Setup

Update gcloud to latest:

```bash
gcloud components update --quiet
```

Set a cluster name and zone:

```bash
export CLUSTER_NAME=windows-cluster
export ZONE=europe-west1-b
gcloud config set compute/zone ${ZONE}
```

## Create a cluster

Create a GKE cluster on rapid channel and ip-alias enabled:

```bash
gcloud beta container clusters create ${CLUSTER_NAME} \
--enable-ip-alias \
--num-nodes=2 \
--release-channel=rapid
```

Create a Windows node-pool:

```bash
gcloud container node-pools create windows-node-pool \
--cluster=${CLUSTER_NAME} \
--image-type=WINDOWS_LTSC \
--no-enable-autoupgrade \
--machine-type=n1-standard-2
```

Enable kubectl to work with the cluster:

```bash
gcloud container clusters get-credentials ${CLUSTER_NAME}
```

Ensure that the webhook for Windows containers is created:

```bash
kubectl get mutatingwebhookconfigurations

NAME                                                 CREATED AT
...
windows.config.common-webhooks.networking.gke.io     2020-01-20T15:09:16Z
```

## Deploy a Windows container

Create an [iis.yaml](iis.yaml) deployment file with windows node selector:

```yaml
nodeSelector:
   kubernetes.io/os: windows
```

Create the deployment:

```bash
kubectl apply -f iis.yaml

deployment.apps/iis created
```

Expose the deployment behind a service:

```bash
kubectl expose deployment iis \
--type=LoadBalancer \
--name=iis
```

Validate that pods are running:

```bash
kubectl get pods

NAME                   READY   STATUS    RESTARTS   AGE
iis-85cbdcf7ff-xc6r2   1/1     Running   0          6m45s
```

Validate that the service has an external ip:

```bash
kubectl get service iis

NAME   TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
iis    LoadBalancer   10.100.10.143   35.195.123.123   80:31318/TCP   5m26s
```

Test the Windows container:

```bash
curl http://${EXTERNAL_IP}
```