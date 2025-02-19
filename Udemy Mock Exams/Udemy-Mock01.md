### SECTION: APPLICATION DESIGN AND BUILD

02. kubectl config use-context cluster1

Create a storage class with the name banana-sc-ckad08-str as per the properties given below:


- Provisioner should be kubernetes.io/no-provisioner,

- Volume binding mode should be WaitForFirstConsumer.

- Volume expansion should be enabled.

我的配置:

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: banana-sc-ckad08-str
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer


正確配置:
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: banana-sc-ckad08-str
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer



07. (空題) Helm

For this question, please set the context to cluster2 by running:

kubectl config use-context cluster2

On the student-node, a Helm chart repository is given under the /opt/ path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. The files have some issues. Fix those issues and deploy them with the following specifications: -

The release name should be webapp-color-apd.
All the resources should be deployed on the frontend-apd namespace.
The service type should be node port.
Scale the deployment to 3.
Application version should be 1.20.0.
NOTE: - Remember to make necessary changes in the values.yaml and Chart.yaml files according to the specifications, and, to fix the issues, inspect the template files.

You can start new terminal to open student-node command.


03. (回答正確) For this question, please set the context to cluster2 by running:


kubectl config use-context cluster2



In the ckad-job namespace, create a cronjob named simple-node-job to run every 30 minutes to list all the running processes inside a container that used node image (the command needs to be run in a shell).



In Unix-based operating systems, ps -eaf can be use to list all the running processes.





08. (配置正確) For this question, please set the context to cluster3 by running:


kubectl config use-context cluster3



We have deployed a pod pod22-ckad-svcn in the default namespace. Create a service svc22-ckad-svcn that will expose the pod at port 6335.


Note: Use the imperative command for the above scenario.
我的配置:

student-node ~ ➜  k get svc
NAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
external-webserver-ckad01-svcn   ClusterIP   172.20.19.20     <none>        80/TCP           80m
kubernetes                       ClusterIP   172.20.0.1       <none>        443/TCP          140m
route-apd-svc                    NodePort    172.20.185.40    <none>        8080:31993/TCP   93m
svc22-ckad-svcn                  NodePort    172.20.211.213   <none>        6335:31348/TCP   82m

### SECTION: APPLICATION DEPLOYMENT
05.  
For this question, please set the context to cluster3 by running:


kubectl config use-context cluster3



In this task, we have to create two identical environments that are running different versions of the application. The team decided to use the Blue/green deployment method to deploy a total of 10 application pods which can mitigate common risks such as downtime and rollback capability.

Also, we have to route traffic in such a way that 30% of the traffic is sent to the green-apd environment and the rest is sent to the blue-apd environment. All the development processes will happen on cluster 3 because it has enough resources for scalability and utility consumption.


Specification details for creating a blue-apd deployment are listed below: -

The name of the deployment is blue-apd.
Use the label type-one: blue.
Use the image kodekloud/webapp-color:v1.
Add labels to the pod type-one: blue and version: v1.

Specification details for creating a green-apd deployment are listed below: -

The name of the deployment is green-apd.
Use the label type-two: green.
Use the image kodekloud/webapp-color:v2.
Add labels to the pod type-two: green and version: v1.

We have to create a service called route-apd-svc for these deployments. Details are here: -

The name of the service is route-apd-svc.
Use the correct service type to access the application from outside the cluster and application should listen on port 8080.
Use the selector label version: v1.

NOTE: - We do not need to increase replicas for the deployments, and all the resources should be created in the default namespace.

You can check the status of the application from the terminal by running the curl command with the following syntax:

curl http://cluster3-controlplane:NODE-PORT

You can SSH into the cluster3 using ssh cluster3-controlplane command.

Solution:

Step 1. Use the kubectl create command to create a deployment manifest file as follows: -
kubectl create deployment blue-apd --image=kodekloud/webapp-color:v1 --dry-run=client -o yaml > <FILE-NAME-1>.yaml

Do the same for the other deployment and service.

kubectl create deployment green-apd --image=kodekloud/webapp-color:v2 --dry-run=client -o yaml > <FILE-NAME-2>.yaml
kubectl create service nodeport route-apd-svc --tcp=8080:8080 --dry-run=client -oyaml > <FILE-NAME-3>.yaml


Step2: Open the file with any text editor such as vi or nano and make the changes as per given in the specifications. It should look like this: -

---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    type-one: blue
  name: blue-apd
spec:
  replicas: 7
  selector:
    matchLabels:
      type-one: blue
      version: v1
  template:
    metadata:
      labels:
        version: v1
        type-one: blue
    spec:
      containers:
        - image: kodekloud/webapp-color:v1
          name: blue-apd

