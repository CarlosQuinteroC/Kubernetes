# Install Kubernetes

1. Click the cp tab to access the cp node. Become root and update and upgrade the system. You may be asked a few questions. If so, allow restarts and keep the local version currently installed. Which would be a yes then a 2.

```shell
student@cp:~$ sudo -i 
```

```shell
root@cp:~#

apt update && apt upgrade -y`
```

```console
Expected Output

<output_omitted>
```
2. The main choices for a container environment are **containerd, cri-o, and Docker** on older clusters. We suggest containerd for class, as it is easy to deploy and commonly used by cloud providers.
Please note, install one engine only. If more than one are installed the kubeadm init process search pattern will use Docker at the moment. Also be aware that engines other than containerd may show different output on some commands.

3. There are several packages we should install to ensure we have all dependencies take care of. Please note the backslash is not necessary and can be removed if typing on a single line.

```console
root@cp:~# 
apt install -y apt-transport-https tree \
software-properties-common ca-certificates socat
```
```console
Expected Output

<output_omitted>
```
4. Disable swap if not already done. Cloud providers disable swap on their images.

```console
root@cp:~#

swapoff -a
```

5. Load modules to ensure they are available for following steps.

```console
root@cp:~#

modprobe overlay
modprobe br_netfilter
```

6. Update kernel networking to allow necessary traffic. Be aware the shell will add a greater than sign (>) to indicate the command continues after a carriage return.

```console
root@cp:~#

cat << EOF | tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

```console
Expected Output

net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```
7. Ensure the changes are used by the current kernel as well

```console
root@cp:~#

sysctl --system
```
```console
Expected Output

* Applying /etc/sysctl.d/10-console-messages.conf ...
kernel.printk = 4 4 1 7
* Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
* Applying /etc/sysctl.d/10-kernel-hardening.conf ...
kernel.kptr_restrict = 1
<output_omitted>
```
8. Install the necessary key for the software to install

```console
root@cp:~#

mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
```console
root@cp:~#

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
  ```

9. Install the containerd software.

```console
root@cp:~#

apt update && apt install -y containerd.io
```
```console
root@cp:~#

containerd config default | tee /etc/containerd/config.toml
```
```console
root@cp:~#

sed -e 's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml
```
```console
root@cp:~#

systemctl restart containerd
```

10. Download the public signing key for the Kubernetes package repositories.

```console
root@cp:~#

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

11. Add the appropriate Kubernetes apt repository. Please note that this repository have packages only for Kubernetes 1.34; for other Kubernetes minor versions, you need to change the Kubernetes minor version in the URL to match your desired minor version

```console
root@cp:~#

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /" \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
  ```

12. Update with the new repo declared, which will download updated repo information.

```console
root@cp:~#

apt update
```

13. Install the Kubernetes software. There are regular releases, the newest of which can be used by omitting the equal sign and version information on the command line. Historically new versions have lots of changes and a good chance of a bug or five. As a result we will hold the software at the recent but stable version we install. In a later lab we will update the cluster to a newer version.

```console
root@cp:~#

apt install -y kubeadm=1.34.2-1.1 kubelet=1.34.2-1.1 kubectl=1.34.2-1.1
```
```console
Expected Output

<output_omitted>
```

14. Mark the packages to hold at current version.

```console
root@cp:~#

apt-mark hold kubelet kubeadm kubectl
```

15. Determine the IP address of the cp node and note it for later use. Note: The output shown in the Expected Output box is only a sample and is provided for reference.


```console
root@cp:~#

hostname -i
```
```console
Expected Output

10.244.0.3
```
```console
root@cp:~#

ip addr show
```
```console
Expected Output

....
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
    link/ether 42:01:0a:80:00:18 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.3/32 brd 10.244.0.3 scope global ens4
       valid_lft forever preferred_lft forever
    inet6 fe80::4001:aff:fe80:18/64 scope link
       valid_lft forever preferred_lft forever
....
```
16. Add a local DNS alias for our cp server. Edit the /etc/hosts file and add the above IP address and assign a name k8scp.

```console
root@cp:~#

vim /etc/hosts
```
```console
Expected Output

10.244.0.3 k8scp      #<-- Add this line
10.244.0.3 cp         #<-- Add this line
127.0.0.1 localhost
....
```

17. It is customary to create a configuration file to initialize the cluster. There are many options we could include, and they differ for containerd, Docker, and cri-o. Use the configuration file included in the course tarball. After our cluster is initialized we will view other default values used. Be sure to use the node alias we added to /etc/hosts, not the IP so the network certificates will continue to work when we deploy a load balancer in a future lab. The file is already included in the course tarball and does not require any changes.

```console
root@cp:~#

cp /home/student/LFS258/SOLUTIONS/s_03/kubeadm-config.yaml /root/
```
```console
root@cp:~#

cat kubeadm-config.yaml
```
```yaml
kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
kubernetesVersion: 1.34.2                     
controlPlaneEndpoint: "k8scp:6443"            #<-- Use the alias we put in /etc/hosts not the IP
networking:
  podSubnet: 192.168.0.0/16
  ```
