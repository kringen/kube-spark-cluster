# Spark Cluster on Kubernetes
This repo contains the files necessary to deploy a [Spark cluster](https://spark.apache.org/) on a basic [Kubernetes](https://kubernetes.io/) cluster.  This deployment will create 1 master node and any number of worker nodes.  Each node runs as a kubernetes pod.  This should work regardless of the kubernetes deployment.  It should work with any cloud provider or bare metal.

## Installation
### Deployment Values
Deployment variables should be updated in the [values.yaml](charts/spark-cluster/values.yaml) file.

Master and worker nodes use the same image version by default.  In addition, 1 master node is deployed by default.  This is configured in the *master* section:
```bash
master:
  name: spark-master
  replicas: 1
  container:
    name: spark-base
    imageRepository: kringen
    image: spark-base
    imageVersion: 3.8.0
```
The number of worker nodes is configured in the *worker* section.  The number of replicas controls how many worker nodes are created.
```bash
worker:
  name: spark-worker
  replicas: 5
  container:
    name: spark-base
    imageRepository: kringen
    image: spark-base
    imageVersion: 3.8.0
```
Resource limits are important to consider for worker nodes since Spark will not accept jobs if the correct resources (memory and CPU cores) are not available.

```bash
worker:
  container:
    resources:
      requests:
        memory: "1Gi"
        cpu: "2"
      limits:
        memory: "2Gi"
        cpu: "3"
```
### Deployment Command
The Spark cluster is deployed using the following command from the *kube-spark-cluster/charts* folder:
```bash
helm install spark-cluster ./spark-cluster
```
## Cluster Dashboard
Upon successful installation, the master node will be listening on port 8080.  This can be accessed by using kubectl port-forward comman and then opening the dashboard in a browser.
```bash
kubectl port-forward svc/spark-master --namespace spark 8080:8080
```
This command will forward traffic on http://localhost:8080 to the spark-master service.  Opening this URL in a browser (from the same machine issuing the kubectl command) should display a page like this: ![Spark Dashboard](../assets/assets/spark-dashboard-1.png)

## Docker Images
### spark-base
Both the master and worker nodes use the same image: [spark-base](docker/spark-base/Dockerfile).  This image starts with a basic python image and installs some key prerequisites such as:
* openjdk8-jre
* Spark
* Hadoop (Not really a prerequisite)

In addition, the image installs some libraries commonly used for working with remote storage (Azure Blob and AWS S3).

### jupyter-spark
Often times, users of Spark like to use Jupyter notebooks for an interactive development environment.  Since the notebook server acts as the "spark driver", it must use a kernel that can communicate with the cluster.  It must also contain the same version of libraries the cluster uses.  For this reason, a Dockerfile - [jupyter-spark](docker/jupyterhub/Dockerfile) - for use in JupyterHub deployments is included in this repo. A separate section has been added for implementing this image in a kubernetes installation.