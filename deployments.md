---
title: Deployments
permalink: /deployments
icon: cog
order: 05
---

Change into the directory for Exercise 5.

```
cd ~/lab/exercise-5/
```

Let's compare the `replicaset.yaml` Manifest file from Exercise 4 to the `deployment.yaml` Manifest file.

```
colordiff -yb ~/lab/exercise-4/replicaset.yaml deployment.yaml
```

```yaml
apiVersion: apps/v1                                             apiVersion: apps/v1
kind: ReplicaSet                                              | kind: Deployment
metadata:                                                       metadata:
  name: helloworld-replicaset                                 |   name: helloworld-deployment
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
          image: rackspacetraining/helloworld:1.0                         image: rackspacetraining/helloworld:1.0
          ports:                                                          ports:
            - name: web                                                     - name: web
              containerPort: 80                                               containerPort: 80
          volumeMounts:                                                   volumeMounts:
            - name: nfs                                                     - name: nfs
              mountPath: "/mnt/images"                                        mountPath: "/mnt/images"
      volumes:                                                        volumes:
        - name: nfs                                                     - name: nfs
          nfs:                                                            nfs:
            server: nfs-server.default.svc.cluster.local                    server: nfs-server.default.svc.cluster.local
            path: "/exports"                                                path: "/exports"
```

As you can see, the only real difference is the `Kind` attribute being set to `Deployment`.

Create your Deployment by running the following `kubectl` command. Make note of the `--record` option being used. We'll get back to this a little later.

```
kubectl apply -f deployment.yaml --record
```

```console
deployment.apps "helloworld-deployment" created
```

Check on the status of your Deployment by running the following `kubectl` command:

```
kubectl get deployment -o wide
```

```console
NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-deployment   2         2         2            2           38s       helloworld   rackspacetraining/helloworld:1.0   app=helloworld
```

We can see that the desired number of Pods is 2, and after a few seconds, you should indeed have 2 that are running. Now, this is where Deployments differ from ReplicaSets. Deployment don't directly manage any Pods. Instead, they manage a ReplicaSet, which then manages the Pods.

Check on the status of your ReplicaSets by running the following `kubectl` command:

```
kubectl get replicasets -o wide
```

```console
NAME                               DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-deployment-576c94d5f7   2         2         2         1m        helloworld   rackspacetraining/helloworld:1.0   app=helloworld,pod-template-hash=1327508193
```

And check on the status of your Pods by running the following `kubectl` command:

```
kubectl get pods -o wide
```

```console
NAME                                     READY     STATUS    RESTARTS   AGE       IP            NODE
helloworld-deployment-576c94d5f7-b5sdx   1/1       Running   0          1m        10.244.2.11   summit-student-0-worker-0
helloworld-deployment-576c94d5f7-r4fwf   1/1       Running   0          1m        10.244.1.12   summit-student-0-worker-1
nfs-server                               1/1       Running   0          1h        10.244.1.3    summit-student-0-worker-1
```

Let's try scaling up our Deployment to 10 replicas.

```
colordiff -y deployment.yaml deployment-scale.yaml
```

```yaml
apiVersion: apps/v1                                             apiVersion: apps/v1
kind: Deployment                                                kind: Deployment
metadata:                                                       metadata:
  name: helloworld-deployment                                     name: helloworld-deployment
  labels:                                                         labels:
    app: helloworld                                                 app: helloworld
spec:                                                           spec:
  replicas: 2                                                 |   replicas: 10
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
              mountPath: "/mnt/images"                                        mountPath: "/mnt/images"
      volumes:                                                        volumes:
        - name: nfs                                                     - name: nfs
          nfs:                                                            nfs:
            server: nfs-server.default.svc.cluster.local                    server: nfs-server.default.svc.cluster.local
            path: "/exports"                                                path: "/exports"
```

Since we are updating our Deployment object, we just need to `kubectl apply` the updated Manifest file. Again, notice we are using the `--record option`. More on this in a bit.

```
kubectl apply -f deployment-scale.yaml --record
```

```console
deployment.apps "helloworld-deployment" configured
```

Check on the status of your Deployment by running the following kubectl command:

```
kubectl get deployment -o wide
```

```console
NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-deployment   10        10        10           10          4m        helloworld   rackspacetraining/helloworld:1.0   app=helloworld
```

The Deployment has successfully been scaled up to 10 replicas. Digging in, we can check on the underlying ReplicaSet by running the following `kubectl` command

```
kubectl get replicasets -o wide
```

```console
NAME                               DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-deployment-576c94d5f7   10        10        10        5m        helloworld   rackspacetraining/helloworld:1.0   app=helloworld,pod-template-hash=1327508193
```

