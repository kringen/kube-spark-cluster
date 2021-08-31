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
### Deployment Command
The Spark cluster can now be deployed by:

1. Navigating to the *kube-spark-cluster/charts* folder
2. Running `helm install spark-cluster ./spark-cluster`

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

## Docker Image
### spark-base
Both the master and worker nodes use the same image: [spark-base](docker/spark-base/Dockerfile).  This image starts with a basic python image and installs some key prerequisites such as:
* openjdk8-jre
* Spark
* Hadoop (Not really a prerequisite)

In addition, the image installs some libraries commonly used for working with remote storage (Azure Blob and AWS S3).

## JupyterHub Integration
Often times, users of Spark like to use Jupyter notebooks for an interactive development environment.  Since the notebook server acts as the "spark driver", it must use a kernel that can communicate with the cluster.  It must also contain the same version of libraries the cluster uses.  

![Jupyter Launcher Menu](../assets/assets/jupyter-launcher-pyspark.png)

After integration, a new *Pyspark* kernel appears on the launcher menu

### Docker Image
One of the great features of Jupyterhub is the fact that a user-specific pod spins up when a user logs in.  The image used for this pod contain the software required for interacting with the pod using a Jupyter notebook.  There are different stock images that JupyterHub can deploy.  You can also create a custom image.  Since our user pod will interact with our Spark cluster, a Dockerfile - [jupyter-spark](docker/jupyterhub/Dockerfile) - for use in JupyterHub deployments is included in this repo.

The jupyter-spark image uses the [jupyter/datascience-notebook](https://hub.docker.com/r/jupyter/datascience-notebook/tags/?page=1&ordering=last_updated) base image.  Then openjdk-8-jdk and Spark are installed.