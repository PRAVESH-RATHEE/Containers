Kubernetes
================================================================

Kubernetes cluster is master-worker concept. It is collections of nodes. 

Kubernetes instructions are given in YAML file to Master. Master helps to communicate between dockers and with dockers through YAML file. My different services are present in nodes of cluster. 
Now I have written my YSML file and asks master to run it. Instruction is called through tool called kubectl

Kubectl helps to communicate any Kubernetes cluster.

End user cant access the pod(any of the microservice) of kubernetes directly though IP, because theie IP keeps changigng. So if end user wants to have any service access it must go though the load balancer to provide the static IP of the pod. 
Kubernetes Load balancer is called kubernetes service. 

So if end user wants to have access to the pod, kubernetes service opens the port and allows access oen of the  particular pod. 

Minikube is used to create cluster with one node.  1 node can have many pods inside it. All these pods resides in one container. 
1 node works as worker node as well as master. Kubectl is used to communicate with cluster.

Minikube is installed in virtual machine of ubuntu. Kubectl is installed through Minikube in this VM. 

starting the cluster

minikube start


Running the nginx docker by kubernetes
To run by docker , we write k8s YAML files.

docker container run -p 8080:80 -d --name pravesh-nginx nginx  >> running nginx with docker

Infrastrcuture as a code >> if you are configuring anything in kubernetes, you must write script for it. To di this, we use YML file.
This should be pushed to git as well. And this YML file needs to pass through CI/CD pipeline to make any change in k8s. this is call pd.yml file . 


Create yml file and write the parameters you wanrt to trACK inside the kubernetes.

apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec: 
  containers: 
  - name: pravesh-nginx1
    image: nginx
    ports: 
    - containerPort: 8080


Goto bash and the folder containing the yml file and  run the kubectl to create the pod:

kubectl create -f pod.yml  >> creates the pod

This will run the nginx docker with k8s.

kubectl get pod >> lists all the pods. 

Just like inspect , we have describe to find all about the pod..

kubectl describe pod <pod-name>
  kubectl describe pod pod1

To delete a pod, we callthe command as 

 kubectl delete pod pod1 



 Steps to build and deploy containers:
 ===================================
 1. Dockerise the application(JAVA/NET/CPP/PYTHON/ANY_PROECT)
    1.1 Dockerise the application.
    1.2 Add instructions in the Dockerfile.
    1.3 Docker build to crete the docker image.
    1.4 Upload image to Conatiner Registry
2. Run it on the docker to Kubernetes.

=================================================================================================================================
Replicating the pods in K8s service:
=================================================================================================================================
there are two reasons:

To replicate any pod, we cant have same name and kind which we want to replicate, it will be confused, so we use Labels in order to replicate any pod. Then we create node selector.

when we are exposing our pod, we expose it through kubernetes service, there we need to first replicate the pod as service, we cant allow th user to have direct access to pod. This replicateion is done by kubernetes service. this is another reason to have labels in our yml file. 

man help in kubernetes ":

kubectl get --help

kubectl explain Pod
kubectl explain Pod --recursive


if any pod is already created with same label(key:value), then kubectl will count it as replica already creted and then additional will create replicas-1 pods.

To test this : Run pod4.yml and then replicaset.yl You'll see 2 pods created, while we delete the pod4 , then we will 3 pods get created as replica.


we use apply insteads of create , if we want to apply changes to already created service.Lets say initially we asked to have 3 copies, now we want to create 10. To get this change , we usr this command to update changes : kubectl apply -f replicas.yml
 \
===============================
deleting replicas
===================================

Delete replicaset from kubernetes:

kubectl delete replicaset <app-name>

replicaset alias is "rs"

kubectl delete rs helloworld-container-replicas

===================================================================================================================================================