And it also shows the desired number of replicas is now 10. And just to confirm, check on the status of your Pods by running the following `kubectl` command:

```
kubectl get pods -o wide
```

```console
NAME                                     READY     STATUS    RESTARTS   AGE       IP            NODE
helloworld-deployment-576c94d5f7-6m7lb   1/1       Running   0          1m        10.244.1.15   summit-student-0-worker-1
helloworld-deployment-576c94d5f7-99x26   1/1       Running   0          1m        10.244.1.16   summit-student-0-worker-1
helloworld-deployment-576c94d5f7-b5sdx   1/1       Running   0          5m        10.244.2.11   summit-student-0-worker-0
helloworld-deployment-576c94d5f7-fkz4r   1/1       Running   0          1m        10.244.2.13   summit-student-0-worker-0
helloworld-deployment-576c94d5f7-l6w9j   1/1       Running   0          1m        10.244.1.14   summit-student-0-worker-1
helloworld-deployment-576c94d5f7-lx2sn   1/1       Running   0          1m        10.244.2.14   summit-student-0-worker-0
helloworld-deployment-576c94d5f7-nj2sc   1/1       Running   0          1m        10.244.2.15   summit-student-0-worker-0
helloworld-deployment-576c94d5f7-qqvnc   1/1       Running   0          1m        10.244.1.13   summit-student-0-worker-1
helloworld-deployment-576c94d5f7-r4fwf   1/1       Running   0          5m        10.244.1.12   summit-student-0-worker-1
helloworld-deployment-576c94d5f7-zd5c5   1/1       Running   0          1m        10.244.2.12   summit-student-0-worker-0
nfs-server                               1/1       Running   0          1h        10.244.1.3    summit-student-0-worker-1
```

And there are all 10 Pods.

At this point, there doesn't appear to be any difference between using a ReplicaSet, and using a Deployment. But if you recall, ReplicaSets were great at scaling up and down, but not so great at rolling out updates.

So let's try rolling out an update using our Deployment. Diffing a modified Manifest file for our Deployment, we can see the Container Image has been updated to `HelloWorld 1.1`.

```
colordiff -y deployment-scale.yaml deployment-update.yaml
```

```yaml
apiVersion: apps/v1                                             apiVersion: apps/v1
kind: Deployment                                                kind: Deployment
metadata:                                                       metadata:
  name: helloworld-deployment                                     name: helloworld-deployment
  labels:                                                         labels:
    app: helloworld                                                 app: helloworld
spec:                                                           spec:
  replicas: 10                                                    replicas: 10
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
          image: rackspacetraining/helloworld:1.0             |           image: rackspacetraining/helloworld:1.1
          ports:                                                          ports:
            - name: web                                                     - name: web
              containerPort: 80                                               containerPort: 80
          volumeMounts:                                                   volumeMounts:
            - name: nfs                                                     - name: nfs
              mountPath: "/mnt/images"                                        mountPath: "/mnt/images"
      volumes:                                                        volumes:
        - name: nfs                                                     - name: nfs
          nfs:                                                            nfs:
            server: nfs-server.default.svc.cluster.local                    server: nfs-server.default.svc.cluster.local
            path: "/exports"                                                path: "/exports"
```

Since we are updating our Deployment object, we just need to `kubectl apply` the updated Manifest file.

```
kubectl apply -f deployment-update.yaml --record
```

```console
deployment.apps "helloworld-deployment" configured
```

Check on the status of your Deployment by running the following `kubectl` command:

```
kubectl get deployment -o wide
```

```console
NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-deployment   10        10        10           10          8m        helloworld   rackspacetraining/helloworld:1.1   app=helloworld
```

And we can see the Container Image has been updated to `HelloWorld 1.1`. Now, this is where things fell apart when we used a ReplicaSet.

Check on the status of your ReplicaSet by running the following `kubectl` command:

```
kubectl get replicasets -o wide
```

```console
NAME                               DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-deployment-576c94d5f7   0         0         0         8m        helloworld   rackspacetraining/helloworld:1.0   app=helloworld,pod-template-hash=1327508193
helloworld-deployment-5f48d668fb   10        10        10        30s       helloworld   rackspacetraining/helloworld:1.1   app=helloworld,pod-template-hash=1904822496
```

Well, this is interesting...

When we updated the Container Image for the Deployment, instead of pushing that change down to the ReplicaSet, a new ReplicaSet was created using the new Container Image. Then the old ReplicaSet was scaled down, while the new ReplicaSet was scaled up, giving us a rolling update.

Check on the status of your Pods by running the following `kubectl` command:

```
kubectl get pods -o wide
```