18. Initialize the cp. Scan through the output. Expect the output to change as the software matures. At the end are configuration directions to run as a non-root user. The token is mentioned as well. This information can be found later with the kubeadm token list command. The output also directs you to create a pod network to the cluster, which will be our next step. Pass the network settings Cilium has in its configuration file. Please note: the output lists several commands which following exercise steps will complete.

```console
root@cp:~#

kubeadm init \
--config=kubeadm-config.yaml \
--upload-certs \
--node-name=cp \
| tee kubeadm-init.out 
```
```
Expected Output

[init] Using Kubernetes version: v1.34.2
[preflight] Running pre-flight checks

<output_omitted>


You can now join any number of the control-plane node
running the following command on each as root:

kubeadm join k8scp:6443 --token vapzqi.et2p9zbkzk29wwth \
  --discovery-token-ca-cert-hash
  sha256:f62bf97d4fba6876e4c3ff645df3fca969c06169dee3865aab9d0bca8ec9f8cd \
  --control-plane --certificate-key
  911d41fcada89a18210489afaa036cd8e192b1f122ebb1b79cce1818f642fab8

Please note that the certificate-key gives access to cluster sensitive
data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If
necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following
on each as root:

kubeadm join k8scp:6443 --token vapzqi.et2p9zbkzk29wwth \
  --discovery-token-ca-cert-hash
    sha256:f62bf97d4fba6876e4c3ff645df3fca969c06169dee3865aab9d0bca8ec9f8cd
```

19. As suggested in the directions at the end of the previous output we will allow a non-root user admin level access to the cluster. Take a quick look at the configuration file once it has been copied and the permissions fixed.

```console
root@cp:~#

exit
```

```console
Expected Output

logout
```
```console
student@cp:~$

mkdir -p $HOME/.kube
```
```console
student@cp:~$

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
```console
student@cp:~$
```
```console
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
```console
student@cp:~$

less .kube/config
```
```console
Expected Output

apiVersion: v1
clusters:
- cluster:
<output_omitted>
```

20. Deciding which pod network to use for Container Networking Interface (CNI) should take into account the expected demands on the cluster. There can be only one pod network per cluster, although the Multus CNI project is trying to change this.

The network must allow container-to-container, pod-to-pod, pod-to-service, and external-to-service communications. We will use Cilium as a network plugin which will allow us to use Network Policies later in the course. Currently Cilium does not deploy using CNI by default.

```
Cilium Installation Reference

Cilium is genereally installed using "cilium install" or using "helm install" commands. We have generated the cilium-cni.yaml file using the below commands for your convenience. Note: You dont need to execute the commands in this box, they are just for reference.


$ helm repo add cilium https://helm.cilium.io/
$ helm repo update
$ helm template cilium cilium/cilium --version 1.19.1 \
   --namespace kube-system > cilium.yaml
student@cp:~$

find $HOME -name cilium-cni.yaml
```

```console
student@cp:~$

kubectl apply -f /home/student/LFS258/SOLUTIONS/s_03/cilium-cni.yaml
```
```console
Expected Output

serviceaccount/cilium created
serviceaccount/cilium-operator created
secret/cilium-ca created
configmap/cilium-config created
<output_omitted>
```

21. While many objects have short names, a kubectl command can be a lot to type. We will enable bash auto-completion. Begin by adding the settings to the current shell. Then update the $HOME/.bashrc file to make it persistent. Ensure the bash-completion package is installed. If it was not installed, log out then back in for the shell completion to work.

```console
student@cp:~$

sudo apt install -y bash-completion
Expected Output

<exit and log back in>
```
```console
student@cp:~$

source <(kubectl completion bash)
```

```console
student@cp:~$

echo "source <(kubectl completion bash)" >> $HOME/.bashrc
```
```console
student@cp:~$

source $HOME/.bashrc
```

22. Test by describing the node again. Type the first three letters of the sub-command then type the Tab key. Auto-completion assumes the default namespace. Pass the namespace first to use auto-completion with a different namespace. By pressing Tab multiple times you will see a list of possible values. Continue typing until a unique name is used. First look at the current node (your node name may not start with cp), then look at pods in the kube-system namespace. If you see an error instead such as -bash: _get_comp_words_by_ref: command not found revisit the previous step, install the software, log out and back in.

```console
student@cp:~$

kubectl des<Tab> n<Tab><Tab> cp<Tab>
```
```console
student@cp:~$

kubectl -n kube-s<Tab> g<Tab> po<Tab>
23. Explore the kubectl help command. The output has been omitted from commands. Take a moment to review help topics.
```
```console
student@cp:~$

kubectl help
```
```console
student@cp:~$

kubectl help create
```

24. View other values we could have included in the kubeadm-config.yaml file when creating the cluster.

```console
student@cp:~$

sudo kubeadm config print init-defaults
```
```console
Expected Output

apiVersion: kubeadm.k8s.io/v1beta4
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
<output_omitted>
```



kubeadm join k8scp:6443 --token 1wd7xo.yakf0tzec28sb1cb \
        --discovery-token-ca-cert-hash sha256:0b962e7303338b99e94adb1b231f1a803fb8a1cf6fbfe00e68d51c7adeb81899