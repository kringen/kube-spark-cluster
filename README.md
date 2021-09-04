# Spark Cluster on Kubernetes
This repo contains the files necessary to deploy a [Spark cluster](https://spark.apache.org/) on a basic [Kubernetes](https://kubernetes.io/) cluster.  This deployment will create 1 master node and any number of worker nodes.  Each node runs as a kubernetes pod.  This should work regardless of the kubernetes deployment.  It should work with any cloud provider or bare metal.

## Installation
### Dependency Setup
In order to deploy the clusters locally for testing, there are a few things that will need to be installed.  All steps in the section assume you are running on a Mac and have [Homebrew](https://brew.sh/) installed.

Make sure that you have cask installed and up-to-date by running `brew tap homebrew/cask`.

#### Install Docker
[Docker](https://docs.docker.com/) is an open platform for developing, shipping, and running applications.  To get started, you will need to [Install Docker](https://docs.docker.com/docker-for-mac/install/) from the Docker website.

#### Install Virtualbox
[VirtualBox](https://www.virtualbox.org/wiki/Documentation) is open-source software for virtualizing the x86 computing architecture. It acts as a hypervisor, creating a VM (virtual machine) where the user can run another OS (operating system).

To install it, run `brew install virtualbox`.

This is the same as `brew install --cask virtualbox` and is a replacement for the old method of calling `brew cask install virtualbox`.

Make sure that during install in `System Preferences > Security & Privacy > Allow` you checked the box for `Oracle`.  If it was not checked, check it, reinstall virtualbox using the steps above, and restart your computer.

#### Install Kubectl
[Kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) is a Kubernetes command-line tool that allows you to run [commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.

To install it run `brew install kubectl`.

#### Install Minikube
[Minikube](https://kubernetes.io/docs/tutorials/hello-minikube/) is a tool that lets you run Kubernetes locally.  It runs a single-node Kubernetes cluster on your personal computer (including Windows, macOS and Linux PCs) so that you can try out Kubernetes or for daily development work.  To work with it, you will also need virtualbox and kubectl so make sure to follow the installation steps above.

To install it, run the following curl command:

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.27.0/minikube-darwin-amd64 &&\
  chmod +x minikube &&\
  sudo mv minikube /usr/local/bin/
```

With minikube installed, you can start your minicube cluster with the `minikube start` command.  Hold tight, it might take a few minutes.

Once the cluster is up and running, you should be able to confirm by executing the command `kubectl api-versions` to get a list of versions or `kubectl get pods --all-namespaces` to see all running pods and health state of the cluster.

If you get an error that looks something like `Error starting host: Error getting state for host: machine does not exist` when running `minikube start` you can try to fix it by running:

1. `minikube stop` to stop the cluster
2. `minikube delete && rm -rf ~/.minikube && rm -rf ~/.kube` to delete the machine
3. `minikube start` to re-run the cluster

If you get an error that looks something like `Error starting host:  Error creating host: Error executing step: Creating VM.` you will want to make sure that virtualbox is installed correctly and that you have restarted your machine.

If you are running the cluster locally, you may also need to install [hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/) with `brew install docker-machine-driver-hyperkit` and start minikube using the kit with `minikube start --vm-driver=hyperkit --bootstrapper=localkube`.

#### Install Helm
[Helm](https://helm.sh/docs/) helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.

To install it, run `brew install helm`.


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
    imageVersion: 3.8.1
```
The number of worker nodes is configured in the *worker* section.  The number of replicas controls how many worker nodes are created.
```bash
worker:
  name: spark-worker
  replicas: 1
  container:
    name: spark-base
    imageRepository: kringen
    image: spark-base
    imageVersion: 3.8.1
```
Resource limits are important to consider for worker nodes since Spark will not accept jobs if the correct resources (memory and CPU cores) are not available.

```bash
worker:
  container:
    resources:
      requests:
        memory: "1Gi"
        cpu: "1"
      limits:
        memory: "2Gi"
        cpu: "3"
```

#### Persistent Volumes
If you want to persist volumes and mount them in the nodes, you can optionally reference these values in a separate values file.  An example file has been added that connects to [Azure Files](https://docs.microsoft.com/en-us/azure/storage/files/storage-files-introduction)) and mounts the remote share in the /srv/spark/data directory.  The benefit of this approach is that data files can be placed in the Azure Files share and all nodes will have access to read that file during processing.

A prerequisite for this is to create a secret that contains the storage account name and access key.  Here is an example of how to create that secret:
```
kubectl create secret generic azurefilesecret -n spark \
  --from-literal=azurestorageaccountname=<storage account name goes here> \
  --from-literal=azurestorageaccountkey=<storage account key goes here>
```
Once the secret is created, you can reference it by name in [values-azure-storage.yaml](values-azure-storage.yaml).

### Deployment Command
The Spark cluster can now be deployed by:

1. Navigating to the *kube-spark-cluster/charts* folder
2. Running `helm install spark-cluster ./spark-cluster` or if you want to use Azure Files for shared storage, reference the additional values file: `helm install spark-cluster ./spark-cluster -f ./spark-cluster/values-azure-storage.yaml`

You can test that this worked by running the command `kubectl get pods --namespace spark` and making sure the `spark-test` pod is present.

```
NAME             READY   STATUS    RESTARTS   AGE
spark-master-0   1/1     Running   0          11m
spark-test       1/1     Running   0          20s
spark-worker-0   1/1     Running   0          11m
```

If you have issues during installation, you can always start over by running `helm uninstall spark-cluster`.

## Cluster Dashboard
Upon successful installation, the master node will be listening on port 8080.  This can be accessed by using kubectl port-forward comman and then opening the dashboard in a browser.
```bash
kubectl port-forward svc/spark-master --namespace spark 8080:8080
```
This command will forward traffic on http://localhost:8080 to the spark-master service.  Opening this URL in a browser (from the same machine issuing the kubectl command) should display a page like this: ![Spark Dashboard](../assets/assets/spark-dashboard-1.png)

## Submitting a Spark Job
Once the spark cluster is deployed we can test it using the same image that got deployed for the master and worker nodes.  To create a temporary pod that will last 1 hour you can execute the following command:

```
kubectl run spark-test -n spark --image=kringen/spark-base:3.8.1 -- /bin/sh -c "sleep 3600"
```
Once the pod is up and running you can connect to the bash terminal by executing the following:

```
kubectl exec -it spark-test -n spark -- /bin/sh
```
You should see a prompt from within the spark-test pod logged in as root.

```
/ #
```
 One important task to do before executing a spark job is to define the pod's IP address as its spark.driver.host by executing the following commands. This is necessary so the cluster can reach out to the pod to update job status.  If left out, the cluster nodes will try to report back to "spark-test" which is not a valid DNS service in the cluster.
```
echo "spark.driver.host=$(hostname -i)" > /usr/local/spark/conf/spark-defaults.conf

cat /usr/local/spark/conf/spark-defaults.conf
```
You should see something like `spark.driver.host=xx.xx.xx.xx`

To create an interactive pypark session connecting to the cluster:
```
/usr/local/spark/bin/pyspark --master spark://spark-master:7077
```
At this point you should see the pyspark terminal prompting you for input!
```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 3.0.0
      /_/

Using Python version 3.9.5 (default, May  4 2021 18:33:52)
SparkSession available as 'spark'.
>>> 

```
You can check for a spark context by entering `sc`:
```
>>> sc
<SparkContext master=spark://spark-master:7077 appName=PySparkShell>

```
To test reading a text file:
```
>>> df=spark.read.text("/etc/hosts")
>>> df.show()
```
You should see some output containing the text of /etc/hosts.  This file was chosen since it will exists on **all worker nodes**.  In order to do some real data reads you will need to make sure your source files are persisting either in shared storage or in HDFS.

Once testing is complete, use `CTRL+d` to exit out of the pyspark prompt and `exit` to exit out of the test pod.

## Docker Image
### spark-base
Both the master and worker nodes use the same image: [spark-base](docker/spark-base/Dockerfile).  This image starts with a basic python image and installs some key prerequisites such as:
* openjdk8-jre
* Spark
* Hadoop (Not really a prerequisite)

In addition, the image installs some libraries commonly used for working with remote storage (Azure Blob and AWS S3).

## JupyterHub Integration
Often times, users of Spark like to use Jupyter notebooks for an interactive development environment.  Since the notebook server acts as the "spark driver", it must use a kernel that can communicate with the cluster.  It must also contain the same version of libraries the cluster uses.  These instructions are adapted from the [JupyterHub documentation](https://zero-to-jupyterhub.readthedocs.io/en/latest/jupyterhub/installation.html).

To add the [JupyterHub helm repo](https://github.com/jupyterhub/helm-chart), execute the following command:
```
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update

```
Once the repo is installed you can deploy it in the *same namespace as the cluster*:
```
helm install jhub jupyterhub/jupyterhub -n spark -f ./jupyterhub/values-simple.yaml
```
This will install a "simple" setup without TLS or persistent storage.  For instructions on installing a more advanced setup, refer to this [README](charts/jupyterhub/README.md).

Upon completion of installation, you can access the JupyterHub application by using port kubectl port forwarding:
```
kubectl port-forward svc/proxy-public -n spark 8081:80

```
This will forward port 8081 of your computer to port 80 of the service.  You can now open up a browser and browse to the launcher page at https://localhost:8081

The default username and password are:
* username: jovyan
* password: jupyter

A new "user" pod will spin up and redirect you to the launcher page.

![Jupyter Launcher Menu](../assets/assets/jupyter-launcher-pyspark.png)

Notice the *Pyspark* kernel on the launcher menu? Clicking this will start a new notebook running the Spark Context of the cluster we deployed previously.  Once the notebook starts, enter `sc` in the first cell and press `CTRL+ENTER`.  You should see the following results:
```
SparkContext

Spark UI

Version   v3.0.0
Master    spark://spark-master:7077
AppName   pyspark-shell

```

### Docker Image
One of the great features of Jupyterhub is the fact that a user-specific pod spins up when a user logs in.  The image used for this pod contain the software required for interacting with the pod using a Jupyter notebook.  There are different stock images that JupyterHub can deploy.  You can also create a custom image.  Since our user pod will interact with our Spark cluster, a Dockerfile - [jupyter-spark](docker/jupyterhub/Dockerfile) - for use in JupyterHub deployments is included in this repo.

The jupyter-spark image uses the [jupyter/datascience-notebook](https://hub.docker.com/r/jupyter/datascience-notebook/tags/?page=1&ordering=last_updated) base image.  Then openjdk-8-jdk and Spark are installed.
