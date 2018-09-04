---
title: ReplicaSets
permalink: /replicasets
icon: layer-group
order: 04
---

Change into the directory for Exercise 4.

```
cd ~/lab/exercise-4/
```

Let's take a look at the `replicaset.yaml` Manifest file.

```
cat replicaset.yaml
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: helloworld-replicaset
  labels:
    app: helloworld
spec:
  replicas: 2
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
        - name: helloworld
          image: rackspacetraining/helloworld:1.0
          ports:
            - name: web
              containerPort: 80
          volumeMounts:
            - name: nfs
              mountPath: "/mnt/images"
      volumes:
        - name: nfs
          nfs:
            server: nfs-server.default.svc.cluster.local
            path: "/exports"
```

We can see the `Kind` attribute is set to `ReplicaSet`, and there are some new attributes, and some that we've seen before. To put the differences in a better context, let's diff the Manifest for one of the Pods we deployed in Exercise 3, and this `replicaset.yaml` Manifest.

```
colordiff -yb ~/lab/exercise-3/static-pod-vol-1.yaml replicaset.yaml
```

<pre>
<span style="color:teal;">apiVersion: v1                                                | apiVersion: apps/v1</span>
<span style="color:teal;">kind: Pod                                                     | kind: ReplicaSet</span>
metadata:                                                       metadata:
<span style="color:teal;">  name: helloworld-static-pod-vol-1                           |   name: helloworld-replicaset</span>
  labels:                                                         labels:
    app: helloworld                                                 app: helloworld
spec:                                                           spec:
<span style="color:green;">                                                              &gt;   replicas: 2</span>
<span style="color:green;">                                                              &gt;   selector:</span>
<span style="color:green;">                                                              &gt;     matchLabels:</span>
<span style="color:green;">                                                              &gt;       app: helloworld</span>
<span style="color:green;">                                                              &gt;   template:</span>
<span style="color:green;">                                                              &gt;     metadata:</span>
<span style="color:green;">                                                              &gt;       labels:</span>
<span style="color:green;">                                                              &gt;         app: helloworld</span>
<span style="color:green;">                                                              &gt;     spec:</span>
  containers:                                                         containers:
    - name: helloworld                                                  - name: helloworld
      image: rackspacetraining/helloworld:1.0                             image: rackspacetraining/helloworld:1.0
      ports:                                                              ports:
      - name: web                                                           - name: web
        containerPort: 80                                                     containerPort: 80
      volumeMounts:                                                       volumeMounts:
        - name: nfs                                                         - name: nfs
          mountPath: &quot;/mnt/images&quot;                                            mountPath: &quot;/mnt/images&quot;
  volumes:                                                            volumes:
    - name: nfs                                                         - name: nfs
      nfs:                                                                nfs:
        server: nfs-server.default.svc.cluster.local                        server: nfs-server.default.svc.cluster.local
        path: &quot;/exports&quot;                                                    path: &quot;/exports&quot;
</pre>

The ReplicaSet Spec consists of a few key attributes. The `Replicas` attribute defines the desired number of Pods that should exist. The `Selector` attribute is how the ReplicaSet determines how many Pods currently exist by matching on the `app:helloworld` Label. And finally, we can see the Pod Spec becomes the `Template` the ReplicaSet uses to deploy new Pods.

So, if the current number of Pods is less or greater than the desired number of Pods, then the ReplicaSet will react accordingly, creating new Pods based on the Template, or deleting any unneeded Pods.

Create your ReplicaSet by running the following `kubectl` command:

```
kubectl apply -f replicaset.yaml
```

```console
replicaset.apps "helloworld-replicaset" created
```

Check on the status of your ReplicaSet by running the following `kubectl` command:

```
kubectl get replicasets -o wide
```

```console
NAME                    DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-replicaset   2         2         2         2m        helloworld   rackspacetraining/helloworld:1.0   app=helloworld
```

