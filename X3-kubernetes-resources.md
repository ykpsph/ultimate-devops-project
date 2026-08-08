# Kubernetes Resources
 ### Deployment
 ### Service
 <br>
- if you want to deploy this application on local environment, containers are fine, but containers are **ephemeral** in nature. That means they are very short lived. 
- And anytime if the container goes down and if it comes up, the IP address of containers is going to change.

    - That is one of the reasons why Kubernetes is used in your local environment, higher environment as well as the staging environment.

There are two primary reasons to choosing Kubernetes : 
1. Scaling and healing features of Kubernetes
2. Service discovery of Kubernetes

![kubernetes](/notes/images/image-12.png)

note : scaling and healing are handled by deployment resource of kubernetes
note : service discovery is handled by service resource of kubernetes

### What is Scaling and Healing ?
- Let's say you deployed a container, by default if you don't have any restart policy, if the container goes down and if you don't have restart policy, the container does not come up. That is, there is no auto healing for containers by default.

  - And similarly, let's say you have deployed the container and you know, during the peak peak time, since we are talking about e-commerce applications. During the peak sale time you would need high availability for your containers.
  - That means just one copy of your container will not be enough. You would need multiple copies of your container and you need a load balancer balancing the node.


So here we are talking about two things.
- scaling
- healing
> These particular things are provided by a resource of Kubernetes which is called as **deployment**.

- Deployment is a very popular resource of Kubernetes in fact, most of the time, you would deploy your microservices in Kubernetes using the deployment resource.

#### So what exactly this deployment resource does to implement these things?
1. In Kubernetes there is the lowest level of deployment which is called as **Pod**. You can consider it as somewhat equivalent to a container in a world of Docker.
However, a pod can be a single container or multiple containers as well.
But just like how container holds your microservice, a pod is a resource in Kubernetes that actually holds your microservice.

2. What Kubernetes does is, because a container goes down in the typical container environment because of multiple reasons, it asks users to use a deployment resource called **deployment**.
And this resource called deployment will handle your pod.
So when you want to deploy a microservice, if you create that microservice as **deployment**, it will create an intermediate resource called **replica set**.
And this **replica set** will spin up the containers or the pods in Kubernetes world. 

3. in the deployment YAML file if you say replica count as 3, then the intermediate resource called replica set, it is going to make sure that all the time the pod count is 3.
That means for some reason, if pod goes down, that is the microservice in the pod goes down and because of that, if the pod goes down, replica set will immediately create a new pod.
So it will always make sure that the count is as mentioned in the deployment.

4. `So what problem it is solving?` - It is solving the problem of **auto healing**. So in deployment, if you mention the replica count as four immediately it will create a replica set.
And this replica set will ensure that always four copies of your container are running within the pods.

5. So the replica set is making sure there are 4 replicas. For some reason, a pod has gone down, immediately it will create a new pod and it will connect that pod.
A auto healing is automatically implemented. So a **replica set** is a controller in Kubernetes which is taking care of these things.
Peki, who creates the replica set? It is a **deployment resource**. 

> So that's why overall a deployment resource is the one that takes care of healing and scaling in Kubernetes.
