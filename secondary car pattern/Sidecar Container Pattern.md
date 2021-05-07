
## Kubernetes — Learn Sidecar Container Pattern

### Understanding Sidecar Container Pattern With an Example Project

![Photo by [hidde schalm](https://unsplash.com/@hdsfotografie95?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/5780/0*Pn1zAVjpb0gWGAjN)

Kubernetes is an open-source container orchestration engine for automating deployment, scaling, and management of containerized applications. A pod is the basic building block of kubernetes application. Kubernetes manages pods instead of containers and pods encapsulate containers. A pod may contain one or more containers, storage, IP addresses, and, options that govern how containers should run inside the pod.

A pod that contains one container refers to a single container pod and it is the most common kubernetes use case. A pod that contains Multiple co-related containers refers to a multi-container pod. There are few patterns for multi-container pods one of them is the sidecar container pattern. In this post, we will see this pattern in detail with an example project.

* ***What are Sidecar Containers***

* ***Other Patterns***

* ***Example Project***

* ***Test With Deployment Object***

* ***How to Configure Resource Limits***

* ***When should we use this pattern***

* ***Summary***

* ***Conclusion***

## ***What are Sidecar Containers***

Sidecar containers are the containers that should run along with the main container in the pod. This sidecar pattern extends and enhances the functionality of current containers without changing it. Nowadays, We know that we use container technology to wrap all the dependencies for the application to run anywhere. A container does only one thing and does that thing very well.

Imagine that you have the pod with a single container working very well and you want to add some functionality to the current container without touching or changing, how can you add the additional functionality or extending the current functionality? This sidecar container pattern really helps exactly in that situation.

![**Sidecar Container Pattern**](https://cdn-images-1.medium.com/max/2000/1*2O7FNVCO5k83WIbbGbbl-A.png)

If you look at the above diagram, you can define any number of containers for Sidecar containers and your main container works along with it successfully. All the Containers will be executed parallelly and the whole functionality works only if both types of containers are running successfully. Most of the time these sidecar containers are simple and small that consume fewer resources than the main container.

## Other Patterns

There are other patterns that are useful for everyday kubernetes workloads

* [Init Container Pattern](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-init-container-pattern-7a757742de6b)

* [Adapter Container Pattern](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-adaptor-container-pattern-97674285983c)

* [Ambassador Container Pattern](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-ambassador-container-pattern-bc2e1331bd3a)

## Example Project

Here is the example project you can clone and run it on your machine. You need to install Minikube as a prerequisite.

    [https://github.com/bbachi/k8s-sidecar-container-pattern.git](https://github.com/bbachi/k8s-sidecar-container-pattern.git)

Let’s implement a simple project to understand this pattern. Here is a simple pod that has main and sidecar containers. The main container is nginx serving on the port **80 **that takes the index.html from the volume mount **workdir** location. The Sidecar container with the image busybox creates logs in the same location with a timestamp. Since the Sidecar container and main container runs parallel Nginx will display the new log information every time you hit in the browser.

 <iframe src="https://medium.com/media/93399daa63df3ade49a84e798c7eefb3" frameborder=0></iframe>

    // create the pod
    kubectl create -f pod.yml

    // list the pods
    kubectl get po

    // exec into pod
    kubectl exec -it sidecar-container-demo -c main-container -- /bin/sh
    # apt-get update && apt-get install -y curl
    # curl localhost

You can install curl and query the localhost and check the response.

![**Testing Sidecar Container**](https://cdn-images-1.medium.com/max/2478/1*fnO5X_9xSjVA-p20McIBcA.png)

## Test With Deployment Object

Let’s create a deployment object with the same pod specification with 5 replicas. I have created a service with the port type NodePort so that we can access the deployment from the browser. Pods are dynamic here and the deployment controller always tries to maintain the desired state that’s why you can’t have one static IP Address to access the pods so that you have to create a service that exposes the static port to the outside world. Internally service maps to the port 80 based on the selectors. You will see that in action in a while.

![**Deployment**](https://cdn-images-1.medium.com/max/2348/1*HDuHfh_GiD-sS11b4BcMgg.png)

Let’s look at the below deployment object where we define one main container and two sidecar containers. All the containers run in parallel. The two sidecar containers create logs in the location **/var/log. **The main container Nginx serves those log files as when we hit the NGINX web server from the port 80. You will see that in action in a while.

 <iframe src="https://medium.com/media/2f6b36402390d3ac72a8867e0c871ce7" frameborder=0></iframe>

Let’s follow these commands to test the deployment.

    // create a deployment
    kubectl create -f manifest.yml

    // list the deployment, pods, and service
    kubectl get deploy -o wide
    kubectl get po -o wide
    kubectl get svc -o wide

![**deployment in action**](https://cdn-images-1.medium.com/max/2686/1*irx1u_Yy0M5zQC9FCKgJ5w.png)

In the above diagram, You can see 5 pods running in different IP addresses and the service object maps the port **32123** to the port **80**. You can actually access this deployment from the browser from the kubernetes master IP address **192.168.64.2** and the service port **32123**.

    http://192.168.64.2:32123

You can even test the pod with the following commands

    // exec into main container of the pod
    kubectl exec -it nginx-webapp-7c8b4d4f8d-9qmdm -c main-container -- /bin/sh

    // install curl
    # apt-get update && apt-get install -y curl
    # curl localhost

![**Sidecar Pattern in action**](https://cdn-images-1.medium.com/max/2710/1*aAwFGqYIFpobBt_BlxTrjw.png)

## How to Configure Resource Limits

Configuring resource limits is very important when it comes to Sidecar containers. The main point we need to understand here is All the containers run in parallel so when you configure resource limits for the pod you have to take that into consideration.

* The sum of all the resource limits of the main containers as well as sidecar containers (Since all the containers run in parallel)

## When should we use this pattern

These are some of the scenarios where you can use this pattern

* Whenever you want to extend the functionality of the existing single container pod without touching the existing one.

* Whenever you want to enhance the functionality of the existing single container pod without touching the existing one.

* You can use this pattern to synchronize the main container code with the git server pull.

* You can use this pattern for sending log events to the external server.

* You can use this pattern for network-related tasks.

## Summary

* A pod that contains one container refers to a single container pod and it is the most common kubernetes use case.

* A pod that contains Multiple co-related containers refers to a multi-container pod.

* The Sidecar container pattern is one of the patterns that we use regularly for extending or enhancing pre-existing containers.

* Sidecar containers run in parallel with the main container. So that you need to consider resource limits of sidecar containers while defining request/resource limits for the pod.

* The application containers and Sidecar containers run-in parallel which means all the containers run at the same time. So that you need to sum up all the request/resource limits of the containers while defining request/resource limits for the pod.

* You should configure health checks for sidecar containers as main containers to make sure they are healthy.

* All the pods in the deployment object don’t have static IP addresses so that you need a service object to expose to the outside world.

* The service object internally maps to the port container port based on the selectors.

* You can use this pattern where your application or main containers need extending or enhancing the current functionality.

## Conclusion

It’s always to good to know already proven kubernetes patterns. Make sure all your sidecar containers are simple and small enough because you have to sum up all the resources/request limits while defining resource limits for the pod. You need to make sure Sidecar containers are also healthy by configuring health checks. So it’s important to know when to combine your functionality into the main container or have a separate container.