We can see that the desired number of Pods is 2, and after a few seconds, you should indeed have 2 that are running. Check on the status of your Pods by running the following `kubectl` command:

```
kubectl get pods -o wide
```

```console
NAME                          READY     STATUS    RESTARTS   AGE       IP           NODE
helloworld-replicaset-955n4   1/1       Running   0          14s       10.244.1.8   lab-kubernetes-0-worker-1
helloworld-replicaset-cnmp5   1/1       Running   0          14s       10.244.2.7   lab-kubernetes-0-worker-0
nfs-server                    1/1       Running   0          43m       10.244.1.3   lab-kubernetes-0-worker-1
```

Back in your browser, refresh the page several times. Notice the new ReplicaSet managed Pods are now being used.

Now let's see how simple it is to scale up our ReplicaSet. By diffing a modified Manifest file for our ReplicaSet, we can see the number of `Replicas` has been changed to `5`.

```
colordiff -y replicaset.yaml replicaset-scale.yaml
```

<pre>
apiVersion: apps/v1                                             apiVersion: apps/v1
kind: ReplicaSet                                                kind: ReplicaSet
metadata:                                                       metadata:
  name: helloworld-replicaset                                     name: helloworld-replicaset
  labels:                                                         labels:
    app: helloworld                                                 app: helloworld
spec:                                                           spec:
<span style="color:teal;">  replicas: 2                                                 |   replicas: 5</span>
  selector:                                                       selector:
    matchLabels:                                                    matchLabels:
      app: helloworld                                                 app: helloworld
  template:                                                       template:
    metadata:                                                       metadata:
      labels:                                                         labels:
        app: helloworld                                                 app: helloworld
    spec:                                                           spec:
      containers:                                                     containers:
        - name: helloworld                                              - name: helloworld
          image: rackspacetraining/helloworld:1.0                         image: rackspacetraining/helloworld:1.0
          ports:                                                          ports:
            - name: web                                                     - name: web
              containerPort: 80                                               containerPort: 80
          volumeMounts:                                                   volumeMounts:
            - name: nfs                                                     - name: nfs
              mountPath: &quot;/mnt/images&quot;                                        mountPath: &quot;/mnt/images&quot;
      volumes:                                                        volumes:
        - name: nfs                                                     - name: nfs
          nfs:                                                            nfs:
            server: nfs-server.default.svc.cluster.local                    server: nfs-server.default.svc.cluster.local
            path: &quot;/exports&quot;                                                path: &quot;/exports&quot;
</pre>

Since we are updating our ReplicaSet object, we just need to `kubectl apply` the updated Manifest file.

```
kubectl apply -f replicaset-scale.yaml
```

```console
replicaset.apps "helloworld-replicaset" configured
```

Check on the status of your ReplicaSet by running the following `kubectl` command:

```
kubectl get replicasets -o wide
```

```console
NAME                    DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-replicaset   5         5         5         3m        helloworld   rackspacetraining/helloworld:1.0   app=helloworld
```

And we can see the number of desired replicas has been updated to 5. Check on the status of your Pods by running the following `kubectl` command:

```
kubectl get pods -o wide
```

```console
NAME                          READY     STATUS    RESTARTS   AGE       IP           NODE
helloworld-replicaset-5lz9d   1/1       Running   0          31s       10.244.2.8   lab-kubernetes-0-worker-0
helloworld-replicaset-955n4   1/1       Running   0          5m        10.244.1.8   lab-kubernetes-0-worker-1
helloworld-replicaset-9vk8h   1/1       Running   0          31s       10.244.1.9   lab-kubernetes-0-worker-1
helloworld-replicaset-cnmp5   1/1       Running   0          5m        10.244.2.7   lab-kubernetes-0-worker-0
helloworld-replicaset-h5k2z   1/1       Running   0          31s       10.244.2.9   lab-kubernetes-0-worker-0
nfs-server                    1/1       Running   0          45m       10.244.1.3   lab-kubernetes-0-worker-1
```

