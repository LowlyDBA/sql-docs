---
title: GPU support and TensorFlow 
titleSuffix: SQL Server big data clusters
description: Deploy a big data cluster with GPU support and use TensorFlow in Azure Data Studio Notebooks.
author: lgongmsft
ms.author: shivprashant
ms.reviewer: jroth
manager: craigg
ms.date: 03/27/2018
ms.topic: conceptual
ms.prod: sql
ms.technology: big-data-cluster
---

# Deploy a big data cluster with GPU support and run TensorFlow

[!INCLUDE[tsql-appliesto-ssver15-xxxx-xxxx-xxx](../includes/tsql-appliesto-ssver15-xxxx-xxxx-xxx.md)]

This article demonstrates how to deploy a big data cluster on Azure Kubernetes Service (AKS) that supports GPU-enabled node pools for compute-intensive workloads. You then run example notebooks in Azure Data Studio that perform image classification with TensorFlow for GPU.

## Prerequisites

- [Big data tools](deploy-big-data-tools.md):
  - **mssqlctl**
  - **kubectl**
  - **Azure Data Studio**
  - **SQL Server 2019 extension**
  - **Azure CLI**

[!INCLUDE [Limited public preview note](../includes/big-data-cluster-preview-note.md)]

## Create an AKS cluster

The following steps use the Azure CLI to create an AKS cluster that supports GPUs.

1. At the command prompt, run the following command and follow the prompts to log in to your Azure subscription:

    ```azurecli
    az login
    ```

1. Create a resource group with the **az group create** command. The following example creates a resource group named `sqlbigdatagroupgpu` in the `eastus` location.

   ```azurecli
   az group create --name sqlbigdatagroupgpu --location eastus
   ```

