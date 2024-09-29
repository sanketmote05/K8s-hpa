# K8s-hpa

Exercise: Implement Horizontal Pod Autoscaler (HPA) in EKS

In this exercise, you will learn how to set up and configure the Horizontal Pod Autoscaler (HPA) on an Amazon Elastic Kubernetes Service (EKS) cluster to automatically scale your applications based on CPU utilization.


---

Prerequisites

1. A working Amazon EKS cluster.


2. kubectl installed and configured for your EKS cluster.


3. Metrics Server installed in the EKS cluster (required for HPA to retrieve metrics).


4. A sample application deployed in the cluster (we will use a simple Nginx deployment for this exercise).


5. AWS CLI configured to interact with EKS.




---

Step 1: Deploy the Metrics Server

1. Clone the metrics server repository and apply the configuration:

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


2. Verify that the Metrics Server is working:

kubectl get apiservices | grep metrics


3. Check if the Metrics Server is able to collect metrics:

kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes"




---

Step 2: Deploy a Sample Application

Let’s create a simple Nginx deployment that we will autoscale.

1. Create an nginx-deployment.yaml file with the following content:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "500m"


2. Deploy the application:

kubectl apply -f nginx-deployment.yaml


3. Verify that the deployment is successful:

kubectl get deployments




---

Step 3: Create the Horizontal Pod Autoscaler

We will now create the Horizontal Pod Autoscaler to scale the Nginx pods based on CPU utilization.

1. Run the following command to create an HPA that targets the Nginx deployment:

kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=5

This command creates an HPA that will:

Scale the pods when CPU utilization exceeds 50%.

Maintain a minimum of 2 pods and a maximum of 5 pods.



2. Verify the HPA has been created:

kubectl get hpa

You should see the HPA for the nginx-deployment.




---

Step 4: Generate Load to Trigger Autoscaling

To observe autoscaling in action, we will generate some load on the Nginx pods.

1. Create a pod to generate load:

kubectl run -i --tty load-generator --image=busybox /bin/sh


2. Inside the pod, run the following command to send continuous requests to the Nginx service:

while true; do wget -q -O- http://nginx-deployment; done

This command will continuously send HTTP requests to the Nginx pods, which should increase CPU utilization.


3. In a separate terminal, monitor the HPA:

kubectl get hpa -w

The HPA should eventually detect the increase in CPU utilization and scale up the number of Nginx pods.




---

Step 5: Verify Scaling

1. Monitor the deployment to observe scaling changes:

kubectl get deployments -w

You should see the number of replicas increase as the HPA scales the deployment based on the CPU load.


2. Once the load generation is stopped or CPU utilization falls below the threshold, the HPA will automatically scale down the pods.




---

Step 6: Clean Up

Once you're done with the exercise, clean up the resources by running the following commands:

kubectl delete deployment nginx-deployment
kubectl delete hpa nginx-deployment


---

Recap

1. Deployed a Metrics Server to enable resource metrics collection.


2. Created a sample Nginx deployment.


3. Configured a Horizontal Pod Autoscaler to automatically scale your deployment based on CPU utilization.


4. Generated load to trigger the autoscaler and observed the scaling behavior.



This exercise demonstrates how to leverage Kubernetes' built-in autoscaling capabilities in an EKS environment.


