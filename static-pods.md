---
title: Static Pods
permalink: /static-pods
icon: cube
order: 01
---

Change into the directory for Exercise 1.

```
cd ~/lab/exercise-1/
```

In here are some files we'll be using for this exercise. These files are known as Manifest files, and they describe one or more resources to deploy within Kubernetes. Let's take a look at our first Manifest file.

```
cat static-pod-1.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: helloworld-static-pod-1
  labels:
    app: helloworld
spec:
  containers:
  - name: helloworld
    image: rackspacetraining/helloworld:1.0
    ports:
    - name: web
      containerPort: 80
```

The first thing to note is the `Kind` attribute, which declares the kind of object this is. Here, we are deploying a `Pod`, which is the smallest deployable unit in Kubernetes.

The `Metadata` section contains two identifying attributes of the Pod, a `Name`, and one or more `Labels`. The `Name` attribute is an identifier for this particular Pod, and must be unique. `Labels` are an adhoc way of associating related or similar Pods using one or more arbitrary Key / Value pairs. For example, our Pod here has a Label of `app:helloworld`. We will make use of this `Label` when we start scaling out to multiple Pods, and need to refer to the entire group of them.

The `Spec` section describes the resources that are a part of this Pod. We only have one type of resource in this Pod, a `Container`, but we will encounter others later on.

Our single `Container` is composed of a few of its own attributes. It has a `Name` for identifying it within the Pod, an `Image` from which it is deployed from, and a set of `Ports` the Container exposes.

Create your first Pod by running the following `kubectl` command:

```
kubectl create -f static-pod-1.yaml
```

```console
pod "helloworld-static-pod-1" created
```

Check on the status of your Pod by running the following `kubectl` command:

```
kubectl get pods
```

```console
NAME                      READY     STATUS    RESTARTS   AGE
helloworld-static-pod-1   1/1       Running   0          42s
```

It might take a few seconds for the Pod to start for the first time, as the Image will need to be pulled down over the network. The Pod will be ready to use when the Status shows as `Running`.

As mentioned in the Getting Started section, we can get a bit more information about our Pod by adding `-o wide` to the command.

```
kubectl get pods -o wide
```

```console
NAME                      READY     STATUS    RESTARTS   AGE       IP           NODE
helloworld-static-pod-1   1/1       Running   0          1m        10.244.1.2   summit-student-0-worker-1
```

And even more information by running `kubectl describe` on the Pod.

```
kubectl describe pod helloworld-static-pod-1
```

```console
Name:         helloworld-static-pod-1
Namespace:    default
Node:         summit-student-0-worker-1/23.253.111.235
Start Time:   Mon, 14 May 2018 18:17:11 +0000
Labels:       app=helloworld
Annotations:  <none>
Status:       Running
IP:           10.244.1.2
[...snip...]
```

The sample output above had been trimmed, as there is too much to display it all here.

Using either the `-o wide` or `kubectl describe` commands, we can see our Pod has an IP address assigned to it. Let's get that IP address, and save it to an environment variable, so we can refer to it later.

```
STATIC_POD_1_IP=`kubectl get pods -o wide | awk '/helloworld-static-pod-1/ { print $6 }'`
echo $STATIC_POD_1_IP
```

```console
10.244.1.2
```

Since our HelloWorld application is a web application, we should be able to acces it over HTTP via the Pod IP address. We will be using HTTPie for making our HTTP requests from the command line, since it has a much cleaner syntax than the more commonly used `curl`.

```
http --body $STATIC_POD_1_IP | head -n 25
```

```html
<!DOCTYPE html>
<!--

                                                                                                              
        ,--,                                                                                                    
      ,--.'|            ,--,    ,--,                               .---.                     ,--,               
   ,--,  | :          ,--.'|  ,--.'|                              /. ./|                   ,--.'|         ,---, 
,---.'|  : '          |  | :  |  | :     ,---.                .--'.  ' ;   ,---.    __  ,-.|  | :       ,---.'| 
|   | : _' |          :  : '  :  : '    '   ,'\              /__./ \ : |  '   ,'\ ,' ,'/ /|:  : '       |   | : 
:   : |.'  |   ,---.  |  ' |  |  ' |   /   /   |         .--'.  '   \' . /   /   |'  | |' ||  ' |       |   | | 
|   ' '  ; :  /     \ '  | |  '  | |  .   ; ,. :        /___/ \ |    ' '.   ; ,. :|  |   ,''  | |     ,--.__| | 
'   |  .'. | /    /  ||  | :  |  | :  '   | |: :        ;   \  \;      :'   | |: :'  :  /  |  | :    /   ,'   | 
|   | :  | '.    ' / |'  : |__'  : |__'   | .; :         \   ;  `      |'   | .; :|  | '   '  : |__ .   '  /  | 
'   : |  : ;'   ;   /||  | '.'|  | '.'|   :    |          .   \    .\  ;|   :    |;  : |   |  | '.'|'   ; |:  | 
|   | '  ,/ '   |  / |;  :    ;  :    ;\   \  /            \   \   ' \ | \   \  / |  , ;   ;  :    ;|   | '/  ' 
;   : ;--'  |   :    ||  ,   /|  ,   /  `----'              :   '  |--"   `----'   ---'    |  ,   / |   :    :| 
|   ,/       \   \  /  ---`-'  ---`-'                        \   \ ;                        ---`-'   \   \  /   
'---'         `----'                                          '---"                                   `----'    
                                                                                                              

