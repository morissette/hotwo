# How to run k8s at home
 If you want to run a full on kubernetes cluster at home (so you can do reall testing, like draining nodes, etc) you're going to need more than minikube. 
 
 The good news is that it's really easy as long as you can spin up 3 ubuntu 16.04 vm's. I run kvm/libvirt and manage my vm's with virt-manager. Setup of that is beyond the scope of this document (see [Ubuntu Documentation](https://help.ubuntu.com/community/KVM/Installation) ) for                                                          
 
## Prerequiretes
- 3 Ubuntu16.04 vm's.
- Docker
- Kubeadm
- 32GB of disk space per node
 
 
## Install the  Prerequisites
Follow [this](https://kubernetes.io/docs/tasks/tools/install-kubeadm/) guide to install the prerequisites
```https://kubernetes.io/docs/tasks/tools/install-kubeadm/```


## Setup your cluster
On the master:

`kubeadm init  --pod-network-cidr=192.168.0.0/16`

`export KUBECONFIG=/etc/kubernetes/admin.conf`

 **Save output** *it should look something like below*: 
 `kubeadm join 10.0.0.168:6443 --token s29wj8.0ejnla7c46ikb41d --discovery-token-ca-cert-hash sha256:08860729203944ede7718b509bafa3ff8a0941d10b4c8f734ebfe74963713172`

#### Enable Calico Networking
1. Deploy Calico Pod
`kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml`
2. Wait for pods to become ready
`watch kubectl get pods --all-namespaces`
3. Allow pods on master
`kubectl taint nodes --all node-role.kubernetes.io/master-`
4. Join worker nodes with the output from `kubeadm init` above.
5. Verify that all nodes are ready
`kubectl get nodes -o wide`

#### Add other nodes to cluser
`kubeadm join 10.0.0.168:6443 --token s29wj8.0ejnla7c46ikb41d --discovery-token-ca-cert-hash sha256:08860729203944ede7718b509bafa3ff8a0941d10b4c8f734ebfe74963713172`


#### Add routes for cluster-ip-range
On the master, in `/etc/kubernetes/manifests/kube-apiserver.yaml` you will find `--service-cluster-ip-range=.....` 
Add this subnet to your routes so you can access services in your cluster.

#### Create your first deployment
`kubectl run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0 --port=8080`

#### Expose your service
`kubectl expose deployment hello-world –type=NodePort –name=example-service`

## k8s Dasboard
1. Install the pod
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml`

2. Grant permission to the dashboard
```sh
echo "apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system" > perms.yaml
```
`kubectl create -f ./perms.yaml`

Conenct to dashboard
`kubectl proxy`
`http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/`


## In case yo need to start over
Do this on all the nodes, then the master
`kubeadm reset`
