# simplek8s
Notes from Kubernetes Tutorials.


## Yaml config files
You'll have multiple Config files, not just one. 
Each config file defines an Object - this is not exactly the same as creating a Docker Container
Kubectl pares both configs and creates two Objects

## Objects\
  - a thing that exists inside of the K8s cluster\
  - Objects can beStatefulSet, REplicaController Pod, Service

## Kinds of Objects\
 - Pod : used to run a container\
 - Service: Sets up networking stuff for k8s cluster

## Client-pod.yaml  Config file (Pod)\
 - 'metadata: labels' - are important way to tag Objects\
 - apiVersion v1 opens up access to a predefined set of Object types\
    - Each API Version defiens a different set of 'objects' we can use\
    - First you have to figure out what kind of Object you want to create,\
      look up that api version, then specify that API version\
 - 'Kind' specifies the type of object you are creating with it\
 - 'spec:containers:' specifies the containers running inside the Pod spec. kinda similar to the docker-compose spec for containers\
    - The port mapping in 'spec:containers' does not allow access to Port 3000.\
      That's the purpose of the networking config file/ Service Object
 - Node(Pod(Container))\

## Client-node-port.yaml Config File (Service)\
 - 'spec:type' specifies the subtype of the Service\
- 'spec:selector: - specifies every Object with these selectors, and exposes its ports to the outside world\
- 'spec:ports':\
    - port: a port that a different container inside the cluster could use to access this pod/container (multi-client)\
    - targetPort: port inside of the Pod you want to open up traffic to\
    - nodePort: the port we type into our browser to access the Pod

## Deploying Objects\
 - Command `kubectl apply -f client-pod.yaml` `kubectl apply -f client-node-port.yaml`

 ## Pods\
  - A grouping of containers with a very common purpose\
  - There's no ability run a naked container in k8s\
  - The smallest hing you can deploy is a Pod\
    - Node(Pod(Container))\
  - Pods are meant to group containers with a very similar purpose\
    - Containers that are very tightly coupled\
    - Example: A Pod containing Postgres, a Logger, and a backup manager Container\
        - If the Postgres goes down, the logger/backup containers have no purpose\
        - Very close integration, so contain it in a Pod\
    - Reasons you'd run more than one container in a pod is if they're tightly integrated like this Example

## Services\
 - sets up some amount of networking inside a k8s cluster\
 - Services have subtypes:\
    - ClusterIP\
    - NodePort\
      - exposes a Container to the outside world (Only good for dev purposes!)\
      - Kubernetes Node(Kube-Proxy -> NodePort -> Pod('multi-client' container))\
    - LoadBalancer\
    - Ingress

## Imperative vs Declarative Updaates?\
 - Use declarative (There should be X) and not imperative (Create X)

## Declarative Updaes\
 - Updates the config file that originally created the pod\

 - If you update a config file - it will know which Pod to update\
    - Config Files has a Name, Type and image associated to it\
    - Name and Kind are Unique IDs together. Don't change these\
      - every time you update a config file, if the Name and Kind are the same, KubeCtl will update all pods with the same Name and Kind
      - if the Name and Kind are different from the file previous to the update - KubeCtl will create a new Pod
    - If you change anything else in the config file, kubectl will update the object rather than create a brand new Object\
    - Pod updates may not add or remove containers
 - `kubectl apply -f client-pod.yaml` to update with new config

## Looking inside pods / Inspecting Pods\
  - use command `kubectl describe <object type> <object name>`\
  - omit name to get details about all objects of that type\
  - shows all events that happened inside that pod\
  - You can see the image(s) being used on that pod

## Limitiations updating pods - and introducing "Deployments"\
 - You can't update fields "port", "name", or "containers" if the type of Object is a Pod\
 - You can use the type "Deployment" to achieve this\
    - Deployment Objects: Runs a set of identical pods (one or more Pods with the exact same set of containers),\
      ensuring they have the correct config\
 - You can use Deployments to achieve the same functionality as just a Pod, since it can run just one Pod

 - In practice,\
    - Pods are good for one-off dev purposes and rarely used in production.\
    - Deployments are good for dev and for production

- Deployments let you update fields on (a set of) Pods that you cannot update if it was just a Pod


## Creating Deployment config files

 - "spec:replicas" details how many Pods to create using the "spec:template"
 - "spec:template" lists out the config for every single pod that is contained by this deployment. it's like the exact configuration for a pod. it has its own "metadata" and "spec"
 - "spec:selector" matches labels to make updates to pods
  - labels are unique by key and value. label "component: web" is unique from label "component: notweb" or label "notcomponent: web"


## Removing existing Object
`kubectl delete -f <config file>`
- unfortunately this is imperative by default. declarative might be more wprk
 

## Updating Deployment config files
`kubectl apply -f client-deployemtn.yaml` 
to apply a new deployment or update existing deployment

`kubectl get deployments` to view existing deployemnts
`kubectl get pods` to view existing pods


## Why use Services?
 Every single pod you create gets its own randomly assigned IP address that is internal to the virtual machine. 
   -  If the pod gets changed for any reason - it might get an entirely new IP address
Pods are coming adn going all the time - Services abstract away the difficultys of connecting to Pods



## Updating Deployment Images
 - Must be done imperatively via kubectl
 - Tag your Docker image with a version number `sharadshekar:multi-client:v4` and push it up to DOckerhub
 - Run a kubectl command to imperatively force the deployment to use the new image version

`kubectl set image <object type>/<object name> <container_name>=<new_image_to_use>`
`kubectl set image deployment/client-deployment client=sharadshekar/multi-client:v5`

## Configuring Docker CLI
 - you have a copy of Docker unning on your computer, and a second copy of Docker running in minikubeVM
 - Reconfiguring your local copy of docker?
  - run `eval $(minikube docker-env`)` 
  - just `minikube docker-env` will print out env variables inside the minikube instance
    - This only configures you current terminal window (current bash session)
 - Why mess with Docker in the node?
  - use all the same debugging techniques learned with Docker CLI
    - `docker ps`
    - `docker logs <id>`
    - `docker exec -it <id> sh`
    - You can do this with kubectl too
      - `kubectl get pods`
      - `kubectl exect -it <pod_name> sh`
  - Manually kill containers to test Kubernetes's ability to self-heal
  - Delete cached images inside the node


