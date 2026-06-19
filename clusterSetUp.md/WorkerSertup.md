# Add a Worker Node
1. Become root on the worker node.

```console
student@worker:~$

sudo -i
```

2. Update and upgrade the system.
```Console
root@worker:~#

apt update && apt upgrade -y
```

3. Install the containerd engine, starting with dependent software.
```console
root@worker:~#

apt install -y apt-transport-https \
software-properties-common ca-certificates tree socat
```
```console
root@worker:~#

swapoff -a
```
```console
root@worker:~#

modprobe overlay
modprobe br_netfilter
```
```console
root@worker:~#

cat << EOF | tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

```console
root@worker:~#

sysctl --system
```
```console
root@worker:~#

mkdir -p /etc/apt/keyrings
```
``` console
root@worker:~#

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
```console
root@worker:~#

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```console
root@worker:~#

apt update && apt install -y containerd.io

```
```console
root@worker:~#

containerd config default | tee /etc/ontainerd/config.toml
```
```console
root@worker:~#

sed -e 's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml
```
```console
root@worker:~#

systemctl restart containerd
```
4. Get the GPG key for the software.

```console
root@worker:~#

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

5. Add Kubernetes repo.

```console
root@worker:~#

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```

6. Update repos then install the Kubernetes software. Be sure to match the version on the cp.
```console
root@worker:~#

apt update
```

7. Install kubeadm, kubelet, and kubectl.
```
root@worker:~#

apt install -y kubeadm=1.34.2-1.1 kubelet=1.34.2-1.1 kubectl=1.34.2-1.1
```

8. Ensure the version remains if the system is updated.
```console
root@worker:~#

apt-mark hold kubeadm kubelet kubectl
```

## Attention

*The following few steps should be executed on the cp node. Click the cp tab to return, temporarily, to the cp node.*

9. Find the IP address of your cp server. The interface name will be different depending on where the node is running. Currently inside of GCE the primary interface for this node type is ens4. Your interfaces names may be different. From the output we know our cp node IP is 10.244.0.3.

```console
student@cp:~$

hostname -i
```
```
Expected Output
10.244.0.3
```
```console
student@cp:~$

ip addr show | grep inet
```
```console
Expected Output

inet 10.244.0.3/32 brd 10.244.0.3 scope global ens4
inet6 fe80::4001:aff:fe8e:2/64 scope link
```

10. At this point we could copy and paste the join command from the cp node. That command only works for 2 hours, so we will build our own join should we want to add nodes in the future. Find the token on the cp node. The token lasts 2 hours by default. If it has been longer, and no token is present you can generate a new one with the sudo kubeadm token create command, presented below.

```console
student@cp:~$

sudo kubeadm token create --print-join-command
```
```console
Expected Output

kubeadm join k8scp:6443 --token kcu55w.7jso85i0e2dsn05y \
--discovery-token-ca-cert-hash \
sha256:0fb62b3c47bfd3af3c15d21f2ab6082fad1f913b244d5980816f8147ce9936ef
```

## Attention

*Make a note of your cp instance IP address and your newly generated join command in its entirety, and return once again to the worker instance by clicking the worker tab. The following commands are executed on the worker node.*

11. On the worker node add a local DNS alias for the cp server. Edit the /etc/hosts file and add the cp IP address and assign the name k8scp. The entry should be exactly the same as the edit on the cp.

```console
root@worker:~#

vim /etc/hosts
```
```console
/etc/hosts
10.244.0.3 k8scp    #<-- Add this line
10.244.0.3 cp       #<-- Add this line
127.0.0.1 localhost
....
```

12. Use the the join command from init output from cp. Note: append the --node-name=worker option to the join command.
```console
root@worker:~#
kubeadm join \
k8scp:6443 --token wcal99.hxv9v0gtnz42g6dr \
--discovery-token-ca-cert-hash \
sha256:0fb62b3c47bfd3af3c15d21f2ab6082fad1f913b244d5980816f8147ce9936ef \
--node-name=worker
Note: Only Sample, not to be used as-is.
```
```console
Expected Output

[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm
    kubeadm-config -oyaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file
    "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

13. Try to run the kubectl command on the worker system. It should fail. You do not have the cluster or authentication keys in your local .kube/config file.

```console
root@worker:~#

exit
```
```console
student@worker:~$

kubectl get nodes
```
```console
Expected Output

The connection to the server localhost:8080 was refused - did you specify the right host or port?
```
```console
student@worker:~$

ls -l .kube
```
```
Expected Output

ls: cannot access '.kube': No such file or directory
```