1. Create a Kubernetes cluster in AKS with the [az aks create](https://docs.microsoft.com/cli/azure/aks) command. The following example creates a Kubernetes cluster named `gpucluster` in the `sqlbigdatagroupgpu` resource group.

   ```azurecli
   az aks create --name gpucluster --resource-group sqlbigdatagroupgpu --generate-ssh-keys --node-vm-size Standard_NC6 --node-count 3 --node-osdisk-size 50 --kubernetes-version 1.11.7 --location eastus
   ```

   > [!NOTE]
   > This cluster uses the  **Standard_NC6** [GPU-optimized virtual machine size](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-gpu), which is one of the specialized virtual machines available with single or multiple NVIDIA GPUs. For more information, see [Use GPUs for compute-intensive workloads on Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/gpu-cluster).

1. To configure kubectl to connect to your Kubernetes cluster, run the [az aks get-credentials](https://docs.microsoft.com/cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials) command.

   ```azurecli
   az aks get-credentials --overwrite-existing --resource-group=sqlbigdatagroupgpu --name gpucluster
   ```

## Apply YAML manifest

1. Use **kubectl** to create a Kubernetes namespace named `gpu-resources`.

   ```
   kubectl create namespace gpu-resources
   ```

1. Save the contents of the following YAML file to a file named **nvidia-device-plugin-ds.yaml**. Save this to the working directory on your machine that you are running **kubectl** from.

   Update the `image: nvidia/k8s-device-plugin:1.11` half-way down the manifest to match your Kubernetes version. For example, if your AKS cluster runs Kubernetes version 1.12, update the tag to `image: nvidia/k8s-device-plugin:1.12`.

   ```yaml
   apiVersion: extensions/v1beta1
   kind: DaemonSet
   metadata:
     labels:
       kubernetes.io/cluster-service: "true"
     name: nvidia-device-plugin
     namespace: gpu-resources
   spec:
     template:
       metadata:
         # Mark this pod as a critical add-on; when enabled, the critical add-on scheduler
         # reserves resources for critical add-on pods so that they can be rescheduled after
         # a failure.  This annotation works in tandem with the toleration below.
         annotations:
           scheduler.alpha.kubernetes.io/critical-pod: ""
         labels:
           name: nvidia-device-plugin-ds
       spec:
         tolerations:
         # Allow this pod to be rescheduled while the node is in "critical add-ons only" mode.
         # This, along with the annotation above marks this pod as a critical add-on.
         - key: CriticalAddonsOnly
           operator: Exists
         containers:
         - image: nvidia/k8s-device-plugin:1.11 # Update this tag to match your Kubernetes version
           name: nvidia-device-plugin-ctr
           securityContext:
             allowPrivilegeEscalation: false
             capabilities:
               drop: ["ALL"]
           volumeMounts:
             - name: device-plugin
               mountPath: /var/lib/kubelet/device-plugins
         volumes:
           - name: device-plugin
             hostPath:
               path: /var/lib/kubelet/device-plugins
         nodeSelector:
           beta.kubernetes.io/os: linux
           accelerator: nvidia
   ```

1. Now use the kubectl apply command to create the DaemonSet. **nvidia-device-plugin-ds.yaml** must be in the working directory when you run the following command:

   ```
   kubectl apply -f nvidia-device-plugin-ds.yaml
   ```

## Deploy the big data cluster

To deploy a SQL Server 2019 big data cluster (preview) that supports GPUs, you must deploy from a specific docker registry and repository. Specifically, you use different values for **DOCKER_REGISTRY**, **DOCKER_REPOSITORY**, **DOCKER_USERNAME**, **DOCKER_PASSWORD**, and **DOCKER_EMAIL**. The following sections provide examples of how to set the environment variables. Use either the Windows or Linux sections depending on the platform of the client you are using to deploy the big data cluster.

### Windows

   1. Using a CMD window (not PowerShell), configure the following environment variables. Do not use quotes around the values.

      ```cmd
      SET ACCEPT_EULA=yes
      SET CLUSTER_PLATFORM=aks

      SET CONTROLLER_USERNAME=<controller_admin_name - can be anything>
      SET CONTROLLER_PASSWORD=<controller_admin_password - can be anything, password complexity compliant>
      SET KNOX_PASSWORD=<knox_password - can be anything, password complexity compliant>
      SET MSSQL_SA_PASSWORD=<sa_password_of_master_sql_instance, password complexity compliant>

      SET DOCKER_REGISTRY=marinchcreus3.azurecr.io
      SET DOCKER_REPOSITORY=ctp24-8-0-61-gpu
      SET DOCKER_USERNAME=<your username, gpu-specific credentials provided by Microsoft>
      SET DOCKER_PASSWORD=<your password, gpu-specific credentials provided by Microsoft>
      SET DOCKER_EMAIL=<your email address>
      SET DOCKER_PRIVATE_REGISTRY=1
      SET STORAGE_SIZE=10Gi
      ```

   1. Deploy the big data cluster:

      ```cmd
      mssqlctl cluster create --name gpubigdatacluster
      ```

### Linux

   1. Initialize the following environment variables. In bash, you can use quotes around each value.

      ```bash
      export ACCEPT_EULA=yes
      export CLUSTER_PLATFORM="aks"

      export CONTROLLER_USERNAME="<controller_admin_name - can be anything>"
      export CONTROLLER_PASSWORD="<controller_admin_password - can be anything, password complexity compliant>"
      export KNOX_PASSWORD="<knox_password - can be anything, password complexity compliant>"
      export MSSQL_SA_PASSWORD="<sa_password_of_master_sql_instance, password complexity compliant>"

      export DOCKER_REGISTRY="marinchcreus3.azurecr.io"
      export DOCKER_REPOSITORY="ctp24-8-0-61-gpu"
      export DOCKER_USERNAME="<your username, gpu-specific credentials provided by Microsoft>"
      export DOCKER_PASSWORD="<your password, gpu-specific credentials provided by Microsoft>"
      export DOCKER_EMAIL="<your email address>"
      export DOCKER_PRIVATE_REGISTRY="1"
      export STORAGE_SIZE="10Gi"
      ```

   1. Deploy the big data cluster:

      ```bash
      mssqlctl cluster create --name gpubigdatacluster
      ```

## Run the TensorFlow example

The following two example notebooks demonstrate training two image classification models on a single node of the Spark cluster using TensorFlow for GPU.

| Notebook download | Description |
|---|---|
| [**tf-cuda8.ipynb**](https://aka.ms/AA4jdgd) | Uses CUDA 8, CUDNN 6, and TensorFlow 1.4.0.  |
| [**tf-cuda9.ipynb**](https://aka.ms/AA4ixzr) | Uses CUDA 9, CUDNN 7, and TensorFlow 1.12.0. |

Place the appropriate notebook file to your local machine, and then open and run it in Azure Data Studio using the PySpark3 kernel. Unless you have a specific need for an older version of CUDA or TensorFlow, choose CUDA 9/CUDNN 7/TensorFlow 1.12.0. For more information about how to use notebooks with big data clusters, see [How to use notebooks in SQL Server 2019 preview](notebooks-guidance.md).

> [!NOTE]
> Note that the notebooks install software in system locations. This is possible because notebooks currently run with root privileges in CTP 2.4.

After installing NVIDIA GPU libraries and TensorFlow for GPU, the notebooks list GPU devices available. They then fit and evaluate a TensorFlow model to recognize handwritten digits using the MNIST data set. After checking available disk space, they download and run the CIFAR 10 image classification example from [https://github.com/tensorflow/models.git](https://github.com/tensorflow/models.git). By running the CIFAR 10 example on clusters having different GPUs, you can observe the speed increase offered by each generation of GPU available in Azure.

## Clean up resources

To delete the big data cluster, use the following command:

```
mssqlctl cluster delete --name gpubigdatacluster
```

The previous command does not remove the AKS cluster. To delete the AKS cluster and the resources associated with it, use the following command:

```azurecli
az group delete -n sqlbigdatagroupgpu
```

## Next steps

For more information about SQL Server 2019 big data clusters (preview), see [What are SQL Server 2019 big data clusters?](big-data-cluster-overview.md).