The ReplicaSet created 3 new Pods, bringing the total up to 5. Since since the new Pods had a Label of `app:helloworld`, they were automatically added to the Service.

Back in your browser, refresh the page several times. Notice the new Pods are now being used.

Scaling down is just as easy. If we update our ReplicaSet object using the original Manifest, which has the `Replica` attribute set to 2, then any excess Pods will automatically be deleted.

```
kubectl apply -f replicaset.yaml
```

```console
replicaset.apps "helloworld-replicaset" configured
```

Check on the status of your ReplicaSet by running the following `kubectl` command:

```
kubectl get replicasets -o wide
```

```console
NAME                    DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-replicaset   2         2         2         4m        helloworld   rackspacetraining/helloworld:1.0   app=helloworld
```

We can see that the desired number of Pods is back to 2, and after a few seconds, the excess Pods will have been deleted, and you should only have 2 that are left running. Check on the status of your Pods by running the following `kubectl` command:

```
kubectl get pods -o wide
```

```console
NAME                          READY     STATUS    RESTARTS   AGE       IP           NODE
helloworld-replicaset-955n4   1/1       Running   0          7m        10.244.1.8   lab-kubernetes-0-worker-1
helloworld-replicaset-cnmp5   1/1       Running   0          7m        10.244.2.7   lab-kubernetes-0-worker-0
nfs-server                    1/1       Running   0          47m       10.244.1.3   lab-kubernetes-0-worker-1
```

With ReplicaSets, scaling up and down is quite trivial. But what about rolling out an update to our application? Can we just update the Container Image version, and let the ReplicaSet handle the rest?

Only one way to find out. So let's diff the `replicaset.yaml` and `replicaset-update.yaml` Manifest files.

```
colordiff -y replicaset.yaml replicaset-update.yaml
```

<pre>
apiVersion: apps/v1                                             apiVersion: apps/v1
kind: ReplicaSet                                                kind: ReplicaSet
metadata:                                                       metadata:
  name: helloworld-replicaset                                     name: helloworld-replicaset
  labels:                                                         labels:
    app: helloworld                                                 app: helloworld
spec:                                                           spec:
  replicas: 2                                                     replicas: 2
  selector:                                                       selector:
    matchLabels:                                                    matchLabels:
      app: helloworld                                                 app: helloworld
  template:                                                       template:
    metadata:                                                       metadata:
      labels:                                                         labels:
        app: helloworld                                                 app: helloworld
    spec:                                                           spec:
      containers:                                                     containers:
        - name: helloworld                                              - name: helloworld
<span style="color:teal;">          image: rackspacetraining/helloworld:1.0             |           image: rackspacetraining/helloworld:1.1</span>
          ports:                                                          ports:
            - name: web                                                     - name: web
              containerPort: 80                                               containerPort: 80
          volumeMounts:                                                   volumeMounts:
            - name: nfs                                                     - name: nfs
              mountPath: &quot;/mnt/images&quot;                                        mountPath: &quot;/mnt/images&quot;
      volumes:                                                        volumes:
        - name: nfs                                                     - name: nfs
          nfs:                                                            nfs:
            server: nfs-server.default.svc.cluster.local                    server: nfs-server.default.svc.cluster.local
            path: &quot;/exports&quot;                                                path: &quot;/exports&quot;
</pre>

We can see, the only difference is the Container Image has been updated to `HelloWorld 1.1`. The main difference between these versions of the Container Image relates to the version of PHP being used.

* HelloWorld 1.0 - **PHP 5.6**
* HelloWorld 1.1 - **PHP 7.0**

Let's update our ReplicaSet object one more time, applying the updated Manifest file.

```
kubectl apply -f replicaset-update.yaml
```

```console
replicaset.apps "helloworld-replicaset" configured
```

Check on the status of your ReplicaSet by running the following `kubectl` command:

```
kubectl get replicasets -o wide
```

```console
NAME                    DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-replicaset   2         2         2         19m       helloworld   rackspacetraining/helloworld:1.1   app=helloworld
```

