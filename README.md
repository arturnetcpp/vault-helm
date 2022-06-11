# vault-helm

The vault helm chart that we will be using creates vault serviceaccount, clusterrole, clusterrolebinding, statefulset, configmap, secret, services and pvc. But before using the helm chart to create kubernetes resources for vault, we need to configure storage class and nfs client provisioner, which is an automatic provisioner for Kubernetes that uses your already configured NFS server, automatically creating Persistent Volumes.

To create the storage class and nfs client provisioner:

helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=$NFS_SERVER_IP --set nfs.path=/nfs/helm/vault

Form the storage class we need to take the storage class name, which in this case is nfs-client and use it in our helm chart to dynamically create persistent volumes.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Installing the Chart
To install the vault chart use the following command:

helm repo add hashicorp https://helm.releases.hashicorp.com

Before deploying vault we need to change its default values including vault storage type, nodeport, configuration data. For that we create vault-values.yml, where we add the configuration data we want to change.

helm install vault hashicorp/vault -n $Namespace --values vault-values.yml

or create yaml file from vault-values.yml which will contain deployment, svc, configmaps, secrets, etc. and store it in templates folder:

helm template vault hashicorp/vault --namespace $Namespace -f vault-values.yml > templates/vault_resources.yml

and then create the resources using:

kubectl apply -f templates/vault_resources.yml -n $Namespace

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Now Vault is up and running and with this configuration it can be accessed via http://$Master_IP:30000
However, it is still uninitialized and unsealed.
To initialized vault:

kubectl exec -n $Namespace vault-0 -- vault operator init

Now we need to enable automatic unsealing process for vault. To do that install the following helm chart:

helm repo add omegion https://charts.omegion.dev

helm repo update

helm search repo omegion/vault-unseal-cronjob

This helm chart will create a cronjob that will check every x time whether vault is sealed or not and unseal it with the Shards provided in vault-unseal-values.yml file.

helm install -n $Namespace vault-unseal-cronjob omegion/vault-unseal-cronjob --values vault-unseal-values.yml

or again create yaml file from vault-unseal-values.yml which will contain our cronjob yaml file and store it in templates folder:

helm template omegion/vault-unseal-cronjob --namespace $Namespace -f vault-unseal-values.yml > templates/vault_auto_unseal.yml

and then create the resources using:

kubectl apply -f templates/vault_auto_unseal.yml -n $Namespace
