# Jupyterhub Installation
These instructions will walk you through installing Jupyterhub so it can execute Spark jobs on the cluster.

These instructions are adapted from the [JupyterHub documentation](https://zero-to-jupyterhub.readthedocs.io/en/latest/jupyterhub/installation.html).

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