Concept of service :
We know when we create the pod and its replicas, they are not directly accessed by the end -user because their Ip keeps changing, user needs stable ip.So we expose the pod and pod replicas as service. Because immediately if one pod get clsoed, or deleted, replicas creates another pod with and random IP get generated for it . 
Service gives Static IP, even if the pods get deleted,created or stopped and restared. The service also behaves as load balancer. 
So when we say expose the pod to end -user is same as create the service for end user. Both are same. Exposing pod = creating serrvice.To have particulat static IP, we need to define particular port. 

Label Selector is used by service to map to the Pod . To run the service we must have Pod already running else we wont see the output. 


Different Kinds of services:
Cluster IP

When two pods want to communicate(or two Micro-servcie), they wil communicate through their service to service and not directly from pod-to-pod or service-to-pod or pod-servuice. They need to have IP for this service-to-service talk called as Cluster IP.

So,when request from end user, it will goto cluster-ip called as nodeport, then nodeport will send it to service port of any microservice whichever is requested, then service will send it to the Pod port. 
This is how external user connects to the Pod.

externalIP/userIP:nodePort(MiniKube IP)(31002) <--> Service(StaticIP):Port (31001)<--> Pod(Actual Service): TargetPort(3000)


Serrvice YML
==============

apiVersion: v1
kind: Service
metadata:
  name: helloworld-service
spec:
  ports:
  - port: 31001  (port of the service)
    nodePort: 31002  (ip of the cluster, here it is minikube)
    targetPort: 3000  (port of the pod, if the pod is running then service describe will show it as endpoint.)
    protocol: TCP
  selector:
    app: helloworld
  type: NodePort

  kubectl get svc
  kubectl describe svc helloworld-service\
  kubectl get svc

  minikube service helloworld-service --url 

 >> return the Node IP and port through which external user is connecting to Node and if this node is connected to some IP, it will get the service output.So basically it will connect the external user to the Pod Service using minikube IP(CLuster IP). for e.g. try running the curl  http://192.168.49.2:31002   >> this will provide the output of the Pod acceesed.    
=========================================================================================================================================
DEPLOYMENT TO PRODUCTION:


 How we apply changes made to the source code to the Production/Running Service.lets say we want to update features.  They shoudl goto the docker first and then to the service and then to kubernetes cluster and then apply . How to apply changes without downtime. 

 To do this , we will terminbate one of the replicated pod, UPDATER IT AND THEN RESTART. THEN TAKE ANOTHET REPLICA, TERMINATE IT & APPLY THE CODE CHANGES AND THEN RESTART. HOW  TO DECIDE HOW MANY TO TERMINATE, IT DEPENDS ON PARAM CALLED ROLLING-PARAM. THIS IS CALLED  ROLLING UPDATE. tHIS IS ONE WAY OF DEPLOYMENT, oTHER IS CALLED AS BLUE-GREEN dEPLOYMENT.

 iN BLUE-GREEN DEPLOYMENT, BLUE COLORED PODS ARE THE ONES IN THE RUNNING STATE, GREEN ARE THE ONES WHICH HAS CODE UPDATE. iN BLUE GREEN DEPLOYMENT, WE TELL THE SERVICE TO DISCONNECT FROM BVUE PODS AND CONNECT TO THE GREEN PODS/vms. bLUE BECOMES OLD AND GREEN BECOMES NEW IN PRODUCTION.
 AND THERE ARE MANY STRATEGIES AVAILABLE. oNE MORE IS CALLED KENNEDY STRATEGY . IN THIS , we first deploy the features to some user group, then we will send deployment to those, if it successful, then we will move to other regions.


whenever deployment yml file is created, we always create the service yml file with it. We need to have both. We can keep both deployment and service in one file. To septrate deployment and service manifest file we divide it by using three hyphens and put the code. 

Note : when we create the deployment file, we can specify the replicas within it only, we dont need to specify the separate replica file.

Deployment file 
---
Service file
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: display-hello
  template:
    metadata:
      labels:
        app: display-hello
    spec:
      containers: 
        - name :  pravesh-k8sdemo
          image: arjunachari12/k8s-demo
          ports:
            - name: pod-port
              containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: helloworld-service