We will deploy a total of 10 application pods. Also, we have to route 70% traffic to blue-apd and 30% traffic to the green-apd deployment according to the task description.

Since the service distributes traffic to all pods equally, we have to set the replica count 7 to the blue-apd deployment so that the given service will send ~70% traffic to the deployment pods.



green-apd deployment should look like this: -

---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    type-two: green
  name: green-apd
spec:
  replicas: 3
  selector:
    matchLabels:
      type-two: green
      version: v1
  template:
    metadata:
      labels:
        type-two: green
        version: v1
    spec:
      containers:
        - image: kodekloud/webapp-color:v2
          name: green-apd

route-apd-svc service should look like this: -


---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: route-apd-svc
  name: route-apd-svc
spec:
  type: NodePort
  ports:
    - port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    version: v1

Step3: Now, create a deployment and service by using the kubectl create -f command: -
kubectl create -f <FILE-NAME-1>.yaml -f <FILE-NAME-2>.yaml -f <FILE-NAME-3>.yaml

06. For this question, please set the context to cluster3 by running:


kubectl config use-context cluster3



On cluster3, in the dev-001 namespace, one of the interns deployed one web application called news-apd.

After successfully deploying on the worker node, we start getting alerts about the pod crashing.

We want you to inspect the dev-001 namespace and fix those issues.


Solution:
In this task, we will use the kubectl get, kubectl describe, kubectl logs and kubectl edit commands. Here are the steps: -


Step 1. To check all the resources in the specific namespaces in the cluster3, we would have to run the following command:
kubectl get all -n dev-001



It will list all the available resources of the dev-001 namespace.



Step2 We can see that one of the pods is in an error state. Use the kubectl describe command to get detailed information of that pod: -
kubectl describe -n dev-001 po <POD-NAME>



The output of the kubectl describe command includes information about the Pod's status, including its IP address, the containers running in the Pod, and their current state.
It also includes information about the Pod's configuration, such as its labels, annotations, and resource requirements.



Step3 Now, we will use the kubectl logs command to retrieve the logs of a container running in a Pod. Use the following command as follows: -
kubectl logs -n dev-001 <POD-NAME> <CONTAINER-NAME>



Here, <POD-NAME> is the name of the Pod in which the container is running, and <CONTAINER-NAME> is the name of the container whose logs we want to retrieve. If the Pod has only one container, we can omit the <CONTAINER-NAME> argument.



In the logs, we will see that there is a typo in the sleep command which needs to be correct. Use the kubectl edit command as follows: -
kubectl edit deploy -n dev-001 news-apd



After fixing the typo, press the ESC button and type :wq. This command will save the changes to the file and then quit vi editor.



Run the kubectl get command again to check the pod's status.

kubectl get pods -n dev-001



The pod should be running.

### SECTION: SERVICES AND NETWORKING
10. For this question, please set the context to cluster3 by running:


kubectl config use-context cluster3


We have an external webserver running on student-node which is exposed at port 9999.

We have also created a service called external-webserver-ckad01-svcn that can connect to our local webserver from within the cluster3 but, at the moment, it is not working as expected.



Fix the issue so that other pods within cluster3 can use external-webserver-ckad01-svcn service to access the webserver.

Solution:

Let's check if the webserver is working or not:
student-node ~ ➜  curl student-node:9999
...
<h1>Welcome to nginx!</h1>
...

Now we will check if service is correctly defined:
student-node ~ ➜  kubectl describe svc external-webserver-ckad01-svcn 
Name:              external-webserver-ckad01-svcn
Namespace:         default
.
.
Endpoints:         <none> # there are no endpoints for the service
...

As we can see there is no endpoints specified for the service, hence we won't be able to get any output. Since we can not destroy any k8s object, let's create the endpoint manually for this service as shown below:

student-node ~ ➜  export IP_ADDR=$(ifconfig eth0 | grep inet | awk '{print $2}')

student-node ~ ➜ kubectl --context cluster3 apply -f - <<EOF
apiVersion: v1
kind: Endpoints
metadata:
# the name here should match the name of the Service
  name: external-webserver-ckad01-svcn
subsets:
  - addresses:
      - ip: $IP_ADDR
    ports:
      - port: 9999
EOF

Finally check if the curl test works now:
student-node ~ ➜  kubectl --context cluster3 run --rm  -i test-curl-pod --image=curlimages/curl --restart=Never -- curl -m 2 external-webserver-ckad01-svcn
...
<title>Welcome to nginx!</title>
...

11. For this question, please set the context to cluster2 by running:


kubectl config use-context cluster2



Deploy a pod with name webapp-svcn using the kodekloud/webapp-color image with the label tier=msg.