And we can see the ReplicaSet is now using the `HelloWorld 1.1` Container Image.

Back in your browser, refresh the page several times, but notice the old version is still being used.

Let's dig a little deeper, using `kubectl describe` to see what Container Image each Pod is using.

```
kubectl describe pod helloworld | grep -e "^Name:" -e "Image:"
```

```console
Name:           helloworld-replicaset-955n4
    Image:          rackspacetraining/helloworld:1.0
Name:           helloworld-replicaset-cnmp5
    Image:          rackspacetraining/helloworld:1.0
```

And this actually makes sense. If you recall how ReplicaSets work, they count the number of Pods matching a specific Label, and create or delete Pods to match the desired number of Pods to be running. So what happens if we then scale up this ReplicaSet?

```
colordiff -y replicaset-update.yaml replicaset-update-scale.yaml
```

<pre>
apiVersion: apps/v1                                             apiVersion: apps/v1
kind: ReplicaSet                                                kind: ReplicaSet
metadata:                                                       metadata:
  name: helloworld-replicaset                                     name: helloworld-replicaset
  labels:                                                         labels:
    app: helloworld                                                 app: helloworld
spec:                                                           spec:
<span style="color:teal;">  replicas: 2                                                 |   replicas: 5</span>
  selector:                                                       selector:
    matchLabels:                                                    matchLabels:
      app: helloworld                                                 app: helloworld
  template:                                                       template:
    metadata:                                                       metadata:
      labels:                                                         labels:
        app: helloworld                                                 app: helloworld
    spec:                                                           spec:
      containers:                                                     containers:
        - name: helloworld                                              - name: helloworld
          image: rackspacetraining/helloworld:1.1                         image: rackspacetraining/helloworld:1.1
          ports:                                                          ports:
            - name: web                                                     - name: web
              containerPort: 80                                               containerPort: 80
          volumeMounts:                                                   volumeMounts:
            - name: nfs                                                     - name: nfs
              mountPath: &quot;/mnt/images&quot;                                        mountPath: &quot;/mnt/images&quot;
      volumes:                                                        volumes:
        - name: nfs                                                     - name: nfs
          nfs:                                                            nfs:
            server: nfs-server.default.svc.cluster.local                    server: nfs-server.default.svc.cluster.local
            path: &quot;/exports&quot;                                                path: &quot;/exports&quot;
</pre>

Let's update our ReplicaSet object, changing the number of Replicas to `5`.

```
kubectl apply -f replicaset-update-scale.yaml
```

```console
replicaset.apps "helloworld-replicaset" configured
```

Check on the status of your ReplicaSet by running the following `kubectl` command:

```
kubectl get replicasets -o wide
```

```console
NAME                    DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-replicaset   5         5         5         26m       helloworld   rackspacetraining/helloworld:1.1   app=helloworld
```

Back in your browser, refresh the page several times, but notice a mix of the old version and new version being used. When scaling up to 5, the new Pods used the new Container Image, but the old Pods still remain. We can confirm this again, by using `kubectl describe` to see what Container Image each Pod is using.

```
kubectl describe pod helloworld | grep -e "^Name:" -e "Image:"
```

```console
  Name:           helloworld-replicaset-955n4
      Image:          rackspacetraining/helloworld:1.0
  Name:           helloworld-replicaset-9d2qx
      Image:          rackspacetraining/helloworld:1.1
  Name:           helloworld-replicaset-cnmp5
      Image:          rackspacetraining/helloworld:1.0
  Name:           helloworld-replicaset-mf8fd
      Image:          rackspacetraining/helloworld:1.1
  Name:           helloworld-replicaset-t7v2m
      Image:          rackspacetraining/helloworld:1.1
```

So clearly, ReplicaSets are great for scaling, but not so great for rolling out updates. Let's clean up this mess, and delete the ReplicaSet.

```
kubectl delete -f replicaset.yaml
```

```console
replicaset.apps "helloworld-replicaset" deleted
```