spec:
  ports:
    - port: 31001
      nodePort: 31002
      targetPort: 3000
      protocol: TCP
  type: NodePort
  selector:
    app: display-hello


kubectl get pods
kubectl get pods
kubectl get all
kubectl get deployment.apps

Deploying changes to the POD:: we can use the command directly to use it. 


kubectl set image deployment/<deployment-name> <container-name>=<image-to-be-update>

kubectl set image deployment/helloworld-deployment pravesh-k8sdemo=arjunachari12/k8s-demo:latest

kubectl get pod -w >> to watch the deployment happening in terminal


if deployment gets wrong, want to undo the deployment:
kubectl rollout undo deployment/helloworld-deployment

kubectl rollout --help

Another way of deleting the deployment without deleting the deployment.

kubectl delete -f deployment.yml

===============================================

Resources and Limits:

if we dont limit the resources, it will eat all the memory and cpu in any mishappening. 

resources: 
  memory: "250Mi"
  cpu:"500m"


Checking the health of the POD:

if we want to deploy update to kubernetes , we dont want to stop the end-user to get stopped / errors in getting the service . So this happens whenever new container is deployed , but kubernetes service switches from old container to new container and seesapplication inside it is not ready to work . Now older container is also disconnected . This stops the service to end-user. We dont want that user stops getting the service. So before disconnecting the old container, for new container we do check that if the containers is ready and it application inside is also ready. Sp to do this we use liveness and readinesss probe in our depoyment manifest file. o what happens while switching the service from old container

So we send the request to new container, only after checking the health, helath is checked by below lines. 
ports:
            - containerPort: 5000
          resources:
            limits: 
              memory: "128Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /
              port: 5000
            initialDelaySeconds: 15
            timeoutSeconds: 30
          readinessProbe:
            httpGet: 
              path: /
              port: 5000
            initialDelaySeconds: 15
            timeoutSeconds: 30

========================================================

how to passing arguments  to the containers inside the pod, without disturbing the deployments.
in mysql we were passing a parameter -e followed by key=value in launching it, similarly how do we that here ? It means how to pass envinromnemt variables to the particular container inside the pod ? We can call environment varibale as attribute. We use key:data specs in deployment file.


    resources:
      limits:
        memory: "250Mi"
        cpu: "500m"
    env: 
     - name: DEMO_GREETING
       value: "Hello from Pravesh"
     - name: DEMO_FAREWELL
       value: "BYE BYE"


Exec into container of pod and enter into bash:

kubectl exec --stdin --tty environmental-pod -- /bin/bash
kubectl exec --stdin --tty <pod-name> -- /bin/bash
Check the parameters of the pod/container in the pod:
kubectl exec  <pod-name> -- printenv
kubectl exec environmental-pod -- printenv



Better way of passing the environment variables to the container is through the config_map. This will save the headache of going to deployment file and isntead we can do it from separate file. wE can even pass any file also through it.  We will reasd the configmap into deployment file.


env:
      - name: DEMO_GREETING
        valueFrom:
          configMapKeyRef:
            key: DEMO_GREETING
            name: env-configmap
      - name: DEMO_FAREWELL
        valueFrom:
          configMapKeyRef:
            key: DEMO_FAREWELL
            name: env-configmap


  


configmap >> pods. 

We ready the deployment environment variables from configMap.yml file . It is similar to configmap. 

===============================

Secrets in Kubernetes: used to pass the secret info just like environment variables.We use secret.yml file. It is similar like ConfigMap.

=============================================================

Very dangerous to change the replicas or rolling update diorectly in the running pod / already deployed pod

kubectl edit deployments.app <depoyment-name>

===============================================================================================================================================


Namespaces are important for the perspective where many individual components are being developed for a big project and we have smaller projects within it.. say project1 project2 project3...project n. We have many projects inside a cluster. Each project can be used by any person outside the cluster or any other project from cluster. OR lets say we categorise the each compnonet of whole project,then we would like to separate space for them. We do this by creating the namespace first... then we create the pods inside the namespace.

