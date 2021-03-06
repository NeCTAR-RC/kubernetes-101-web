---
title: Services
permalink: /services
icon: cloud
order: 02
---

Change into the directory for Exercise 2.

```
cd ~/lab/exercise-2/
```

Let's take a look at the `service.yaml` Manifest file.

```
cat service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloworld-service
spec:
  ports:
    - port: 80
  selector:
    app: helloworld
```

You'll notice the `Kind` attribute is set to `Service`. A Service exposes a set of Pods under a single IP address, load balancing between them. The Service also has a uniquie `Name` defined as part of the `Metadata` section. The `Spec` section defines the characterists of the Service, including the `Ports` to be exposed, and a `Selector`, which determies which Pods make up the pool.

Looking at the Pods from Exercise 1, they were deployed with a label of `app:helloworld`. The Service's `Selector` looks for Pods with a matching Label, and adds them to the pool.

```
grep metadata -A 3 ~/lab/exercise-1/static-pod-*.yaml
```

<pre>
<span style="color:pink;">/home/ubuntu/lab/exercise-1/static-pod-1.yaml</span><span style="color:teal;">:</span><span style="color:red;">metadata</span>:
<span style="color:pink;">/home/ubuntu/lab/exercise-1/static-pod-1.yaml</span><span style="color:teal;">-</span>  name: helloworld-static-pod-1
<span style="color:pink;">/home/ubuntu/lab/exercise-1/static-pod-1.yaml</span><span style="color:teal;">-</span>  labels:
<span style="color:pink;">/home/ubuntu/lab/exercise-1/static-pod-1.yaml</span><span style="color:teal;">-</span>    app: helloworld
<span style="color:teal;">--</span>
<span style="color:pink;">/home/ubuntu/lab/exercise-1/static-pod-2.yaml</span><span style="color:teal;">:</span><span style="color:red;">metadata</span>:
<span style="color:pink;">/home/ubuntu/lab/exercise-1/static-pod-2.yaml</span><span style="color:teal;">-</span>  name: helloworld-static-pod-2
<span style="color:pink;">/home/ubuntu/lab/exercise-1/static-pod-2.yaml</span><span style="color:teal;">-</span>  labels:
<span style="color:pink;">/home/ubuntu/lab/exercise-1/static-pod-2.yaml</span><span style="color:teal;">-</span>    app: helloworld
</pre>

Create your Service by running the following `kubectl` command:

```
kubectl apply -f service.yaml
```

```console
service "helloworld-service" created
```

Check on the status of your Service by running the following kubectl command:

```
kubectl get service -o wide
```

```console
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE       SELECTOR
helloworld-service   ClusterIP   10.105.78.81     <none>        80/TCP    18s       app=helloworld
kubernetes           ClusterIP   10.96.0.1        <none>        443/TCP   46m       <none>
```

You can see the Service has been created, and it has been provisioned its own IP address.

And if you describe the Service, you can see the two previously deployed Pods have been added as `Endpoints` for the Service.

```
kubectl describe service helloworld-service
```

```console
Name:              helloworld-service
Namespace:         default
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration=...
Selector:          app=helloworld
Type:              ClusterIP
IP:                10.105.78.81
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.2:80,10.244.2.2:80
Session Affinity:  None
Events:            <none>
```

Let's get the Service IP address, and save it to an environment variable.

```
SERVICE_CLUSTER_IP=$(kubectl get service -o wide | awk '/helloworld-service/{print $3}')
echo $SERVICE_CLUSTER_IP
```

```console
10.110.223.163
```

And then we can test that the Service is load balancing between the Pods. The HTTPie command we're using here only outputs the request and response headers.

```
http --print=Hh $SERVICE_CLUSTER_IP
```

```console
GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: 10.110.223.163
User-Agent: HTTPie/0.9.2

HTTP/1.1 200 OK
Connection: close
Content-Encoding: gzip
Content-Length: 1763
Content-Type: text/html; charset=UTF-8
Date: Mon, 14 May 2018 18:58:30 GMT
Server: Apache/2.4.10 (Debian)
Vary: Accept-Encoding
X-Forwarded-To: helloworld-static-pod-1
X-Powered-By: PHP/5.6.36
```

