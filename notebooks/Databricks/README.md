# Azure Databricks

The recommended approach for the OpenHack is to use the Data Science Virtual Machine. However, some participants may want to experiment with machine learning in Azure Databricks. This folder contains some challenge solutions that have been validated on Azure Databricks.

***Tip:*** If you are new to Azure Databricks, consider completing the following hands-on labs to familiarize yourself with the service:

* https://aka.ms/dbintro
* https://aka.ms/dbml

## Azure Databricks Setup

Azure Databricks is a platform for data analytics based on *Apache Spark* clusters. Spark works by distributing data processing workloads across multiple *worker* nodes, managed by a central *driver* node.

***Important:*** Some machine learning frameworks (in particular PyTorch) include internal mechanisms for distributing data IO operations that may not be compatible with the underlying MapReduce implementation used in Spark prior to version 2.4.0. Databricks 5.0 introduces support for Spark 2.4.0; and with it, support for *Barrier Execution Mode*. This enables machine learning frameworks to run distributed jobs on Spark clusters successfully. Use the following instructions to provision your Databricks environment.

1. Create an Azure Databricks Workspace. Choose any available location, and select the **Standard** pricing tier.
2. Launch your workspace and create a cluster using the followng settings:

    * **Cluster Name:** *Any valid name*
    * **Cluster Mode:** Standard
    * **Databricks Runtime Version:** 5.0 (includes Apache Spark 2.4.0) - *You can also choose the edition of 5.0 that supports GPUs if you have sufficient quota in your Azure subscription)*
    * **Python Version:** 3
    * **Driver Type:** Same as worker
    * **Worker Type:** Standard_D16_v3 (*minimum - choose a GPU image if supported*)
    * **Min Workers:** 1
    * **Max Workers:** 2 (*You can choose more if you don't need to minimize costs!*)
    * **Enable autoscaling:** *Selected*
    * **Auto Termination:** Terminate after 60 minutes of inactivity
3. In your **Home** folder, import the solution notebooks.
4. For each notebook, create the required *libraries* by installing the specified **PyPi** packages, and attach the libraries to your cluster.

## Notes

* Creating and attaching libraries is the equivalent of using *pip* to install Python libraries in a regular Python environment. See https://docs.databricks.com/user-guide/libraries.html for more information.
* Notebooks in Azure Databricks do ***not*** support the `%matplotlib inline` command used to display plots in Jupyter notebooks. Instead, you must create a pyplot *figure* and use the **display()** function to show it inline. See https://docs.databricks.com/user-guide/visualizations/matplotlib-and-ggplot.html.
* Databricks clusters share a DBFS file system across all nodes, and additionally each node has its own local file system. This can be confusing! You can use the **/dbfs** mountpoint on the local file system to access the shared DBFS file system, which is where you should store data files you want to process (because, (a.) this enables all cluster nodes to access them and distribute the processing, and (b.) the local file system is deleted every time you restart the cluster).
* There are some known issues when using the Azure Machine Learning SDK on Databricks (see https://docs.microsoft.com/en-us/azure/machine-learning/service/resource-known-issues#databricks). To work around these, explicitly create libraries for the following (in some cases, version-specific) PyPi packages:
  * psutil
  * cryptography==1.5
  * pyopenssl==16.0.0
  * ipython==2.2.0
  * azureml-sdk\[databricks\]
* If you need to use a pre-2.4.0 version of Spark, de-select the auto-scaling option and specify **Min Workers** value of **0** (so that all code will run on the driver node). Running machine learning code on a single driver node means that you don't get the scale-out benefit of Spark. However, in production environments where Spark on Databricks is already used for big data processing, it may still be useful to integrate ML-related workloads into the same environment.