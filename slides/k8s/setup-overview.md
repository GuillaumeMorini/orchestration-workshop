# Setting up Kubernetes

- Kubernetes is made of many components that require careful configuration

- Secure operation typically requires TLS certificates and a local CA

  (certificate authority)

- Setting up everything manually is possible, but rarely done

  (except for learning purposes)

- Let's do a quick overview of available options!

---

## Local development

- Are you writing code that will eventually run on Kubernetes?

- Then it's a good idea to have a development cluster!

- Instead of shipping containers images, we can test them on Kubernetes

- Extremely useful when authoring or testing Kubernetes-specific objects

  (ConfigMaps, Secrets, StatefulSets, Jobs, RBAC, etc.)

- Extremely convenient to quickly test/check what a particular thing looks like

  (e.g. what are the fields a Deployment spec?)

---

## One-node clusters

- It's perfectly fine to work with a cluster that has only one node

- It simplifies a lot of things:

  - pod networking doesn't even need CNI plugins, overlay networks, etc.

  - these clusters can be fully contained (no pun intended) in an easy-to-ship VM or container image

  - some of the security aspects may be simplified (different threat model)

  - images can be built directly on the node (we don't need to ship them with a registry)

- Examples: Docker Desktop, k3d, KinD, MicroK8s, Minikube

  (some of these also support clusters with multiple nodes)

---

## Managed clusters ("Turnkey Solutions")

- Many cloud providers and hosting providers offer "managed Kubernetes"

- The deployment and maintenance of the *control plane* is entirely managed by the provider

  (ideally, clusters can be spun up automatically through an API, CLI, or web interface)

- Given the complexity of Kubernetes, this approach is *strongly recommended*

  (at least for your first production clusters)

- After working for a while with Kubernetes, you will be better equipped to decide:

  - whether to operate it yourself or use a managed offering

  - which offering or which distribution works best for you and your needs

---

## Node management

- Most "Turnkey Solutions" offer fully managed control planes

  (including control plane upgrades, sometimes done automatically)

- However, with most providers, we still need to take care of *nodes*

  (provisioning, upgrading, scaling the nodes)

- Example with Amazon EKS ["managed node groups"](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html):

  *...when bugs or issues are reported [...] you're responsible for deploying these patched AMI versions to your managed node groups.*

---

## Managed clusters differences

- Most providers let you pick which Kubernetes version you want

  - some providers offer up-to-date versions

  - others lag significantly (sometimes by 2 or 3 minor versions)

- Some providers offer multiple networking or storage options

- Others will only support one, tied to their infrastructure

  (changing that is in theory possible, but might be complex or unsupported)

- Some providers let you configure or customize the control plane

  (generally through Kubernetes "feature gates")

---

## Choosing a provider

- Pricing models differ from one provider to another

  - nodes are generally charged at their usual price

  - control plane may be free or incur a small nominal fee

- Beyond pricing, there are *huge* differences in features between providers

- The "major" providers are not always the best ones!

- See [this page](https://kubernetes.io/docs/setup/production-environment/turnkey-solutions/) for a list of available providers

---

## Kubernetes distributions and installers

- If you want to run Kubernetes yourselves, there are many options

  (free, commercial, proprietary, open source ...)

- Some of them are installers, while some are complete platforms

- Some of them leverage other well-known deployment tools

  (like Puppet, Terraform ...)

- There are too many options to list them all

  (check [this page](https://kubernetes.io/partners/#conformance) for an overview!)

---

## kubeadm

- kubeadm is a tool part of Kubernetes to facilitate cluster setup

- Many other installers and distributions use it (but not all of them)

- It can also be used by itself

- Excellent starting point to install Kubernetes on your own machines

  (virtual, physical, it doesn't matter)

- It even supports highly available control planes, or "multi-master"

  (this is more complex, though, because it introduces the need for an API load balancer)

---

## Manual setup

- The resources below are mainly for educational purposes!

- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) by Kelsey Hightower

  *step by step guide to install Kubernetes on GCP, with certificates, HA...*

- [Deep Dive into Kubernetes Internals for Builders and Operators](https://www.youtube.com/watch?v=3KtEAa7_duA)

  *conference talk setting up a simplified Kubernetes cluster - no security or HA*

- 🇫🇷[Démystifions les composants internes de Kubernetes](https://www.youtube.com/watch?v=OCMNA0dSAzc)

  *improved version of the previous one, with certs and recent k8s versions*

---

## About our training clusters

- How will we set up these Kubernetes clusters that we will use?

--

- We will use `kubeadm` on our VMs running Debian

    1. We already install Docker

    2. We will need to install Kubernetes packages

    3. Run `kubeadm init` on the first node (it deploys the control plane on that node)

    4. Set up  Calico (the overlay network) with two `kubectl apply` commands

    5. Run `kubeadm join` on the other nodes (with the token produced by `kubeadm init`)

    6. Copy the configuration file generated by `kubeadm init`

- More detailed instructions to follow

---

## `kubeadm` "drawbacks"

- Doesn't set up Docker or any other container engine

  (this is by design, to give us choice)

- Doesn't set up the overlay network

  (this is also by design, for the same reasons)

- HA control plane requires [some extra steps](https://kubernetes.io/docs/setup/independent/high-availability/)

- Note that HA control plane also requires setting up a specific API load balancer

  (which is beyond the scope of kubeadm)

???

:EN:- Various ways to install Kubernetes
:FR:- Survol des techniques d'installation de Kubernetes

---

## Lab instructions - preps 1/8

.lab[
On both VMs, edit as root the file `/etc/containerd/config.toml`
He should look like this:
```bash
#disabled_plugins = ["cri"]
version = 2
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd]
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
```
Don't forget to restart containerd with the command `sudo systemctl restart containerd`
]

---

## Lab instructions - preps 2/8

.lab[

On both VMs

```bash
sudo swapoff -a
```

]

---

## Lab instructions - master 3/8

.lab2[

On both VMs
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

On the first VM
You can then init the Kubernetes cluster:
```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
```

]

---

## Lab instructions - output master 4/8

.lab[

You should have a message similar to this one

```bash

Your Kubernetes control-plane has initialized successfully!

....

Then you can join any number of worker nodes by running this on each as root:

kubeadm join 172.16.25.4:6443 --token 43fmlr.ng2dscnjrjsxom45 
	--discovery-token-ca-cert-hash sha256:3545364e549f 

```
Copy the last 2 lines for later

]

---

## Lab instructions - check master 5/8

.lab[


Check everything is working well, you should have
```bash
$ mkdir -p ~/.kube
$ sudo cp /etc/kubernetes/admin.conf ~/.kube/config
$ sudo chmod 0644 ~/.kube/config
$ kubectl get nodes
NAME             STATUS     ROLES           AGE   VERSION
1224bdebstd002   NotReady   control-plane   13s   v1.32.2
$ 
```

Still on the first VM, allow workload to run on the control plane
```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

]

---

## Lab instructions - add a node 6/8

.lab[

On the second VM, reuse the 2 lines you copied previously
```bash
kubeadm join xxxxxx --token xxxx --discovery-token-ca-cert-hash xxxx
```


]

---

## Lab instructions - install CNI 7/8

.lab3[



Finally, on the first VM, install the CNI
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml
```
]

---

## Lab instructions - check CNI 8/8

.lab3[

Check if everything works well, after few minutes
```bash
$ kubectl get pods -A
NAMESPACE          NAME                                       READY   STATUS    RESTARTS   AGE
calico-apiserver   calico-apiserver-9657959cd-rztkj           1/1     Running   0          23m
calico-apiserver   calico-apiserver-9657959cd-zb7wx           1/1     Running   0          23m
calico-system      calico-kube-controllers-79655dc7cf-8zxcl   1/1     Running   0          23m
calico-system      calico-node-7q9bm                          1/1     Running   0          23m
calico-system      calico-node-jhbwp                          1/1     Running   0          23m
calico-system      calico-typha-cbc6667cf-mv7kr               1/1     Running   0          23m
calico-system      csi-node-driver-fnq5r                      2/2     Running   0          23m
calico-system      csi-node-driver-k5sxj                      2/2     Running   0          23m
kube-system        coredns-668d6bf9bc-2kfhq                   1/1     Running   0          32m
kube-system        coredns-668d6bf9bc-4zsjx                   1/1     Running   0          32m
kube-system        etcd-1224bdebstd002                        1/1     Running   23         32m
kube-system        kube-apiserver-1224bdebstd002              1/1     Running   27         32m
kube-system        kube-controller-manager-1224bdebstd002     1/1     Running   0          32m
kube-system        kube-proxy-5w926                           1/1     Running   0          32m
kube-system        kube-proxy-fh5l8                           1/1     Running   0          31m
kube-system        kube-scheduler-1224bdebstd002              1/1     Running   26         32m
tigera-operator    tigera-operator-ccfc44587-fnzb4            1/1     Running   0          23m
$ 
```

]

---

class: pic
![](images/congrats.gif)

