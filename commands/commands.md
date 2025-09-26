Pod

kubectl get po
kubectl describe po pod_name
kubectl get po -o 
kubetcl delete po pod_name

Create a new Pod 
in 2 way
1st by yaml
2nd by command:
kubectl run redis --image=redis --dry-run=client -o yaml

kubectl run redis --image=redis --dry-run=client -o yaml > redis.yaml

to change the image of the Pod which is running
kubectl edit pod (pod_name)
kubetl set  image pod/(pod_name) container_name/imageto be changes


RepplicaSet
kunectl get rs
 kubectl get rs -o wide
 ![alt text](image.png)

 


to scale the replicas
kubectl edit rs (rs_name)
kubectl edit yamlfilename.yaml
kubectl replace -f replicaset-defination.yaml
kubectl scale --replicas=6 -f yamlfilename.yaml
kubectl scale --replicas=6 replicaset (rs-name) myapp-replicaset

Deployment:
What is a Deployment?

A Deployment is a higher-level Kubernetes object that manages ReplicaSets and Pods.

It provides declarative updates (rolling updates, rollbacks, scaling) for applications.

Compared to a ReplicaSet, a Deployment is the recommended way to run and manage applications because it handles updates automatically.


ey Features of Deployment

Replica Management → Keeps the desired number of Pods running.

Rolling Updates → Updates Pods with a new image without downtime.

kubectl set image deployment/nginx-deployment nginx=nginx:1.26
kubectl rollout status deployment/nginx-deployment


Rollbacks → Easily revert to a previous version.

kubectl rollout undo deployment/nginx-deployment


Scaling → Increase or decrease replicas on the fly.

kubectl scale deployment nginx-deployment --replicas=5


Self-Healing → If a Pod crashes, the Deployment automatically creates a new one.

Summary:

A ReplicaSet ensures availability of Pods.

A Deployment manages ReplicaSets and makes updating/scaling apps much easier.

Always prefer Deployment over using a ReplicaSet directly.

kubectl get deployments
kubectl get replicas
kubectl get pods

Tips
Certification Tip!
Here's a tip!

As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. Using the kubectl run command can help in generating a YAML template. And sometimes, you can even get away with just the kubectl run command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with specific name and image you can simply run the kubectl run command.

Use the below set of commands and try the previous practice tests again, but this time try to use the below commands instead of YAML files. Try to use these as much as you can going forward in all exercises

Reference (Bookmark this page for exam. It will be very handy):

https://kubernetes.io/docs/reference/kubectl/conventions/

Create an NGINX Pod

kubectl run nginx --image=nginx

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

kubectl run nginx --image=nginx --dry-run=client -o yaml

Create a deployment

kubectl create deployment --image=nginx nginx

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

Make necessary changes to the file (for example, adding more replicas) and then create the deployment.

kubectl create -f nginx-deployment.yaml



OR

In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.

kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml

