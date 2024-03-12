### Task 1: Setup and Configuration
#setup

1. **Create a GitRepository: Create a Github repository and host your source code in the same repository.**

	First we create a github repository as shown
	![[Pasted image 20240309083403.png]]

	Then we clone it locally and push the source code to github.
	![[Pasted image 20240309082524.png]]



    
2. **Install Argo CD on Your Kubernetes Cluster: Follow the official Argo CD documentation to install and set up Argo CD.**

	I am installing Argo CD on K3s cluster on VM 
	The steps are from official docs and are as follows:

	First we create namespace for argocd
	![[Pasted image 20240309172442.png]]

	Mentioning the same namespace we apply the yaml config
	![[Pasted image 20240309172620.png]]

	Now we will wait until the pods are ready
	![[Pasted image 20240309172745.png]]

	after all the pods are ready then we press Ctrl+C to exit the running command

	Now to access the dashboard of ArgoCD first we should expose the argocd-server service as by default the service-type is ClusterIP
	Change the argocd-server service type to `LoadBalancer`:
	![[Pasted image 20240309173303.png]]

	we can see below the service type is changed to LoadBalancer
	![[Pasted image 20240309173343.png]]
	now we open the browser and go to https://localhost:3228
	![[Pasted image 20240309173911.png]]

	That brings us to argocd Dashboard
	![[Pasted image 20240309174126.png]]

	now we will require password so we go to terminal and use the command
