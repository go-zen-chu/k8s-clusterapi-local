# k8s-clusterapi-local

local manifest for k8s clusterapi

## environment

- WSL2 in Windows

## setup

```bash
# https://cluster-api.sigs.k8s.io/user/quick-start
$ curl -L https://github.com/kubernetes-sigs/cluster-api/releases/download/v1.5.3/clusterctl-linux-amd64 -o clusterctl
$ chmod +x clusterctl
$ mv clusterctl /usr/local/bin/

# run docker desktop k8s
$ kubectl cluster-info
Kubernetes control plane is running at https://kubernetes.docker.internal:6443
CoreDNS is running at https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

# Enable the experimental Cluster topology feature.
$ export CLUSTER_TOPOLOGY=true
# Initialize the management Cluster
$ clusterctl init --infrastructure docker

Fetching providers
Installing cert-manager Version="v1.12.2"
Waiting for cert-manager to be available...
Installing Provider="cluster-api" Version="v1.5.3" TargetNamespace="capi-system"
Installing Provider="bootstrap-kubeadm" Version="v1.5.3" TargetNamespace="capi-kubeadm-bootstrap-system"
Installing Provider="control-plane-kubeadm" Version="v1.5.3" TargetNamespace="capi-kubeadm-control-plane-system"
Installing Provider="infrastructure-docker" Version="v1.5.3" TargetNamespace="capd-system"

Your management cluster has been initialized successfully!

You can now create your first workload cluster by running the following:

  clusterctl generate cluster [name] --kubernetes-version [version] | kubectl apply -f -

$ kubectl get pods -A | grep -v kube-system
NAMESPACE                           NAME                                                             READY   STATUS    RESTARTS   AGE
capd-system                         capd-controller-manager-684b75b6f-b5xzb                          1/1     Running   0          6m16s
capi-kubeadm-bootstrap-system       capi-kubeadm-bootstrap-controller-manager-5848ccdddd-9xr4j       1/1     Running   0          6m16s
capi-kubeadm-control-plane-system   capi-kubeadm-control-plane-controller-manager-6fdc54867d-snx4b   1/1     Running   0          6m16s
capi-system                         capi-controller-manager-5dc576d7d-gdqg7                          1/1     Running   0          6m16s
cert-manager                        cert-manager-5b697c744b-wlhsp                                    1/1     Running   0          6m23s
cert-manager                        cert-manager-cainjector-ff499b874-76khs                          1/1     Running   0          6m23s
cert-manager                        cert-manager-webhook-77db69f7d8-4rxrx                            1/1     Running   0          6m23s

# quickstart deployment
$ clusterctl generate cluster capi-quickstart --flavor development \
  --kubernetes-version v1.28.0 \
  --control-plane-machine-count=3 \
  --worker-machine-count=3 \
  > capi-quickstart.yaml
# apply
$ kubectl apply -f capi-quickstart.yaml

# どういうリソースがあるか確認
$ kubectl api-resources --namespaced=true --verbs=list --output=name | grep "cluster.x-k8s.io"
clusterresourcesetbindings.addons.cluster.x-k8s.io
clusterresourcesets.addons.cluster.x-k8s.io
kubeadmconfigs.bootstrap.cluster.x-k8s.io
kubeadmconfigtemplates.bootstrap.cluster.x-k8s.io
clusterclasses.cluster.x-k8s.io
clusters.cluster.x-k8s.io
machinedeployments.cluster.x-k8s.io
machinehealthchecks.cluster.x-k8s.io
machinepools.cluster.x-k8s.io
machines.cluster.x-k8s.io
machinesets.cluster.x-k8s.io
providers.clusterctl.cluster.x-k8s.io
kubeadmcontrolplanes.controlplane.cluster.x-k8s.io
kubeadmcontrolplanetemplates.controlplane.cluster.x-k8s.io
dockerclusters.infrastructure.cluster.x-k8s.io
dockerclustertemplates.infrastructure.cluster.x-k8s.io
dockermachinepools.infrastructure.cluster.x-k8s.io
dockermachines.infrastructure.cluster.x-k8s.io
dockermachinetemplates.infrastructure.cluster.x-k8s.io
ipaddressclaims.ipam.cluster.x-k8s.io
ipaddresses.ipam.cluster.x-k8s.io

$ kubectl get cluster
NAME              PHASE          AGE    VERSION
capi-quickstart   Provisioning   5m2s   v1.28.0

$ clusterctl describe cluster capi-quickstart
NAME                                                           READY  SEVERITY  REASON                           SINCE  MESSAGE                                                                                 
Cluster/capi-quickstart                                        False  Warning   ScalingUp                        8m14s  Scaling up control plane to 3 replicas (actual 0)                                        
├─ClusterInfrastructure - DockerCluster/capi-quickstart-j4gtv  False  Warning   LoadBalancerProvisioningFailed   8m8s   0 of 1 completed                                                                         
├─ControlPlane - KubeadmControlPlane/capi-quickstart-7qfqv     False  Warning   ScalingUp                        8m14s  Scaling up control plane to 3 replicas (actual 0)                                        
└─Workers                                                                                                                                                                                                        
  └─MachineDeployment/capi-quickstart-md-0-hmjph               False  Warning   WaitingForAvailableMachines      8m14s  Minimum availability requires 3 replicas, current 0 available                            
    └─3 Machines...                                            False  Info      WaitingForClusterInfrastructure  8m14s  See capi-quickstart-md-0-hmjph-csv57-d89q6, capi-quickstart-md-0-hmjph-csv57-jkvqg, ... 
```