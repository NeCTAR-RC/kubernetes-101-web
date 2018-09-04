---
title: Volumes
permalink: /volumes
icon: hdd
order: 03
---

Change into the directory for Exercise 3.

```
cd ~/lab/exercise-3/
```

When we uploaded a background image to our web application, that file was stored within the Container of one of the two Pods. This presents two problems. First, Containers are considered stateless. If a Pod needed to be reschedule because of a failure, all the data saved within the Container would be lost. So we need a way to persist certain data within a Container. Second, our web application wasn't able to consistently serve requests for the background image, because not every Container actually contained the image.

To solve these issues, we can utilize a `Volume`, which provides persistent mounts within a Container, and because it needs to be shared storage, the type of Volume we've chosen to use is an `NFS Volume`. We can quickly spin up an NFS server using the provided `nfs-server.yaml` Manifest file, the details of which are out of scope for this workshop.

Create your NFS Server by running the following `kubectl` command:

```
kubectl create -f nfs-server.yaml
```

```console
service "nfs-server" created
pod "nfs-server" created
```

Check on the status of your NFS Server Pod by running the following `kubectl` command:

```
kubectl get pod nfs-server -o wide
```

```console
NAME         READY     STATUS    RESTARTS   AGE       IP           NODE
nfs-server   1/1       Running   0          4m        10.244.1.3   summit-student-0-worker-1
```

The NFS Server also has a Service provisioned. We've already seen the benefit of using a Service, which can provide load balancing capabilites, but Services also provides a well known DNS name for the Service they expose. We will see this in use shortly.

```
kubectl get service nfs-server -o wide
```

```console
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE       SELECTOR
nfs-server   ClusterIP   10.107.233.255   <none>        2049/TCP,20048/TCP,111/TCP   4m        role=nfs-server
```

Comparing the Manifest file we previously deployed, to an modified version that includes a `Volume`, we can see some minor differences.

```
colordiff -y ~/lab/exercise-1/static-pod-1.yaml static-pod-vol-1.yaml
```

```console
apiVersion: v1                                                  apiVersion: v1
kind: Pod                                                       kind: Pod
metadata:                                                       metadata:
  name: helloworld-static-pod-1                               |   name: helloworld-static-pod-vol-1
  labels:                                                         labels:
    app: helloworld                                                 app: helloworld
spec:                                                           spec:
  containers:                                                     containers:
    - name: helloworld                                              - name: helloworld
      image: rackspacetraining/helloworld:1.0                         image: rackspacetraining/helloworld:1.0
      ports:                                                          ports:
        - name: web                                                     - name: web
          containerPort: 80                                               containerPort: 80
                                                              >       volumeMounts:
                                                              >         - name: nfs
                                                              >           mountPath: "/mnt/images"
                                                              >   volumes:
                                                              >     - name: nfs
                                                              >       nfs:
                                                              >         server: nfs-server.default.svc.cluster.local
                                                              >         path: "/exports"
```

Under the `Spec` section, a new `Volumes` attribute has been added. This is where we define any Volumes we need within the Pod. And you can see the type we are creating is an `NFS Volume`. Then inside the `Containers` section, a `VolumeMounts` attribute controls which Volumes gets mounted where within the Container.

You may have noticed our NFS Volume has been configured to connect to the NFS Server `nfs-server.default.svc.cluster.local`. This is that well known DNS name were spoke of earler. We can confirm that, by seeing where the DNS resolves to.

```
nslookup nfs-server.default.svc.cluster.local
```

```console
Server:		10.96.0.10
Address:	10.96.0.10#53

Non-authoritative answer:
Name:	nfs-server.default.svc.cluster.local
Address: 10.107.233.255
```

And if you refer back to the Cluster IP of your NFS Server Service, you will see it does indeed point to it.

So with that out of the way, lets deploy our new Pods utilizing an NFS Volume. Create your new Pods by running the following `kubectl` command

```
kubectl create -f static-pod-vol-1.yaml -f static-pod-vol-2.yaml
```

```console
pod "helloworld-static-pod-vol-1" created
pod "helloworld-static-pod-vol-2" created
```

Check on the status of your Pods by running the following `kubectl` command:

```
kubectl get pods -o wide
```

```console
NAME                          READY     STATUS    RESTARTS   AGE       IP           NODE
helloworld-static-pod-vol-1   1/1       Running   0          42s       10.244.2.4   summit-student-0-worker-0
helloworld-static-pod-vol-2   1/1       Running   0          28s       10.244.1.4   summit-student-0-worker-1
nfs-server                    1/1       Running   0          8m        10.244.1.3   summit-student-0-worker-1
```

Back in your browser, refresh the page several times. Notice the new Pods are now being used. The Service we previously deployed automatically picked up these new Pods, because they included the `app:helloworld` Label.

Now click "Hello World", and choose an image to upload as the background image (gif, jpg, or png). Refresh the page several times, and notice that the image always loads, no matter which Pod services the request.