```plain
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

put username admin and password given by the above command and we are welcomed with below dashboard.
	![[Pasted image 20240309174532.png]]



3. **Install Argo Rollouts: Install the Argo Rollouts controller in your Kubernetes cluster, following the official guide.**
	By using the official docs we successfully installed the argo rollouts.
	![[Pasted image 20240309235819.png]]

	First we install controller and then we install kubectl plugin
	![[Pasted image 20240309235852.png]]

	we access the dashboard for argo rollouts using `kubectl argo rollouts dashboard`
	it's accessible at https://localhost:3100
    

### Task 2: Creating the GitOps Pipeline
#pipeline

1. **Dockerize the Application: Build a Docker image for the provided web application and push it to a public container registry of your choice.**

	In order to create docker image first we go into folder where Dockerfile is located

	![[Pasted image 20240309083856.png]]

	Now we create a repository on dockerhub by clicking on create repository

	![[Pasted image 20240309084407.png]]


	![[Pasted image 20240309084702.png]]


	Here we can see repository is created 
	![[Pasted image 20240309084744.png]]

	Now we are ready to build the docker image
	Why we did the above step first, it's because when will be pushing the docker image it's name should match the repo name otherwise it will fail.
	
	![[Pasted image 20240309085306.png]]
	![[Pasted image 20240309085311.png]]
	As we can see docker image is built successfully
	![[Pasted image 20240309085400.png]]

	for pushing the image to registery first we perform docker **login** in terminal itself
	![[Pasted image 20240309085639.png]]

	![[Pasted image 20240309085832.png]]
	As we can see the docker image is successfully pushed to public container registery dockerhub.
	![[Pasted image 20240309090014.png]]

    
2. **Deploy the Application Using Argo CD:**
    

- Modify the Kubernetes manifests in your forked repository to use the Docker image you pushed.
		We have added deployment and service files in manifest in github
    
- Set up Argo CD to monitor your repository and automatically deploy changes to your Kubernetes cluster.



	By using this Dashboard we monitor the deployment, we can see the application is deployed successfully and 3 replicas are created.
	![[Pasted image 20240307195040.png]]

	![[Pasted image 20240307195445.png]]

	Now from github we change the replica count to 2 and it is reflected in the dashboard.
	![[Pasted image 20240307195740.png]]

	And since we want three replicas again so what we do is just rollback

	Rollback


	![[Pasted image 20240307202502.png]]
	![[Pasted image 20240307202507.png]]



	![[Pasted image 20240307202511.png]]
	As we see there are 3 replicas, we returned to previous deployment

	Gitops in true sense keeping track of rollback



### Task 3: Implementing a Canary Release with Argo Rollouts
#rollouts

1. **Define a Rollout Strategy: Modify the application's deployment to use Argo Rollouts, specifying a canary release strategy in the rollout definition.**
	
	**A canary rollout is a deployment strategy where the operator releases a new version of their application to a small percentage of the production traffic.**
		
	Hence first we release new version to 20% of traffic and then we promote little by little. 
	Here the major things to take care are 
		- setweight
		- duration
		- promote
		- abort
		

    
2. **Trigger a Rollout: Make a change to the application, update the Docker image, push the new version to your registry, and update the rollout definition to use this new image.**
Below given is the initial state 

![[Pasted image 20240312185104.png]]
![[Pasted image 20240312185355.png]]

Initially we used csborle/node-todo-app image and deployed the rollout it gets deployed similar to a deployment but as we change the image to csborle/node-todo-app-2  using the command

kubectl argo rollouts set image node-todo \
  node-todo=csborle/node-todo-app-2

![[Pasted image 20240312190337.png]]
![[Pasted image 20240312190506.png]]
Here the rollout is paused but as the duration is not given it will be in the same state till we don't promote hence we run the promote command and eventually our new image is scales up and is marked as healthy and the older containers are terminated and scaled down.

![[Pasted image 20240312191748.png]]
![[Pasted image 20240312191753.png]]
		
		
		
    
3. **Monitor the Rollout: Use Argo Rollouts to monitor the deployment of the new version, ensuring the canary release successfully completes.**
		There are two ways to monitor argo rollouts one is through cli by using the command
		`kubectl argo rollouts get rollout nodejs-app-rollout --watch`
	![[Pasted image 20240312191748.png]]
		
and other way is through dashboard that is accessible at https://localhost:3100
![[Pasted image 20240312191753.png]]

### Task 4: Cleanup
#cleanup

**For ArgoCD**, go to the dashboard, we can just click delete the application button. and all the resources are deleted and our cluster is cleaned.
In this case resources are created in myproj namespace 
![[Pasted image 20240312213340.png]]
By using dashboard we confirm delete
![[Pasted image 20240312213345.png]]
![[Pasted image 20240312213601.png]]

**For Rollouts**
here we have deployed rollout in myproj namespace
![[Pasted image 20240312215218.png]]

use `kubectl delete rollouts --all -n myproj`
and in the end for service `kubectl delete svc/node-todo -n myproj`

![[Pasted image 20240312215321.png]]

### But does it follow GitOps Principles ? - Yes
#principles

- [x] **Declarative** 
(yaml, what you see in git is what you deployed)
A system managed  by GitOps must have it's desired state expressed declaratively.

- [x] **Versioned and Immutable** 
(changes are versioned, not only about git any version solutions)
Dersired state is stored in a way that enforces immutability, versioning and retains a complete version history..

- [x] **Pulled Automatically** 
(gitops controller actively monitors, it can be pull or push(webhooks etc) automatically updation)
Software Agents automatically pull the desired state declarations  from the source.

- [x] **Continously Reconciled** 
(due to it provides security as one cannot change resource without declaring in git, it always trys to keep "actual system state = desired state")
Software Agents continously observe actual system state  and attempt to apply the desired state.

*source -* https://opengitops.dev/


### Challenges
#challenges

- **Accessing ArgoCD Dashboard** - Referred documentation but didn't read carefully as there are two ways for accessing dashboard one is by changing the service type of LoadBalancer and other was Ingress method port forwarding, often while trying I opted for second one but didn't access the dashbard so I got back to fundamentals and checked the service type it was ClusterIP which is not exposed so I referred the docs and corrected the mistake.

- **Kubernetes cluster** - It's about minikube I was not able to start it, it gave the virtualization error hence Iooked for alternatives like lightweight k3s and installed it on a VM and then installed argocd and accessed the dashboard.

- **Rollouts** - Initially a bit tricky to interpret.

###  Learning
#learning

- Documentation is good resource if read carefully.
- Whenever there is an error get back to fundamentals of it first to understand it.