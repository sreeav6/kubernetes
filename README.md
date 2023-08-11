# Here we are taking example from google cloud labs

- We will learn how to Deploy, Scale, Update the website with zero downtime with GKE

# Things which will learn as
1) You should have a GCP Account created or You can use which ever cloud you like but I will be using GCP
2) First we will create a cluster
3) We will colne the repo and test the application by installing nodejs dependencies
4) We will create Docker container using Cloud Build
5) We will deploy the container to GKE
6) We will expose the app to the external end users by using services concept
7) We will scale the deployment to 'n' replicas
8) Here basically the business says please update the app with the version 2 and deploy to GKE
9) Once updated we will deploy the app to GKE with zero down time

# Let's break down step by step
- First we will create GCP Account and activate the cloud shell later we will use command like gcloud auth list <which helps to authorize the 
  project you are working> and then check if the project is not configured please set the project so it will easy for cleanup in order to set 
  the project we use gcloud config set project<project-id>, then set zone as well { gcloud config set compute/zone <zone name> }. 

# Let's create a GKE Cluster
- In order to create this we have prerequisites that is you should enable container registry api to do this <gcloud services enable 
  container.googleapis.com>
- Now create cluster with three nodes gcloud container clusters create <give any name here> --num-nodes 3 Example: <gcloud container clusters 
  create firstcluster --num-nodes=3>
- Here it will take time to create the cluster, In the mean while you can observe the logs in the Cloud Logging section how the cluster 
  creating is happening. Once cluster creating is done we will run the command to check whether 3 compute instances were created or not to 
  check this we run <gcloud compute instances list> you will observe the output of it. You can check in the Google cloud console by clicking 
  Navigation Menu-->Kubernetes Engine-->Clusters

# Let's Clone the repository and test the application
- First navigate to your home directory by typing <cd ~> then clone the repo
- <git clone https://github.com/googlecodelabs/monolith-to-microservices.git>
- Then navigate <cd ~/monolith-to-microservices> then run < ./setup.sh > this is for installing nodejs dependencies which requires for 
  deploying the application.
- Then test with this command < nvm install --lts > where it will ensure the latest version of nvm.
- Let's test the application now by navigating to < cd ~/monolith-to-microservices/monolith > and then run < npm start >
- It will display the output <Monolith listening on port 8080!> In the console you can click on the preview icon which is present on the top 
  right corner

# Let's create a Docker Container using Cloud Build 
- Cloud Build helps in creating a Docker container and push to container registery. Futher more Google cloud build will compress the files 
  from a directory and move
  to the Google cloud storage bucket. Now the build will pick the file and start the Docker build process using the Dockerfile.
- In order to use Cloud Build we need to use cloud build API to do this type in the cloud shell as <gcloud services enable 
  cloudbuild.googleapis.com>
- Navigate to this path <cd ~/monolith-to-microservices/monolith> then startbuilding <gcloud builds submit --tag 
  gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 .>
- It will take few minutes you can observe the build process in the console click Navigation Menu-->Cloud Build-->History

# Now Deploy the container to GKE
- If we talk about kubernetes it understand it's applications as Pods, there fore it is called a smallest object in k8s
- The flow goes like this first create deployment resource where it manages multiple copies of your application called replicas and schedules 
  them to run on the individual nodes in the cluster.
- Use this command to deploy the application <kubectl create deployment monolith --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0>
- Now verify the deployment by using < kubectl get all > the output will display the pods, services, deployments, replica sets e.t.c
- You can verify in the console by clicking Navigation Menu-->Kubernetes-->Cluster

# Now see how to solve the erros by troubleshooting with some commands
- If you want to check the pod { kubectl describe pod/< pod name> }
- If you want to see the pods < kubectl get pods >
- If you want to see the deployments < kubectl get deployments >
- If you want to see the replica sets < kubectl get rs >
- Even you can get all once using < kubectl get all >
- If you want to check the deployment { kubectl describe deployment/< deployment name> }
- Even you can edit the pods, deployments, replica sets by using  {kubectl edit deployment/< deployment name > }, { kubectl edit pod/< pod name > }
- In order to delete the pod use <kubectl delete pod {pod name}> 

# Here we will expose the application to the internet
- Here we use a concept called services namely {Cluster IP [which is default one], NodePort [whoever has access to the nodes only they can 
  access, LoadBalancer [where external end users can access yout application] we are using as LoadBalancer for exposing the applciation
- Basically a Service provides networking and IP support to your application's Pods. GKE creates an external IP and a Load Balancer for your  
  application
- command as follows <kubectl expose deployment {type your deployname} --type=LoadBalancer --port 80 --target-port 8080
- Example: kubectl expose deployment monolith --type=LoadBalancer --port 80 --target-port 8080

# Let's access the service  
- Here we are going to access the LoadBalance service which you were exposed by using this command <kubectl get service>
- Even in the console you can click on Navigation Menu-->Kubernetes-->Service and Ingress [here you will find external ip just click on it]
- Now you can able to see the application is up and running

# Now scale our deployment 
- Here we are scaling our deployment to 3 replicas to do this type this command
- <kubectl scale deployment {type your deployment name} --replicas=3
- Verify the deployment has been scaled by using < kubectl get all >

# Now let's do some changes to the application 
- In order to make some changes please follow the below commands
- cd ~/monolith-to-microservices/react-app/src/pages/Home
- mv index.js.new index.js
- To verify the changes took affect or not use this command < cat ~/monolith-to-microservices/react-app/src/pages/Home/index.js >
- Let's build this new app to do this type below commands
      cat ~/monolith-to-microservices/react-app/src/pages/Home/index.js
      npm run build:monolith
- Now again we need to rebuild the Docker container and push it to the Google cloud container registry here we use tag as 2.0.0 to do that      use the below command
- Examples:
      cd ~/monolith-to-microservices/monolith
      gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 .
 # Let's deploy this newer image to GKE
   - Even though the application get updated with newer image across all replicas kubernetes rolling update mechanism will ensure the app is       up and running. Let's inform to kubernetes that we want to deploy the app with newer version to do that please follow below commands
     kubectl set image deployment/{type your deployment name} monolith=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0
   - Example: kubectl set image deployment/monolith monolith=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0
   - Verify your deployment is running  <kubectl get pods>
   - you can just click on the external ip followed by it's port or simply use npm start

  # It's time to clean up
  - Delete git repo to do this type < cd ~ >and then rm -rf monolith-to-microservices
  - Delete google container images to do use below ones
    gcloud container images delete gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --quiet
    gcloud container images delete gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 --quiet
  - Delete build artifacts from cloud storage
    gcloud builds list | grep 'SOURCE' | cut -d ' ' -f2 | while read line; do gsutil rm $line; done
  - Delete GKE service to do that < kubectl delete service {type your service name} >
  - Delete deployment to do that < kubectl delete deployment {deployment name} >
  - Delete GKE Cluster to do that < gcloud container clusters delete {type your cluster name} >

    
    
 
  