For e.g. we get the namespace of minikube cluster:
ns=namespace

kubectl get ns

if we want to see pods of minikube namespace  we use the following command:

kubectl get all -n kube-system

Every project by default ges created into default namespace. We cam create our own namespace, let say we call it porject1 , we do this by following command:

kubectl create ns project1 >> creates namespace in minikube.

Doing deployment of pod to this particular namespace(project1):
kubectl create -f pod.yml -n project1 >> this created pod under namespace project1

Now if we do kubectl get all >> we wont see the pod gettting created since it shows only default namespace pods, but if we want to see project1 pods. we must use kubectl get all -n project1

Now instead of mentioning -n project1 name, we specify this in pod manifest file under metadata:

metadata:
  name: pod1
  namespace: project1
spec: 

to delete pod from namespace , we must specify -n <namespace-name> param in delete command :  kubectl delete pod pod1 -n project1
it always specify the namespace, it makes the code clean.

kubectl describe ns project1  >> describe the namespace. 

We want to restrict the resource allocation to the project1. We limits two things : Quota and Resources.  One is we cant have resources more than this limit, secondly this namespace cant have more than X services/config files in this project.  We specify these with namespace, without this, namespace has no significance. 


apiVersion: v1
kind: Namespace
metadata:
  name: project2
---
apiVersion: v1
kind: ResourceQuota
metadata:
    name: compute-quota 
    namespace: project2
spec:
    hard:
      requests.cpu: "1"
      requests.memory: 1Gi
      limits.cpu: "2"
      limits.memory: 2Gi
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
  namespace: project2
spec:
  hard:
    pods: "4"
    services: "10"
    secrets: "10"
    configmaps: "10"


We can use namespace: prject2 in any project , then it will use the specified resources and limitations set in the limitation file of namespace. 

apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: project2
spec: 
  containers: 
  - name: pravesh-nginx1
    image: nginx
    ports: 
    - containerPort: 8080


===============================================================================================================================================

Volumes in kubernetres. 

whenever we create/delete a pod, all the data inside  the pod  container also gets deleted, and when new pod comes, it will not have the data, so we need to store such data outside the container, so that new pod/container can mount it when old gets deleted. We do this through volumes. This pod / node will connect to the specified volume as soons as it get created. 
We specify volumes in spec of the manifest file.  
This volume is  generally specified outside the node and not in any pod/container.  We specify it through the config map.

The k8s scheduler, created the duplicate nodes not in one node, but in different nodes, so that if one node is down, it can keep up the service from other nodes. 

Lets create volume for pod1
When we create volume, we create multiple volumes: 
Step to create volume:
1. Create the directory/Host the directory.
2. Create persistent volume and allocate 10Gb.
3. Create Persistent volume claim, Claim 3Gb from Peristent volume. 
4. Create pod manifest file and use Volume. 