In the response headers, we've included an `X-Forwarded-To` header, which indicates which Pod serviced the request. Running the command a few more times should show load balancing between the Pods.

```
for i in {1..5}; do http --print=Hh $SERVICE_CLUSTER_IP | grep X-Forwarded-To; done
```

```console
X-Forwarded-To: helloworld-static-pod-1
X-Forwarded-To: helloworld-static-pod-2
X-Forwarded-To: helloworld-static-pod-1
X-Forwarded-To: helloworld-static-pod-1
X-Forwarded-To: helloworld-static-pod-2
```

Now, this Service IP address we've been using is a Cluster IP address, meaning it is only routable within the cluster. If we wish to expose our Service to the outsite world, we need to configure an External IP address for the Service.

We can see the public IP address of our Master Node by running the following command:

```
ifdata -pa eth0
```

```console
45.113.232.96
```

We will use this IP address as the External IP address of our Service. The `service-external.yaml` Manifest file has been updated to include the `externalIps` attribute. You can see these changes by running the following command:

```
colordiff -y service.yaml service-external.yaml
```

<pre>
apiVersion: v1                                                  apiVersion: v1
kind: Service                                                   kind: Service
metadata:                                                       metadata:
  name: helloworld-service                                        name: helloworld-service
spec:                                                           spec:
  ports:                                                          ports:
    - port: 80                                                      - port: 80
  selector:                                                       selector:
    app: helloworld                                                 app: helloworld
<span style="color:green;">                                                              &gt;   externalIPs:</span>
<span style="color:green;">                                                              &gt;     - 45.113.232.96</span>
</pre>

And since we are updating our Service object, we just need to `kubectl apply` the updated Manifest file.

```
kubectl apply -f service-external.yaml
```

```console
service "helloworld-service" configured
```

Check on the status of your Service by running the following `kubectl` command:

```
kubectl get service -o wide
```

```console
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP     PORT(S)   AGE       SELECTOR
helloworld-service   ClusterIP   10.105.78.81   45.113.232.96   80/TCP    2m        app=helloworld
kubernetes           ClusterIP   10.96.0.1      <none>          443/TCP   21h       <none>
```

And we can see that the External IP address has been configure for the Service. Let's get the External IP address, and save it to an environment variable.

```
SERVICE_EXTERNAL_IP=$(kubectl get service -o wide | awk '/helloworld-service/{print $4}')
echo $SERVICE_EXTERNAL_IP
```

```console
45.113.232.96
```

And then we can test that the External IP is working as expected.

```
http --print=Hh $SERVICE_EXTERNAL_IP
```

```console
GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: 45.113.232.96
User-Agent: HTTPie/0.9.2

HTTP/1.1 200 OK
Connection: close
Content-Encoding: gzip
Content-Length: 1762
Content-Type: text/html; charset=UTF-8
Date: Thu, 06 Sep 2018 04:24:39 GMT
Server: Apache/2.4.10 (Debian)
Vary: Accept-Encoding
X-Forwarded-To: helloworld-static-pod-2
X-Powered-By: PHP/5.6.36
```

And since this is an Externl IP address, we can test our web application from a browser.

```
echo "Open URL In Browser: http://$SERVICE_EXTERNAL_IP/"
```

```console
Open URL In Browser: http://45.113.232.96/
```

Refresh the page several times. The Service should be load balancing between the Pods.

Now click "Hello World", and choose an image to upload as the background image (gif, jpg, or png). Refresh the page several times, but notice that the image doesn't always load. Why?

Before moving on to the next Exercise, let's clean up these Pods, deleting them with the following `kubectl delete` command:

```
kubectl delete -f ~/lab/exercise-1/static-pod-1.yaml -f ~/lab/exercise-1/static-pod-2.yaml
```

```console
pod "helloworld-static-pod-1" deleted
pod "helloworld-static-pod-2" deleted
```

And after a few seconds, we can confirm that the Pods have indeed been deleted.

```
kubectl get pods -o wide
```

```console
No resources found.
```
