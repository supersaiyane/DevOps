
## Kubernetes — Learn Adapter Container Pattern

### Understanding Adapter Container Pattern With an Example Project

![Photo by [Krzysztof Niewolny](https://unsplash.com/@epan5?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/6920/0*t_WV41Q2ArfkMaXu)

Kubernetes is an open-source container orchestration engine for automating deployment, scaling, and management of containerized applications. A pod is the basic building block of kubernetes application. Kubernetes manages pods instead of containers and pods encapsulate containers. A pod may contain one or more containers, storage, IP addresses, and, options that govern how containers should run inside the pod.

A pod that contains one container refers to a single container pod and it is the most common kubernetes use case. A pod that contains Multiple co-related containers refers to a multi-container pod. There are few patterns for multi-container pods one of them is the adapter container pattern. In this post, we will see this pattern in detail with an example project.

* ***What are Adapter Containers***

* ***Other Patterns***

* ***Example Project***

* ***Test With Deployment Object***

* ***How to Configure Resource Limits***

* ***When should we use this pattern***

* ***Summary***

* ***Conclusion***

## ***What are Adapter Containers***

There are so many applications that are heterogeneous in nature which means they don’t contain the same interface or not consistent with other systems. This pattern extends and enhances the functionality of current containers without changing it as the sidecar container pattern. Nowadays, We know that we use container technology to wrap all the dependencies for the application to run anywhere. A container does only one thing and does that thing very well.

Imagine that you have the pod with a single container working very well but, it doesn't have the same interface with other systems to integrate or work with it. How can you make this container to have a unified interface with a standardized format so that other systems can to your container? This adapter container pattern really helps exactly in that situation.

![**Adapter Container Pattern**](https://cdn-images-1.medium.com/max/2000/1*xLP8r_kSqM7zZCY3ePlo2A.png)

If you look at the above diagram, you can define any number of containers for Adapter containers and your main container works along with it successfully. All the Containers will be executed parallelly and the whole functionality works only if both types of containers are running successfully. Most of the time these adapter containers are simple and small that consume fewer resources than the main container.

## ***Other Patterns***

There are other patterns that are useful for everyday kubernetes workloads

* [Init Container Pattern](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-init-container-pattern-7a757742de6b)

* [Sidecar Container Pattern](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-sidecar-container-pattern-6d8c21f873d)

* [Ambassador Container Pattern](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-ambassador-container-pattern-bc2e1331bd3a)

## Example Project

Here is the example project you can clone and run it on your machine. You need to install Minikube as a prerequisite.

    [https://github.com/bbachi/k8s-adaptor-container-pattern.git](https://github.com/bbachi/k8s-adaptor-container-pattern.git)

Imagine your main container generates logs in text format and external systems need to consume these logs in a JSON format. You need to have some mechanism to convert text files to JSON format.

![**Adapter Pattern**](https://cdn-images-1.medium.com/max/2000/1*rEbFagY67leNciUcQGnjrA.png)

The Adapter container is a simple express node server that reads from the location /var/log/file.log and produces JSON format. Let’s implement a simple project to understand this pattern. The Adapter container is a simple express API that serves these logs as a JSON response. Here is the file server.js

```node.js
const express = require('express');
const lineReader = require('line-reader'),
      Promise = require('bluebird');
const app = express(),
      port = 3080;

const logs = []

var eachLine = Promise.promisify(lineReader.eachLine);

app.get('/logs', (req,res) => {
    eachLine('/var/log/file.log', function(line) {
        console.log(line);
        logs.push({
            time: line.split("#")[0],
            message: line.split("#")[1]
        });
    }).then(function() {
        console.log("I'm done!!");
        res.send(logs);
    }).then(function(err){
        console.log(err);
    });
});

app.listen(port, () => {
    console.log(`Server listening on the port::${port}`);
});
```

Here is the Dockerfile that converts this app into a Docker image.

```Dockerfile
# base image
FROM node:slim

# setting the work direcotry
WORKDIR /usr/src/app

# copy package.json
COPY ./package.json .

# install dependencies
RUN npm install

# COPY server.js
COPY ./server.js .

CMD ["node","server.js"]
```

The following are the commands creating a Docker image and pushing into Docker Hub.

    // build the Docker image
    docker build -t bbachin1/adapter-node-server .

    // list the images
    docker images

    // push the image
    docker push bbachin1/adapter-node-server

![**Docker Hub**](https://cdn-images-1.medium.com/max/2000/1*q2idB6CsirbI921RB-_Niw.png)

Once the Docker image is ready and pushed into Docker Hub and it can be available publicly. You can pull it when creating a pod. Here is a simple pod that has a main and adapter container. The main container is busybox generating some logs to the path ***/var/log/file.log*** from the volume mount **workdir** location. Since the Adapter container and main container runs parallel node express API will display the new log information every time you hit in the browser.

 <iframe src="https://medium.com/media/edffc6ef7d6eb618aa50a20be37d7ca8" frameborder=0></iframe>

    // create the pod
    kubectl create -f pod.yml

    // list the pods
    kubectl get po

    // exec into pod
    kubectl exec -it adapter-container-demo -c adapter-container -- /bin/sh
    # apt-get update && apt-get install -y curl
    # curl localhost

You can install curl and query the localhost and check the response.

![](https://cdn-images-1.medium.com/max/2284/1*7rBRZ3NVwqNSxm8sC7kVbQ.png)

![**Testing Adapter Container**](https://cdn-images-1.medium.com/max/2426/1*m9YJ3puiZMedoZFdaFQdxw.png)

## Test With Deployment Object

Let’s create a deployment object with the same pod specification with 5 replicas. I have created a service with the port type NodePort so that we can access the deployment from the browser. Pods are dynamic here and the deployment controller always tries to maintain the desired state that’s why you can’t have one static IP Address to access the pods so that you have to create a service that exposes the static port to the outside world. Internally service maps to the port **3080** based on the selectors. You will see that in action in a while.

![**Deployment**](https://cdn-images-1.medium.com/max/2000/1*vQgkOjaIKaQcH3ERa7M2EA.png)

Let’s look at the below deployment object where we define one main container and one adapter container. All the containers run in parallel. The main container creates logs in the location **/var/log/file.log. **The adapter container serves those log files in a JSON format on the REST API on the port **3080**. You will see that in action in a while.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: node-webapp
  name: node-webapp
spec:
  replicas: 5
  selector:
    matchLabels:
      app: node-webapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: node-webapp
    spec:
      containers:
      - image: busybox
        command: ["/bin/sh"]
        args: ["-c", "while true; do echo $(date -u)'#This is log' >> /var/log/file.log; sleep 5;done"]
        name: main-container
        resources: {}
        volumeMounts:
          - name: var-logs
            mountPath: /var/log
      - image: bbachin1/adapter-node-server
        name: adapter-container
        imagePullPolicy: Always
        resources: {}
        ports:
          - containerPort: 3080
        volumeMounts:
        - name: var-logs
          mountPath: /var/log
      dnsPolicy: Default
      volumes:
      - name: var-logs
        emptyDir: {}
status: {}

---

apiVersion: v1
kind: Service
metadata:
  name: node-webapp
  labels:
    run: node-webapp
spec:
  ports:
  - port: 3080
    protocol: TCP
  selector:
    app: node-webapp
  type: NodePort
```  

Let’s follow these commands to test the deployment.

    // create a deployment
    kubectl create -f manifest.yml

    // list the deployment, pods, and service
    kubectl get deploy -o wide
    kubectl get po -o wide
    kubectl get svc -o wide

![**deployment in action**](https://cdn-images-1.medium.com/max/3152/1*oSDFk2MPXzq0teSoIEvz6Q.png)

In the above diagram, You can see 5 pods running in different IP addresses and the service object maps the port **32063** to the port 30**80**. You can actually access this deployment from the browser from the kubernetes master IP address **192.168.64.2** and the service port **32063**.

    http://192.168.64.2:32063/logs

![**Adapter container pattern**](https://cdn-images-1.medium.com/max/2124/1*oZFwGD0Ze82oKnBVlepWBQ.png)

You can even test the pod with the following commands

    // exec into main container of the pod
    kubectl exec -it <pod name> -c adapter-container -- /bin/sh

    // install curl
    # apt-get update && apt-get install -y curl
    # curl localhost

## How to Configure Resource Limits

Configuring resource limits is very important when it comes to Adapter containers. The main point we need to understand here is All the containers run in parallel so when you configure resource limits for the pod you have to take that into consideration.

* The sum of all the resource limits of the main containers as well as adapter containers (Since all the containers run in parallel)

## When should we use this pattern?

These are some of the scenarios where you can use this pattern

* Whenever you want to extend the functionality of the existing single container pod without touching the existing one.

* Whenever you want to enhance the functionality of the existing single container pod without touching the existing one.

* Whenever there is a need to convert or standardize the format for the rest of the systems.

## Summary

* A pod that contains one container refers to a single container pod and it is the most common kubernetes use case.

* A pod that contains Multiple co-related containers refers to a multi-container pod.

* The Adapter container pattern is a specialization of sidecar containers.

* Adapter containers run in parallel with the main container. So that you need to consider the resource limits of adaptor containers while defining request/resource limits for the pod.

* The application containers and Adapter containers run-in parallel which means all the containers run at the same time. So that you need to sum up all the request/resource limits of the containers while defining request/resource limits for the pod.

* You should configure health checks for Adapter containers as main containers to make sure they are healthy.

* All the pods in the deployment object don’t have static IP addresses so that you need a service object to expose to the outside world.

* The service object internally maps to the port container port based on the selectors.

* You can use this pattern where your application or main containers need to standardize some format for the external systems.

## Conclusion

It’s always to good to know already proven kubernetes patterns. Make sure all your adapter containers are simple and small enough because you have to sum up all the resources/request limits while defining resource limits for the pod. You need to make sure Adapter containers are also healthy by configuring health checks.