x`
pv-claim:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi


pv.yml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv-1
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: "/mnt/data/"


Editing the changes onfly for kubernetes using kubectl directly:
pvc >> persisten volume claim shortcut.

kubectl edit pvc

We specify this in pod file like below :

apiVersion: v1
kind: Pod
metadata:
  name: multi-container
  labels:
    name: myVolumePod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: myclaim
  containers: 
  - name : pravesh-nginx
    image: nginx
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports: 
     - containerPort: 8080
    volumeMounts:
      - mountPath: "/usr/share/nginx/html"
        name: task-pv-storage

=============================================================================================================================================


look at the configuration view of the cluster: this will share the 

kubectl config view... 


==========================================================

Ingress service:
It is used to route to multpile services inside the cluster.  Lets say one pod is search, one is product display, one is product tutorial. They map to different services. ingress maps the request to requested service.
we will deploy two services here and install ingress controller to route the request to requested service. 

There are many controllers available. 

This will add ingress controller inside the cluster:minikube

minikube addons enable ingress

to see what has all it installed  >> kubectl get all -n ingress-nginx

installing deployments using primitives methods:  we can directly install the pods deployment using one command instead of creating the yml file. We call that as primitive methods'

kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
Exposing this as service using primitive commands: kubectl expose deployment web --type=NodePort --port=8080
We can verify that we are using the service.  kubectl expose deployment web2 --type=NodePort --port=8080

We create the ingress file with annotations and specify the rules in it what kind of request should goto which service. 

apiVersion: networking.k8s.io/v1
   
kind: Ingress
   
metadata:
   
  name: example-ingress
   
  annotations:
   
    nginx.ingress.kubernetes.io/rewrite-target: /$1
   
spec:
   
  rules:
   
    - host: hello-world.info
   
      http:
   
        paths:
   
          - path: /
   
            pathType: Prefix
   
            backend:
   
              service:
   
                name: web
   
                port:
   
                  number: 8080     
          - path: /v2
            pathType: Prefix
            backend:
              service:
                name: web2
                port:
                  number: 8080
creating the deployment 

 kubectl create -f ingress.yml
 minikube service web2 --url
 minikube service web --url

test the service request where it goes : 

 curl http://192.168.49.2 -H 'Host: hello-world.info'
 curl http://192.168.49.2/v2 -H 'Host: hello-world.info'


 =============================================================

 kubernetes jobs is similar like ChronoJob.  Like every at 9am pull data from this service and one at 5pm do this. We do this everyday by porgram we call it is Jobs. Similar thing we have in Kubernetes called as Jobs.

check job.yml file. 


===================================================
creating roles in Kubernetes : this gives access to the user/ developer/ read/write/delete accessed . This is from administrator point of view. We call it as Roles in K8s

check role.yml file

kubectl get role


=====================

Now Lets alk about clusters:
creating our own clusters, we use Kubeadm. We can easily do it on AWS instance. We create the cluster, we also create the nodes in this cluster, this can be easily done with 2 commands. 

Then while specifying the pod , we can  specify that this pod should goto which node of the cluster. 
There is one field called nodeSelector: key:value in deployment yml file. 


=====================

Helm:Since we create single yml files for deployment, service, secret, config files. The same cnt be used for differnet environments like in QA we want to check different values and versions of code. So Helm used to create reusable conmfigurable kuluster objects for differenr environments(dev,QA,etc) We can specify the differnet varibales here and it will set those values in it deployment,secret,service manfestion files for different environment. Sets the values like image version, code version directly through this file in different enviornments. 
so helm configures all yml files. 

So steps boil down to the following things:




 Steps to build and deploy containers:
 ===================================
 1. Dockerise the application(JAVA/NET/CPP/PYTHON/ANY_PROECT)
    1.1 Dockerise the application.
    1.2 Add instructions in the Dockerfile.
    1.3 Docker build to crete the docker image.
    1.4 Upload image to Conatiner Registry
2. Create Helm Chart
    2.1 Create all YML reusable manifest
    2.2 Upload chart to shared repo.
    2.3 Upload chart to shared repository.

3. Run chart on the docker to Kubernetes.


So whenever we wnt to create reusbale manifest file, we use helm. 

We run the helm repo of sql. Do the port forwarding and see it in prometheus. 
run the following commands to see your pod/deployment doing on prometheus using the following commands: For many sample applications. Helm chart is already compose, we use SQL helm chart here. 

Helm chart has deployment.service.replicas yml files in in and with this one file we are able to run all the files at once. 

E.g. 

sudo apt-get install helm
helm repo add my-repo https://charts.bitnami.com/bitnami   >> install SQL using helm chart. 
kubectl get all  >> gives output of all the files created by helm. 
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts  >> add prometheus helm chart to visualise the nodes. 

helm repo update
helm install [RELEASE_NAME] prometheus-community/prometheus
helm install prom prometheus-community/prometheus >> installed prometheus in our system. 

Run this command to start the server observation used by node/pod. 

export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}") kubectl --namespace default port-forward $POD_NAME 9090

Do port forwarding in vscode and visualise it in browser. 
 
