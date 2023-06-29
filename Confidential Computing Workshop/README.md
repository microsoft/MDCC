# Azure Confidential Computing powered by AMD SEV-SNP - Workshop

>In this technical workshop, you will gain a comprehensive understanding of Azure's Confidential Computing capabilities. We will take you on a journey through deploying confidential VMs, generating attestations, validating report authenticity, and deploying confidential containers on Azure Container Instances and Azure Kubernetes Services. To ensure you are well-prepared, we have a dedicated section that walks you through the installation of all necessary requirements. 

## Workshop agenda

### 🌅 Morning (9:00 – 12:00)

> *Focus: Introduction and first steps*

- 09:00 – 09:30: Introduction to Confidential Computing
- 09:45 – 10:00: Azure Confidential Computing Technologies
- 10:00 – 10:15: Azure Confidential VMs with AMD SEV-SNP 
- 10:15 – 10:30: Break
- 10:30 – 11:00: Hands-on lab: Deploying a confidential VM
- 11:00 – 11:30: Overview of Attestation in AMD SEV-SNP
- 11:30 – 12:00: Hands-on lab: Generating and validating attestation report

### 🍽️ Lunch Break (12:00 – 13:00)

### 🌇 Afternoon (13:00 – 16:30)

- 13:00 – 13:45: Confidential Containers with ACI
- 13:45 – 14:00: Break
- 14:00 – 15:00: Hands-on lab: Deploying confidential containers on ACI 
- 14:45 – 15:00: Break
- 15:00 – 15:30: Confidential Containers on AKS

## Preparation

>This section is crucial to ensure a smooth run of the hands-on labs. If you're only here for the presentations, feel free to skip this.

### Azure subscription and deployments

Please ensure that you have an active Azure subscription. If not, you can [create one here](https://azure.microsoft.com/en-us/free/).

### Environment setup

Below we list all the requirements for this workshop. Be sure to set up everything in advance to avoid delays.

#### 1️⃣ Installation Requirements

Please follow the steps outlined in the [requirements notebook](/exercices/requirements.ipynb) to set up your environment, which includes:

* Installing [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) on Windows.
* Installing [Visual Studio Code](https://code.visualstudio.com/download).
* Installing [Docker Desktop](https://www.docker.com/products/docker-desktop) and creating a Docker account.
* Setting up Windows Subsystem for Linux (WSL). You can follow [these instructions](https://docs.microsoft.com/en-us/windows/wsl/install-win10) from Microsoft.

-------------------

## Notebooks

* :muscle: [Confidential VM](exercices/cvm_deployment.md) - Deploying a confidential VM on Azure.
* :muscle: [Confidential VM Attestation](exercices/cvm_attestation.ipynb) - Generating an attestation and validating its authenticity.
* :muscle: [Confidential VM Attestation Deepdive](exercices/cvm_attestation_deepdive.md) - Fetching and verifying a raw sev-snp report
* :muscle: [Confidential ACI](exercices/confidential_containers_cce.md) - Deploying a confidential container on ACI.
* :muscle: [Confidential ACI Deepdive](exercices/confidential_containers_deepdive_cce.ipynb) - Checking CCE policy enforcement on ACI.