Now, Create a service webapp-service-svcn to expose the pod webapp-svcn application within the cluster on port 6379.

Solution:
On student-node, use the command kubectl run webapp-svcn --image=kodekloud/webapp-color -l tier=msg



Now run the command: kubectl expose pod webapp-svcn --port=6379 --name webapp-service-svcn.

12. For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



For this scenario, create a Service called ckad12-service that routes traffic to an external IP address.


Please note that service should listen on port 53 and be of type ExternalName. Use the external IP address 8.8.8.8



Create the service in the default namespace.

Solution:

Create the service using the following manifest:
apiVersion: v1
kind: Service
metadata:
  name: ckad12-service
spec:
  type: ExternalName
  externalName: 8.8.8.8
  ports:
    - name: http
      port: 53
      targetPort: 53


### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY

13. (配置正確) For this question, please set the context to cluster1 by running:

kubectl config use-context cluster1

Create a pod named ckad17-qos-aecs-3 in namespace ckad17-nqoss-aecs with image nginx and container name ckad17-qos-ctr-3-aecs.

Define other fields such that the Pod is configured to use the Quality of Service (QoS) class of Burstable.

Also retrieve the name and QoS class of each Pod in the namespace ckad17-nqoss-aecs in the below format and save the output to a file named qos_status_aecs in the /root directory.

Format:

NAME    QOS
pod-1   qos_class
pod-2   qos_class

我的配置" 
echo "NAME    QOS" > /root/qos_status_aecs

kubectl get pods -n ckad17-nqoss-aecs \
  -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.qosClass}{"\n"}{end}' \
>> /root/qos_status_aecs


cat /root/qos_status_aecs
NAME    QOS
ckad17-qos-aecs-2    BestEffort
ckad17-qos-aecs-1    BestEffort
ckad17-qos-aecs-3    BestEffort


參考其他方法:

使用自定义列（Custom Columns）

kubectl get pods -n ckad17-nqoss-aecs \
  -o=custom-columns=NAME:.metadata.name,QOS:.status.qosClass \
> /root/qos_status_aecs
这样会自动带有列名，并在 /root/qos_status_aecs 文件里生成类似：

NAME                        QOS
ckad17-qos-aecs-2           BestEffort
ckad17-qos-aecs-1           BestEffort
ckad17-qos-aecs-3           BestEffort
如果仅想要纯数据（无标题），可加上 --no-headers：

kubectl get pods -n ckad17-nqoss-aecs \
  -o=custom-columns=NAME:.metadata.name,QOS:.status.qosClass \
  --no-headers \
> /root/qos_status_aecs

1.  For this question, please set the context to cluster2 by running:


kubectl config use-context cluster2



Create a custom resource my-anime of kind Anime with the below specifications:


Name of Anime: Death Note
Episode Count: 37


TIP: You may find the respective CRD with anime substring in it.

student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  kubectl get crd | grep -i anime
animes.animes.k8s.io

student-node ~ ➜  kubectl get crd animes.animes.k8s.io \
                 -o json \
                 | jq .spec.versions[].schema.openAPIV3Schema.properties.spec.properties
{
  "animeName": {
    "type": "string"
  },
  "episodeCount": {
    "maximum": 52,
    "minimum": 24,
    "type": "integer"
  }
}

student-node ~ ➜  k api-resources | grep anime
animes                            an           animes.k8s.io/v1alpha1                 true         Anime

student-node ~ ➜  cat << YAML | kubectl apply -f -
 apiVersion: animes.k8s.io/v1alpha1
 kind: Anime
 metadata:
   name: my-anime
 spec:
   animeName: "Death Note"
   episodeCount: 37
YAML
anime.animes.k8s.io/my-anime created

student-node ~ ➜  k get an my-anime 
NAME       AGE
my-anime   23s



15. For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



Create a ConfigMap named ckad04-config-multi-env-files-aecs in the default namespace from the environment(env) files provided at /root/ckad04-multi-cm directory.


Solution:

student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl create configmap ckad04-config-multi-env-files-aecs \
         --from-env-file=/root/ckad04-multi-cm/file1.properties \
         --from-env-file=/root/ckad04-multi-cm/file2.properties
configmap/ckad04-config-multi-env-files-aecs created

student-node ~ ➜  k get cm ckad04-config-multi-env-files-aecs -o yaml
apiVersion: v1
data:
  allowed: "true"
  difficulty: fairlyEasy
  exam: ckad
  modetype: openbook
  practice: must
  retries: "2"
kind: ConfigMap
metadata:
  name: ckad04-config-multi-env-files-aecs
  namespace: default

