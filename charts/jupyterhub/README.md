# Jupyterhub Repo Installation
These instructions will walk you through installing Jupyterhub so it can execute Spark jobs on the cluster.

These instructions are adapted from the [JupyterHub documentation](https://zero-to-jupyterhub.readthedocs.io/en/latest/jupyterhub/installation.html).

To add the [JupyterHub helm repo](https://github.com/jupyterhub/helm-chart), execute the following command:
```
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update

```
# Simple Deployment
Once the repo is installed you can deploy it in the *same namespace as the cluster*:
```
helm install jhub jupyterhub/jupyterhub -n spark -f ./jupyterhub/values-simple.yaml
```
This will install a "simple" setup without TLS or persistent storage.  For instructions on installing a more advanced setup, refer to this [README](charts/jupyterhub/README.md).

# Deploy with Persistent Volumes
If you want to have your previous files available When JupyterHub spins up a new user pod, you need to configure it to use persistent storage.

## Azure Files
If you have an Azure storage account, you can create a file share which will be mounted by your user pod.

### Create a share on Azure Storage Account
Create a share named `jupyterhub-shared`.  This will be referenced when deploying a Persistent Volume (PV).

### Create a Secret
A prerequisite for this is to create a secret that contains the storage account name and access key.  Here is an example of how to create that secret:
```
kubectl create secret generic azurefilesecret -n spark \
  --from-literal=azurestorageaccountname=<storage account name goes here> \
  --from-literal=azurestorageaccountkey=<storage account key goes here>
```

### Create a PV and PVC
Included in the repo is an example yaml file to deploy the PV and PVC.  Check out the code to make sure it fits your requirements and deploy it with the following command:
```
kubectl apply -f jupyterhub/azure-files.yaml -n spark
```
This will create a Persistent Volume mounting the `jupyterhub-shared` file share created earlier.  It will also create a PVC that will be referenced in the user pod definition.

### Deploying JupyterHub
Run the following command to install JupytherHub with persistent storage in the user pod:

```
helm install jhub jupyterhub/jupyterhub -n spark -f ./jupyterhub/values-azure-files.yaml
```

# Testing it Out
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