-->
<html lang="en">
	<head>
		<meta charset="utf-8">
		<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

And it appears our HelloWorld web application is up and running.

Now, while everything may seems OK, we only have a single Pod of our application running. This isn't exactly ideal, as it leaves us vulnerable in the event of a Pod or Worker Node failure.

So, let's deploy another Pod of our application. You can see the only difference between `static-pod-1.yaml` and `static-pod-2.yaml` is the name of the Pod, because as we already discussed, that must be unique.

```
colordiff -y static-pod-1.yaml static-pod-2.yaml
```

<pre>
apiVersion: v1                                                  apiVersion: v1
kind: Pod                                                       kind: Pod
metadata:                                                       metadata:
<span style="color:teal;">  name: helloworld-static-pod-1                               |   name: helloworld-static-pod-2</span>
  labels:                                                         labels:
    app: helloworld                                                 app: helloworld
spec:                                                           spec:
  containers:                                                     containers:
  - name: helloworld                                              - name: helloworld
    image: rackspacetraining/helloworld:1.0                         image: rackspacetraining/helloworld:1.0
    ports:                                                          ports:
    - name: web                                                     - name: web
      containerPort: 80                                               containerPort: 80
</pre>

Create your second Pod by running the following `kubectl` command:

```
kubectl create -f static-pod-2.yaml
```

```console
pod "helloworld-static-pod-2" created
```

Check on the status of your Pods by running the following `kubectl` command:

```
kubectl get pods -o wide
```

```console
NAME                      READY     STATUS    RESTARTS   AGE       IP           NODE
helloworld-static-pod-1   1/1       Running   0          12m       10.244.1.2   summit-student-0-worker-1
helloworld-static-pod-2   1/1       Running   0          2m        10.244.2.2   summit-student-0-worker-0
```

Again, it might take a few seconds for the Pod to start for the first time, as Kubernetes probably scheduled it on a different Worker Node, and the Image isn't cached on that Worker Node yet. The Pod will be ready to use when the Status shows as `Running`.

Let's get the IP address of the second Pod, and save it to an environment variable.

```
STATIC_POD_2_IP=`kubectl get pods -o wide | awk '/helloworld-static-pod-2/ { print $6 }'`
echo $STATIC_POD_2_IP
```

```console
10.244.2.2
```

And just as before, confirm we can access the second Pod over HTTP.

```
http --body $STATIC_POD_2_IP | head -n 25
```

```html
<!DOCTYPE html>
<!--

                                                                                                              
        ,--,                                                                                                    
      ,--.'|            ,--,    ,--,                               .---.                     ,--,               
   ,--,  | :          ,--.'|  ,--.'|                              /. ./|                   ,--.'|         ,---, 
,---.'|  : '          |  | :  |  | :     ,---.                .--'.  ' ;   ,---.    __  ,-.|  | :       ,---.'| 
|   | : _' |          :  : '  :  : '    '   ,'\              /__./ \ : |  '   ,'\ ,' ,'/ /|:  : '       |   | : 
:   : |.'  |   ,---.  |  ' |  |  ' |   /   /   |         .--'.  '   \' . /   /   |'  | |' ||  ' |       |   | | 
|   ' '  ; :  /     \ '  | |  '  | |  .   ; ,. :        /___/ \ |    ' '.   ; ,. :|  |   ,''  | |     ,--.__| | 
'   |  .'. | /    /  ||  | :  |  | :  '   | |: :        ;   \  \;      :'   | |: :'  :  /  |  | :    /   ,'   | 
|   | :  | '.    ' / |'  : |__'  : |__'   | .; :         \   ;  `      |'   | .; :|  | '   '  : |__ .   '  /  | 
'   : |  : ;'   ;   /||  | '.'|  | '.'|   :    |          .   \    .\  ;|   :    |;  : |   |  | '.'|'   ; |:  | 
|   | '  ,/ '   |  / |;  :    ;  :    ;\   \  /            \   \   ' \ | \   \  / |  , ;   ;  :    ;|   | '/  ' 
;   : ;--'  |   :    ||  ,   /|  ,   /  `----'              :   '  |--"   `----'   ---'    |  ,   / |   :    :| 
|   ,/       \   \  /  ---`-'  ---`-'                        \   \ ;                        ---`-'   \   \  /   
'---'         `----'                                          '---"                                   `----'    
                                                                                                              

-->
<html lang="en">
	<head>
		<meta charset="utf-8">
		<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

And it works...great job!

But now we have a new problem. How do we distribute requests between these two Pods?
