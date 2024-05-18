# Installing longhorn on GKE

This is a small guidance on setting up [Longhorn](https://longhorn.io/) on GKE. Longhorn supports ReadWriteMany(RWX) volumes by exposing regular Longhorn volumes via NFSv4 servers that reside in share-manager pods. Without Longhorn, we only acquire RWX on GKE by using Filestore.

# Prerequisites
1. Nodepool's image must be Ubuntu.
2.  Cluster admin permission.

# Installing Longhorn
1. Add the Longhorn Helm repository
	```
	helm repo add longhorn https://charts.longhorn.io
	```
2. Fetch the latest charts from the repository
	```
	helm repo update
	```
3. Install Longhorn in the `longhorn-system` namespace.
	```
    helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --version 1.6.1 -f values.yaml
    ```
    > **Note**: **values.yaml** is used to override Longhorn default values.
    
    **Example values.yaml**:
    ```
	longhornManager:
	  nodeSelector:
	    key: value
	  tolerations:
	    - effect: NoSchedule
	      key: key-1
	      operator: Equal
	      value: value-1
	    - effect: NoSchedule
	      key: key-2
	      operator: Equal
	      value: value-2
	longhornDriver:
	  nodeSelector:
	    key: value
	  tolerations:
		- effect: NoSchedule
	      key: key-1
	      operator: Equal
	      value: value-1
	    - effect: NoSchedule
	      key: key-2
	      operator: Equal
	      value: value-2
	longhornUI:
	  nodeSelector:
	    key: value
	  tolerations:
		- effect: NoSchedule
	      key: key-1
	      operator: Equal
	      value: value-1
	    - effect: NoSchedule
	      key: key-2
	      operator: Equal
	      value: value-2
	defaultSettings:
	  taintToleration: "key-1=value-1:NoSchedule; key-2=value-1:NoSchedule"
	  systemManagedComponentsNodeSelector: "key:value"
	  createDefaultDiskLabeledNodes: true
	persistence:
	  defaultNodeSelector:
	    enable: true
	    selector: 'storage'
	```
>**Note:** If you want to use Longhorn on multiple nodepools, you have to specify enough nodeSelector and taintToletation that macth all nodepools.
# Uninstalling Longhorn
1. Set the flag [deleting-confirmation-flag](https://longhorn.io/docs/1.6.1/references/settings/#deleting-confirmation-flag) to true
	```
	kubectl -n longhorn-system patch -p '{"value": "true"}' --type=merge lhs deleting-confirmation-flag
	```
2.  Uninstall Longhorn
	```
	helm uninstall longhorn -n longhorn-system
	```
# Appendix
- When using RWX, Longhorn create a dedicated pod **share-manager** for each RWX PVC, this pod use a lot of memory and if it dies all pods mounting PVC will be restarted.
- If you want to use only some nodes in your cluster as storage set the property  **createDefaultDiskLabeledNodes** in value file to **true** and add label **node.longhorn.io/create-default-disk=true** to your node. 