```console
NAME                                     READY     STATUS    RESTARTS   AGE       IP            NODE
helloworld-deployment-5f48d668fb-5z7zt   1/1       Running   0          6m        10.244.2.19   summit-student-0-worker-0
helloworld-deployment-5f48d668fb-kp5cs   1/1       Running   0          6m        10.244.2.17   summit-student-0-worker-0
helloworld-deployment-5f48d668fb-lld4r   1/1       Running   0          6m        10.244.1.20   summit-student-0-worker-1
helloworld-deployment-5f48d668fb-lsjdd   1/1       Running   0          6m        10.244.2.16   summit-student-0-worker-0
helloworld-deployment-5f48d668fb-m4ks2   1/1       Running   0          6m        10.244.1.21   summit-student-0-worker-1
helloworld-deployment-5f48d668fb-sq8xh   1/1       Running   0          6m        10.244.1.17   summit-student-0-worker-1
helloworld-deployment-5f48d668fb-wxzj2   1/1       Running   0          6m        10.244.2.20   summit-student-0-worker-0
helloworld-deployment-5f48d668fb-x6b9k   1/1       Running   0          6m        10.244.1.19   summit-student-0-worker-1
helloworld-deployment-5f48d668fb-xfq6l   1/1       Running   0          6m        10.244.1.18   summit-student-0-worker-1
helloworld-deployment-5f48d668fb-xpvg7   1/1       Running   0          6m        10.244.2.18   summit-student-0-worker-0
nfs-server                               1/1       Running   0          1h        10.244.1.3    summit-student-0-worker-1
```

Back in your browser, refresh the page several times, and notice the new Container Image being used.

Now that we've rolled out an update, what if something had gone wrong? Could we rollback the change?

The `kubectl rollout history` command shows a history of changes that have been applied to a Deployment. This is where the `--record` option comes in handy, as it ensures the `CHANGE-CAUSE` is populated with the actual command that was run.

```
kubectl rollout history deployment helloworld-deployment
```

```console
deployments "helloworld-deployment"
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=deployment-scale.yaml --record=true
2         kubectl apply --filename=deployment-update.yaml --record=true
```

To rollback your Deployment, run the following `kubectl` command.

```
kubectl rollout undo deployment helloworld-deployment
```

```console
deployment.apps "helloworld-deployment"
```

Confirm the deployment is now using the old Container Image.

```kubectl get deployment -o wide```

```
NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-deployment   10        10        10           10          4m        helloworld   rackspacetraining/helloworld:1.0   app=helloworld
```

Confirm the old ReplicaSet was scaled back up, and the new ReplicaSet was scaled down.

```
kubectl get replicasets -o wide
```

```console
NAME                               DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES                             SELECTOR
helloworld-deployment-576c94d5f7   10        10        10        4m        helloworld   rackspacetraining/helloworld:1.0   app=helloworld,pod-template-hash=1327508193
helloworld-deployment-5f48d668fb   0         0         0         3m        helloworld   rackspacetraining/helloworld:1.1   app=helloworld,pod-template-hash=1904822496
```

And confirm the Pods are from the old ReplicaSet.

```
kubectl get pods -o wide
```

```console
NAME                                     READY     STATUS    RESTARTS   AGE       IP            NODE
helloworld-deployment-576c94d5f7-295z5   1/1       Running   0          2m        10.244.2.34   summit-student-0-worker-0
helloworld-deployment-576c94d5f7-2jwcw   1/1       Running   0          2m        10.244.1.39   summit-student-0-worker-1
helloworld-deployment-576c94d5f7-45t8l   1/1       Running   0          2m        10.244.1.38   summit-student-0-worker-1
helloworld-deployment-576c94d5f7-4fzbp   1/1       Running   0          2m        10.244.1.34   summit-student-0-worker-1
helloworld-deployment-576c94d5f7-52brn   1/1       Running   0          2m        10.244.2.33   summit-student-0-worker-0
helloworld-deployment-576c94d5f7-cg5qv   1/1       Running   0          2m        10.244.2.35   summit-student-0-worker-0
helloworld-deployment-576c94d5f7-fvpxw   1/1       Running   0          2m        10.244.2.36   summit-student-0-worker-0
helloworld-deployment-576c94d5f7-r5fp9   1/1       Running   0          2m        10.244.1.36   summit-student-0-worker-1
helloworld-deployment-576c94d5f7-xvqc9   1/1       Running   0          2m        10.244.1.35   summit-student-0-worker-1
helloworld-deployment-576c94d5f7-zlx8h   1/1       Running   0          2m        10.244.1.37   summit-student-0-worker-1
nfs-server                               1/1       Running   0          1h        10.244.1.3    summit-student-0-worker-1
```

Back in your browser, refresh the page several times, and confirm the rollback was successful.
