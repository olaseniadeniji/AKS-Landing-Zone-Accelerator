## Architecting for High Availability & Resiliency

![Plan Backup Restore](./media/plan_backup_restore.png)

First, check out [Best practices for business continuity and disaster recovery in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-multi-region)

## High Availability Considerations

Checkout the repo section on [High Availability Baseline](https://github.com/Azure/AKS-Landing-Zone-Accelerator/tree/velero-backup-restore/Scenarios/High-Availability-Baseline)

* **AKS Cluster Configuration**:
	- Enable [Uptime SLA](https://docs.microsoft.com/en-us/azure/aks/uptime-sla) for production workloads
	- Use [Availability Zones](https://docs.microsoft.com/en-us/azure/aks/availability-zones) (with Standard Load Balancer)
	- Use [multiple node pools](https://docs.microsoft.com/en-us/azure/aks/use-multiple-node-pools) spanning AZs
	- Enforce [Resource Quotas](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-scheduler#enforce-resource-quotas) and Plan for [pod disruption budgets](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-scheduler#plan-for-availability-using-pod-disruption-budgets)
	- Control Pod scheduling using [Taints & Tolerations](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-advanced-scheduler#provide-dedicated-nodes-using-taints-and-tolerations), & [Pod Affinity](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-advanced-scheduler#control-pod-scheduling-using-node-selectors-and-affinity)



* **Applications**: 
  - Configure applications [requests & limits](https://docs.microsoft.com/en-us/azure/aks/developer-best-practices-resource-management#define-pod-resource-requests-and-limits)
  - to ensure the PVs are located in the same zone as the pods:
     - Use Volume Binding Mode: WaitForFirstConsumer (In your storage classes)
     - Use StatefulSets
     - Use Zone-Redundant (ZRS) Disks (preview)
   - to optimally route traffic within the same zone to avoid unnecessary latency: 
      - Use Service Topology (deprecated in Kubernetes 1.21, +, feature state alpha)
      - Use Topology Aware Hints (from Kubernetes 1.21+, feature state alpha)


* **Data**: 
Storage Class Configuration (used to create dynamic peristent volumes)
	- Use CSI Driver as it is the standard provider for exposing storage to applications running on Kubernetes
	- Use Azure Disk with ZRS (currently in Preview) --> available via Azure Disk CSI Driver
	- Use Azure File with ZRS


## Backup & Restore Considerations

* Plan for Backup & Restore
	-  Prepare Cluster & POD Identities
	- Plan network segmentation & DNS resolution
	- Prepare Subscription for storing backups (optional but recommended)
	- Prepare storage location in Backup Region to store backups

	- Prepare Cluster Node Pools :
	  - Create Nodes & re-deploy Node Configuration
	  - Use Automated configuration using CICD or [GitOps!](https://docs.microsoft.com/en-us/azure/azure-arc/kubernetes/conceptual-gitops-flux2)


	-  Prepare Applications Persistent volumes : 
	  	-  Prepare StorageClasses & VolumeSnapshotClasses 

* Run a Drill Tests:
	* Create secondary AKS ecosystem (ACR, Keyvault, App Gateway, Firewall, NSG)
	* Create secondary AKS Cluster (with its dependencies installed: aad-podid, velero, csi-drivers) + RBAC for Azure services & velero identity (backup tool)

	To restore **Stateless** Application: 
	* Redeploy Application Configuration using Devops CICD

	To restore **stateful** Application, you need to backup and restore:
	* Cluster configuration (storage classes, volumesnapshotclasses, technical pods)
	*  Persistent Volumes (Azure Disk & Azure Fileshare)
	*  Application Configuration (bound to the restored persistent volumes)
	
	➡️ A tool such as Velero simplifies the process fo backup & restore for stateful applications
	
	➡️ **Coming Soon!** Perform Backup for Persistent Volume of AKS clusters using [Azure Backup](https://azure.microsoft.com/en-us/updates/akspvbackupprivatepreview/)
	
	:arrow_forward: [Deep Dive on Velero configuration for AKS](./velero_terraform_sample)




