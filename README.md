# HMS (Hive Metastore Service) Helm Chart 

This Hive chart (package) encapsulates all configurations and components needed to deploy a Hive Metastore service/Thrift service in a k8s cluster.

## Install the Chart
Populate the [values file](https://github.com/aws-samples/hive-emr-on-eks/blob/main/hive-metastore-chart/values.yaml)
```bash
helm repo add hive-metastore https://melodyyangaws.github.io/hive-metastore-chart 
helm install hive hive-metastore/hive-metastore -f values.yaml --namespace=emr --debug
```

## EKS Resources used in this Hive Chart

The resources used in the this chart are defined in yaml files inside [`/templates` directory](./templates). The following resources are used:

- [Configmap](templates/configmap.yaml): this k8a resource creates volumes that can be attached to containers. Here, we're mounting a volume to the [HMS configuration templates directory](templates/deployment.yaml#L65), which will be used by the HMS docker image to render the `metastore-site.yaml` config file.
- [Service](templates/service.yaml): this resources is responsible to expose the HMS service to the external.
- Horizontal Pods Autoscaler ([HPA](templates/hpa.yaml)): this resource increases the number of pods available in a ReplicaSet based on some thresholds (memory and cpu) in order to guarantee the service availability
- [Deployment](templates/deployment.yaml): the main resource, because it specifies pod's configurations and is the link between all resources and the HMS pods.

## Customize the Helm Chart

Modify the chart resources based on your need. Populate the [values.yaml](https://github.com/aws-samples/hive-emr-on-eks/blob/main/hive-metastore-chart/values.yaml) with your value. 
Test locally:

```bash
helm template . --name-template hive-metastore -f values.yaml -n emr | kubectl apply  -f -
kubectl delete cm -l app=hive-metastore -n emr
```
Follow [the steps](https://medium.com/containerum/how-to-make-and-share-your-own-helm-package-50ae40f6c221) to package up your helm chart to share.

## Customize Docker image
- [Dockerfile](docker/Dockerfile): if needed, edit the the docker image definition file that includes Hadoop binary package, standalone hive-metastore and RDS mysql driver.

```bash
cd docker
docker build -t hive-metastore .
docker tag hive-metastore:latest <YOUR_DOCKER_REPO>/hive-metastore:latest 
docker push <YOUR_DOCKER_REPO>/hive-metastore:latest
```
- [values.yaml](../source/app_resources/hive-metastore-values.yaml): update the docker image name and version on the helm chart value file

