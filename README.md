# vault-helm

The vault helm chart that we will be using creates vault serviceaccount, clusterrole, clusterrolebinding, statefulset, configmap, secret, services and pvc. But before using the helm chart to create kubernetes resources for vault, we need to configure storage class.

To create the storage class:

helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=$NFS_SERVER_IP --set nfs.path=/nfs/helm/vault

Form the storage class we need to take the storage class name, which in this case is nfs-client and use it in our helm chart to dynamically create persistent volumes.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Installing the Chart
To install the vault chart use the following command:
helm repo add hashicorp https://helm.releases.hashicorp.com

Before deploying vault we need to change its default values including vault storage type, nodeport, configuration data. For that we create vault-values.yml, where we add the configuration data we want to change.
helm install vault hashicorp/vault -n prod5 --values vault-values.yml

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Now Vault is up and running and with this configuration it can be accessed via http://$Master_IP:30000
However, it is still uninitialized and unsealed.
To initialized vault:
kubectl exec -n prod5 vault-0 -- vault operator init

Now we need to enable automatic unsealing process for vault. To do that install the following helm chart:
helm repo add omegion https://charts.omegion.dev
helm repo update
helm search repo omegion/vault-unseal-cronjob

This helm chart will create a cronjob that will check every x time whether vault is sealed or not and unseal it with the Shards provided in vault-unseal-values.yml file.
helm install -n prod5 vault-unseal-cronjob omegion/vault-unseal-cronjob --values vault-unseal-values.yml
