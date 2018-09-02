---
title: Getting Started
permalink: /getting-started
icon: terminal
order: 00
---

## Accessing Your Lab Environment

This workshop is designed to be completed entirely from the command line. To access your lab environment, you will need an SSH client. Linux and macOS systems should have an SSH client already installed, accessible from your terminal. Windows systems might need to download an SSH client, such as PuTTY, if one is not already installed.

### Linux and macOS
Initate an SSH connection to your Master Node, using the root user and password provided for your lab environment.

```
ssh root@<your-master-node-ip-address>
```

```console
The authenticity of host '23.253.111.163 (23.253.111.163)' can't be established.
ECDSA key fingerprint is SHA256:V9uiWfD7AiO3OzmYqbbu2g/pmrc9FLzgxGOhInZjTYg.
Are you sure you want to continue connecting (yes/no)?
```

Type `yes`, and press enter to continue.

```
yes
```

```console
Warning: Permanently added '23.253.111.163' (ECDSA) to the list of known hosts.
root@23.253.111.163's password: 
```

Type in your password, then press `enter` to continue.

```console
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-119-generic x86_64)

root@summit-student-0-master:~#
```

### Windows PuTTY

Windows systems using PuTTY, enter your Master Node IP Address. Ensure the Connection Type is set to SSH, and the Port is set to 22, then click `Open`. Follow the prompts to accept the connection. Login using the root user and password provided for your lab environment.


## Kubernetes Nodes

Kubernetes Nodes are systems that are part of a Kubernetes Cluster. There are two types of Nodes in your Cluster today: Master Node & Worker Node. The Master Node is where many Kubernetes Components run that make the Cluster work. The Worker Node is where user workloads run.

## Kubeclt

The command line tool used to interact with Kubernetes is known as `kubectl`. It looks for your Kubernetes credentials in `~/.kube/config` by default, which have already been setup for you.

You can see your Nodes by running the following `kubectl` command:

```
kubectl get nodes
```

```console
NAME                        STATUS    ROLES     AGE       VERSION
summit-student-0-master     Ready     master    4m        v1.10.2
summit-student-0-worker-0   Ready     <none>    3m        v1.10.2
summit-student-0-worker-1   Ready     <none>    3m        v1.10.2
```

And you can see, there are 3 Nodes that are part of your Cluster. One Master Node, and two Worker Nodes. Additional information about the Nodes can be dispalyed by modifying the command as such:

```
kubectl get nodes -o wide
```

```console
NAME                        STATUS    ROLES     AGE       VERSION   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
summit-student-0-master     Ready     master    4m        v1.10.2   <none>        Ubuntu 16.04.4 LTS   4.4.0-119-generic   docker://1.13.1
summit-student-0-worker-0   Ready     <none>    3m        v1.10.2   <none>        Ubuntu 16.04.4 LTS   4.4.0-119-generic   docker://1.13.1
summit-student-0-worker-1   Ready     <none>    3m        v1.10.2   <none>        Ubuntu 16.04.4 LTS   4.4.0-119-generic   docker://1.13.1
```

As the command implies, the output is wider, giving room for some additional bits of information. But you can get even more information about one of your nodes by running the following command:

```
kubectl describe node summit-student-0-master
```

There is too much to display here, so this is just a small sample of the output.

```console
Name:               summit-student-0-master
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=summit-student-0-master
                    node-role.kubernetes.io/master=
Annotations:        flannel.alpha.coreos.com/backend-data={"VtepMAC":"d2:66:06:29:2e:7a"}
                    flannel.alpha.coreos.com/backend-type=vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager=true
                    flannel.alpha.coreos.com/public-ip=23.253.111.163
                    node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
CreationTimestamp:  Mon, 14 May 2018 18:10:14 +0000
Taints:             node-role.kubernetes.io/master:NoSchedule
[...snip...]
```

These same commands, `kubectl get` and `kubectl describe`, can be used on other types of Objects within Kubernetes. You will encounter their usage in future exercises.

But there is one more command that helps to get a birds eye view of everthing that's in the cluster, and might save you a few keystrokes.

```
kubectl get all -o wide
```

```console
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE       SELECTOR
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   22h       <none>
```

And as you can see, there is one Object already present, a Service Object. We will encounter Services in a future exercise.
