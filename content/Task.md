### Task 1: Setup and Configuration

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

1. Dockerize the Application: Build a Docker image for the provided web application and push it to a public container registry of your choice.

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

    
2. Deploy the Application Using Argo CD:
    

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

1. Define a Rollout Strategy: Modify the application's deployment to use Argo Rollouts, specifying a canary release strategy in the rollout definition.
	
	**A canary rollout is a deployment strategy where the operator releases a new version of their application to a small percentage of the production traffic.**
		
	Hence first we release new version to 20% of traffic and then we promote little by little. 
	Here the major things are 
		- setweight
		- duration
		- promote
		- abort
		

    
2. Trigger a Rollout: Make a change to the application, update the Docker image, push the new version to your registry, and update the rollout definition to use this new image.
		As we have changed our web app and created another image called node-todo-app-2 we will update the rollout definition.
		![[Pasted image 20240310125329.png]]
		
    
3. Monitor the Rollout: Use Argo Rollouts to monitor the deployment of the new version, ensuring the canary release successfully completes.
		There are two ways to monitor argo rollouts one is through cli by using the command
		`kubectl argo rollouts get rollout nodejs-app-rollout --watch`
		and other way is through dashboard that is accessible at https://localhost:3100

### Task 4: Cleanup

We clean up the cluster by following the below steps
First we get all resources from the respective namespace
And delete the resource by using `kubectl delete resource_name -n namespace`