16. For this question, please set the context to cluster3 by running:


kubectl config use-context cluster3



We have already deployed the required pods and services in the namespace ckad01-db-sec.



Create a new secret named ckad01-db-scrt-aecs with the data given below.


Secret Name: ckad01-db-scrt-aecs

Secret 1: DB_Host=sql01

Secret 2: DB_User=root

Secret 3: DB_Password=password123

Configure ckad01-mysql-server to load environment variables from the newly created secret, where the keys from the secret should become the environment variable name in the Pod.

Solution:

student-node ~ ➜  kubectl config use-context cluster3
Switched to context "cluster3".

student-node ~ ➜  k get all -n ckad01-db-sec
NAME                         READY   STATUS    RESTARTS   AGE
pod/ckad01-mysql-server   1/1     Running   0          3m13s
pod/ckad01-db-pod-aecs       1/1     Running   0          3m13s

NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/ckad01-webapp-service-aecs   NodePort    10.43.190.89    <none>        8080:30080/TCP   3m13s
service/ckad01-db-svc-aecs           ClusterIP   10.43.117.255   <none>        3306/TCP         3m13s

student-node ~ ➜  kubectl create secret generic ckad01-db-scrt-aecs \
   --namespace=ckad01-db-sec \
   --from-literal=DB_Host=sql01 \
   --from-literal=DB_User=root \
   --from-literal=DB_Password=password123
secret/ckad01-db-scrt-aecs created

student-node ~ ➜  k get -n ckad01-db-sec pod ckad01-mysql-server -o yaml > webapp-pod-sec-cfg.yaml

student-node ~ ➜  vim webapp-pod-sec-cfg.yaml

student-node ~ ➜  cat webapp-pod-sec-cfg.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: ckad01-mysql-server
  name: ckad01-mysql-server
  namespace: ckad01-db-sec
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: ckad01-db-scrt-aecs

student-node ~ ➜  kubectl replace -f webapp-pod-sec-cfg.yaml --force 
pod "ckad01-mysql-server" deleted
pod/ckad01-mysql-server replaced

student-node ~ ➜  kubectl exec -n ckad01-db-sec ckad01-mysql-server -- printenv | egrep -w 'DB_Password=password123|DB_User=root|DB_Host=sql01'
DB_Password=password123
DB_User=root
DB_Host=sql01


17. (ResourceQuata) For this question, please set the context to cluster2 by running:


kubectl config use-context cluster2



Create a ResourceQuota called ckad16-rqc in the namespace ckad16-rqc-ns and enforce a limit of one ResourceQuota for the namespace.

Solution:

student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  kubectl create namespace ckad16-rqc-ns
namespace/ckad16-rqc-ns created

student-node ~ ➜  cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ckad16-rqc
  namespace: ckad16-rqc-ns
spec:
  hard:
    resourcequotas: "1"
EOF

resourcequota/ckad16-rqc created

student-node ~ ➜  k get resourcequotas -n ckad16-rqc-ns
NAME              AGE   REQUEST               LIMIT
ckad16-rqc   20s   resourcequotas: 1/1


### SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE
19. For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1

Update the newly created pod simple-webapp-aom with a readinessProbe using the given specifications

Configure an HTTP readiness probe to the existing pod simple-webapp with path value set to /ready and port number to access container is 8080.

Solution:
Use the following YAML file and create a file - for example, simple-webapp-aom.yaml:

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-08-01T04:55:35Z"
  labels:
    name: simple-webapp
  name: simple-webapp-aom
  namespace: default
spec:
  containers:
  - env:
    - name: APP_START_DELAY
      value: "80"
    image: kodekloud/webapp-delayed-start
    imagePullPolicy: Always
    name: simple-webapp
    ports:
    - containerPort: 8080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080

To recreate the pod, run the command:

kubectl replace -f simple-webapp-aom.yaml --force

20. For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1

Pod manifest file is already given under the /root/ directory called ckad-pod-busybox.yaml.

There is error with manifest file correct the file and create resource.

我的配置: 


apiVersion: v1
kind: Pod
metadata:
  name: ckad-pod-busybox
spec:
  containers:
    - command: ["sleep 3600"]
      image: busybox
      name: pods-simple-container



Solution:
You will see following error:
student-node ~ ➜  kubectl create -f ckad-pod-busybox.yaml
Error from server (BadRequest): error when creating "ckad-pod-busybox.yaml": Pod in version "v1" cannot be handled as a Pod.

Use the following yaml file and create resource

apiVersion: v1
kind: Pod
metadata:
  name: ckad-pod-busybox
spec:
  containers:
    - command:
        - sleep
        - "3600"
      image: busybox
      name: pods-simple-container