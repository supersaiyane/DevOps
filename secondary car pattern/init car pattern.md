
## Kubernetes — Learn Init Container Pattern

### Understanding Init Container Pattern With an Example Project

![Photo by [Judson Moore](https://unsplash.com/@judsonlmoore?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/6912/0*yE2YpojrHFticQ6X)

Kubernetes is an open-source container orchestration engine for automating deployment, scaling, and management of containerized applications. A pod is the basic building block of kubernetes application. Kubernetes manages pods instead of containers and pods encapsulate containers. A pod may contain one or more containers, storage, IP addresses, and, options that govern how containers should run inside the pod.

A pod that contains one container refers to a single container pod and it is the most common kubernetes use case. A pod that contains Multiple co-related containers refers to a multi-container pod. There are few patterns for multi-container pods on of them is the init container pattern. In this post, we will see this pattern in detail with an example project.

* ***What are Init Containers***

* ***Other Patterns***

* ***Example Project***

* ***Test With Deployment Object***

* ***How to Configure Resource Limits***

* ***When should we use this pattern***

* ***Summary***

* ***Conclusion***

## ***What are Init Containers***

Init Containers are the containers that should run and complete before the startup of the main container in the pod. It provides a separate lifecycle for the initialization so that it enables separation of concerns in the applications. For example, you need to install some specific software before you want to run your application you can do that installation part in the Init Container of the pod.

![**Init Container Pattern**](https://cdn-images-1.medium.com/max/2000/1*9erV-cjHBzdKA0Mx5jH0PA.png)

If you look at the above diagram, you can define n number of containers for Init containers and your main container starts only after all the Init containers are terminated successfully. All the init Containers will be executed sequentially and if there is an error in the Init container the pod will be restarted which means all the Init containers are executed again. So, it's better to design your Init container as simple, quick, and Idompodent.

## ***Other Patterns***

There are other patterns that are useful for everyday kubernetes workloads

* [Sidecar Container Pattern](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-sidecar-container-pattern-6d8c21f873d)

* [Adapter Container Pattern](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-adaptor-container-pattern-97674285983c)

* [Ambassador Container Pattern](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-ambassador-container-pattern-bc2e1331bd3a)

## ***Example Project***

Here is the example project you can clone and run it on your machine. You need to install Minikube as a prerequisite.

    [https://github.com/bbachi/k8s-init-container-pattern.git](https://github.com/bbachi/k8s-init-container-pattern.git)

Let’s implement a simple project to understand this pattern. Here is a simple pod that has main and init containers. The main container is nginx serving on the port **80 **that takes the index.html from the volume mount **workdir** location. The init container with the image busybox will create the file at the same location. Since the Init container is run first and terminated before the start of the main container Nginx will find the file at the location **/usr/share/nginx/html.**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  # These containers are run during pod initialization
  initContainers:
  - name: busybox
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "echo '<html><h1>Hi I am from Init container</h1><html>' >> /work-dir/index.html"]
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
 ```   

Let’s run the following commands to create the pod and test it out.

    // create the pod
    kubectl create -f pod.yml

    // list the pods
    kubectl get po

    // exec into pod
    kubectl exec -it init-container-demo -- /bin/sh
    # apt-get update && apt-get install -y curl
    # curl localhost

You can install curl and query the localhost and check the response.

![**Testing Init Container**](https://cdn-images-1.medium.com/max/2622/1*_ZFfdkdJuOYoSz5EwOk7sQ.png)

## ***Test With Deployment Object***

Let’s create a deployment object with the same pod specification with 5 replicas. I have created a service with the port type NodePort so that we can access the deployment from the browser. Pods are dynamic here and the deployment controller always tries to maintain the desired state that's why you can’t have one static IP Address to access the pods so that you have to create a service that exposes the static port to the outside world. Internally service maps to the port 80 based on the selectors. You will see that in action in a while.

![**Deployment**](https://cdn-images-1.medium.com/max/2354/1*rJLbGuAOirg94MAasK3kog.png)

Let’s look at the below deployment object where we define two init containers. Init containers run sequentially the first init container writes this line **“<html><h1>Hi I am from Init container</h1>” **and the second container appends this line **“<p>Hi I am from second Init container</p></html>” **to the file index.html. you can see that in action in a while.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-webapp
  name: nginx-webapp
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx-webapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-webapp
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      # These containers are run during pod initialization
      initContainers:
      - name: busybox1
        image: busybox
        command: ["/bin/sh"]
        args: ["-c", "echo '<html><h1>Hi I am from Init container</h1>' >> /work-dir/index.html"]
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      - name: busybox2
        image: busybox
        command: ["/bin/sh"]
        args: ["-c", "echo '<p>Hi I am from second Init container</p></html>' >> /work-dir/index.html"]
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      dnsPolicy: Default
      volumes:
      - name: workdir
        emptyDir: {}
status: {}

---

apiVersion: v1
kind: Service
metadata:
  name: nginx-webapp
  labels:
    run: nginx-webapp
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx-webapp
  type: NodePort
```  

Let’s follow these commands to test the deployment.

    // create a deployment
    kubectl create -f manifest.yml

    // list the deployment, pods, and service
    kubectl get deploy -o wide
    kubectl get po -o wide
    kubectl get svc -o wide

![**Deployment in action**](https://cdn-images-1.medium.com/max/3194/1*F-9T6Lg00CudnR0SdNauRg.png)

In the above diagram, You can see 5 pods running in different IP addresses and the service object maps the port 32244 to the port 80. You can actually access this deployment from the browser from the kubernetes master IP address **192.168.64.2** and the service port **32244**.

    http://192.168.64.2:32244

![**Access the deployment object from the browser**](https://cdn-images-1.medium.com/max/2000/1*hOoz9wcMj2v2kuzFDMiQCw.png)

## ***How to Configure Resource Limits***

Configuring resource limits is very important when it comes to Init containers. The main point we need to understand here is Init containers run first before the start of the main container so when you configure resource limits for the pod you have to take that into consideration.

* The highest init container resource limits (since Init containers run sequentially)

* The sum of all the resource limits of the main containers (Since all the application containers run in parallel)

## ***When should we use this pattern***

These are some of the scenarios where you can use this pattern

* You can use this pattern where your application or main containers need some prerequisites such as installing some software, database setup, permissions on the file system before starting.

* You can use this pattern where you want to delay the start of the main containers.

## Summary

* A pod that contains one container refers to a single container pod and it is the most common kubernetes use case.

* A pod that contains Multiple co-related containers refers to a multi-container pod.

* The Init container pattern is one of the patterns that we use regularly for initialization tasks.

* Init containers run sequentially which means one by one. So that you need to consider using the highest resource limit while defining request/resource limits for the pod.

* The application containers run-in parallel which means all the application containers runs at the same time. So that you need to sum up all the request/resource limits of the containers while defining request/resource limits for the pod.

* All the pods in the deployment object don’t have static IP addresses so that you need a service object to expose to the outside world.

* The service object internally maps to the port container port based on the selectors.

* You can use this pattern where your application or main containers need some prerequisites such as installing some software, database setup, permissions on the file system before starting.

## Conclusion

It’s always to good to know already proven kubernetes patterns. Use this pattern whenever there are initialization tasks you need to perform and you want to have a separation of concerns. Always remember Init containers runs sequentially and should terminate first before the start of the main containers.
