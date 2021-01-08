# k0s Ansible Playbook

Create a Kubernetes Cluster using Ansible and k0s (and multipass).

This script is largely based on the extensive and outstanding work of the contributors of [k3s-ansible](https://github.com/k3s-io/k3s-ansible) and, of course, [kubespray](https://github.com/kubernetes-sigs/kubespray). I put it together to learn about Ansible and the new single binary and vanilla upstream Kubernetes distro [k0s](https://github.com/k0sproject/k0s).

For the quick creation of virtual machines, I have added a script that provisions a bunch of nodes via multipass

## How to run this.

#### Create 5 instances with multipass and import your ssh public key with cloud-init

```ShellSession
$ ./tools/create_instances.sh
Create cloud-init to import ssh key...
[1/5] Creating instance k0s-1 with multipass...
Launched: k0s-1
[2/5] Creating instance k0s-2 with multipass...
Launched: k0s-2
[3/5] Creating instance k0s-3 with multipass...
Launched: k0s-3
[4/5] Creating instance k0s-4 with multipass...
Launched: k0s-4
[5/5] Creating instance k0s-5 with multipass...
Launched: k0s-5
Name                    State             IPv4             Image
k0s-1                   Running           192.168.64.32    Ubuntu 20.04 LTS
k0s-2                   Running           192.168.64.33    Ubuntu 20.04 LTS
k0s-3                   Running           192.168.64.56    Ubuntu 20.04 LTS
k0s-4                   Running           192.168.64.57    Ubuntu 20.04 LTS
k0s-5                   Running           192.168.64.58    Ubuntu 20.04 LTS
```


#### Create your cluster:

```ShellSession
$ ansible-playbook playbooks/deploy_k0s.yml -i inventory/inventory.yml
```

The inventory determines which nodes are control nodes and which are worker nodes via groups. If you want to have more than one control node you have to adjust the inventory to your preference.
For example: 
```
---
all:
  children:
    k0s:
      hosts:
        k0s-[1:5]
    controller:
      hosts:
        k0s-1      
    worker:
      hosts:
        k0s-[2:5]
```
Based on the group membership, control nodes then execute specific tasks that should not be executed on the workers.


#### Connect to your new Kubernetes cluster. The config is ready to use in the `inventory/k0s-1` directory:

```ShellSession
$ export KUBECONFIG=/path/to/kubeconfig/k0s-kubeconfig.yml
$ kubectl get nodes -o wide
NAME    STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
k0s-2   Ready    <none>   2m42s   v1.19.4   192.168.64.33   <none>        Ubuntu 20.04.1 LTS   5.4.0-54-generic   containerd://1.4.3
k0s-3   Ready    <none>   2m42s   v1.19.4   192.168.64.56   <none>        Ubuntu 20.04.1 LTS   5.4.0-54-generic   containerd://1.4.3
k0s-4   Ready    <none>   2m42s   v1.19.4   192.168.64.57   <none>        Ubuntu 20.04.1 LTS   5.4.0-54-generic   containerd://1.4.3
k0s-5   Ready    <none>   2m42s   v1.19.4   192.168.64.58   <none>        Ubuntu 20.04.1 LTS   5.4.0-54-generic   containerd://1.4.3
$ kubectl run hello-k0s --image=quay.io/prometheus/busybox --rm -it --restart=Never --command -- sh -c "echo hello k0s"
hello k0s
pod "hello-k0s" deleted
```

## Todo

- Support multiple control plane nodes
- Create sample a sample
