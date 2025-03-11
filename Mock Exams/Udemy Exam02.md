1. What are the valid way of creating a service in Kubernetes.
    a. k expose deployment my-deployment --port 80 --target-port 80 --name my-service  (yes)
    b. k create service my-test-service --tcp 80:80 (no)
    c. k run my-pod --image nginx --expose --port 80 (yes)
    d. k create service clusterip my-test-service --tcp 80:80

2. If you create a pod from below YAML, under which user container sec-ctx-container-2 processes will run
    apiVersion: v1
    kind: Pod
    metadata:
    name: security-context-example-q13
    spec:
    securityContext:
        runAsUser: 1000
    containers:
    - name: sec-ctx-container-1
        image: gcr.io/google-samples/node-hello:1.0
        securityContext:
        runAsUser: 2000
        allowPrivilegeEscalation: false
    - name: sec-ctx-container-2
        image: tomcat

    a. 2000
    b. 1000
    c. root

    Answer: b. 1000
    Explain:  If you see there is an user defined as the pod level and only for container sec-ctx-container-1, there is a further user defined at container level. For the container sec-ctx-container-2, there is no container level user. Hence, it wll take the user defined at the pod level.
3. If you create a job with below specification, what will be the state of the job after 5 mins?

    apiVersion: batch/v1
    kind: Job
    metadata:
    name: my-job1
    spec:
    backoffLimit: 4
    ttlSecondsAfterFinished: 100
    activeDeadlineSeconds: 15
    completions: 3
    parallelism: 1
    template:
        metadata:
        creationTimestamp: null
        spec:
        containers:
        - command:
            - /bin/sh
            - -c
            - sleep 10
            image: busybox
            name: my-job1
            resources: {}
        restartPolicy: Never

    a. The job will be DELETED from the cluster as per ttlSecondsAfterFinished setting which is set to 100 seconds (correct)
    b. The job will complete all the 3 completions and will remain in COMPLETED state

    Answer: a.
    Explain: There are many different configuration available for Job and CronJob in Kubernetes and it is important to understand them properly.

    In this case, the focus is on ttlSecondsAfterFinished, which defines the clean up time for a resource ( job here) and cluster will automatically delete the job after the time has reached. Hence, the job will be deleted in this scenario. It supersedes any other configuration.

    Please refer to below documentation for more details and example
    https://kubernetes.io/docs/concepts/workloads/controllers/job/#ttl-mechanism-for-finished-jobs

4. A pod is created with below specification. Please note the volume mount to the container. Once the pod is in running state, we create a file in the path /pod-dir with below command.

    kubectl exec test-pd -- touch /pod-dir/myfile.txt

    Question: If you delete the pod and recreate it with the same specification, will the file still be available to the new pod?

    apiVersion: v1
    kind: Pod
    metadata:
    name: test-pd
    spec:
    containers:
    - image: nginx
        name: test-container
        volumeMounts:
        - mountPath: /pod-dir
        name: test-volume
    volumes:
    - name: test-volume
        hostPath:
        path: /tmp/


    a. The file will persist in the node's mount path ( in this case /tmp) and any further pod mounting the same path can see the file (correct)
    b. The file will be remain in the node, however, as the file was created by a previous pod, it won't be visible to any subsequent pods

    Answer: a. 
    Explain: Hostpath is a persistent storage as this saves the file in the node's file system. Even if the pod is deleted, the files will be still present and any other pod mounted to the same path, can see the file.

    Hostpath is not a recommended way of using volume. It has limited usage. However, it is useful from practice point of view.


    In this case, you can ssh to the node and can see the file. If the node name is node01, use the command ssh node01 and then, do a ls on the directory /tmp as ls /tmp

    Read more about hostpath in the below link:

    https://kubernetes.io/docs/concepts/storage/volumes/#hostpath


5. From the options below, select the correct ways of assigning a pod to a node in a Kubernetes cluster.
    a. nodeName
    b. nodeAffinity
    c. nodeSelector
    d. taints and toerations

    Answer: abc
    Explain: This might sound confusing for some. These options are very close to each other. However, when it comes to choosing a node for a pod, taints and tolerations does not fit here. It is helpful for nodes to choose a pod, not pod choosing a node. Also, taints and tolerations does not ensure the pod with get the desired node. Other options makes it mandatory for cluster to assign the desired node to the pod

    Check below documentations. It properly defines the options.

    https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

6. If we create a service of type NodePort with below specification what will the result?
    a. The service will be created successfully, however it cannot expose any deployment as NodePort port definition (as 8080) is not in the permissible range (The range of valid ports is 30000-32767) for NodePort service
    b. The service will fail to create. It will give an error while creating the service as NodePort port definition (as 8080) is not in the permissible range (The range of valid ports is 30000-32767) for NodePort service (correct)

    Answer: b.
    Explain: The port for the node, which is the nodePort attribute in the service definition can only stay in the range of 30000-32767. Kubernetes will not allow you to create a NodePort type service with nodePort outside this range. It will throw below error, if you try to create a service from above specification:

    The Service "my-service1" is invalid: spec.ports[0].nodePort: Invalid value: 8080: provided port is not in the valid range. The range of valid ports is 30000-32767


    If you are not sure of what port to choose while creating a NodePort type service, better leave that attribute. Kubernetes will allocate a port for you in the permissible range

7. A pod is created with below specification. It has 2 containers and there is a emptyDir volume mount to both the containers.

    Once the pod is created, you create a file inside container 1 with below command.

    kubectl exec test-pd -c container1  -- touch /cache/test.txt

    Please note the file is created inside the volume mount of container1

    Question - From the options below, which is correct with respect to the visibility of the file new file to container2

    apiVersion: v1
    kind: Pod
    metadata:
    name: test-pd
    spec:
    containers:
    - image: tomcat
        name: container1
        volumeMounts:
        - mountPath: /cache
        name: cache-volume
    - image: nginx
        name: container2
        volumeMounts:
        - mountPath: /temp
        name: cache-volume
    volumes:
    - name: cache-volume
        emptyDir:
        sizeLimit: 500Mi

    a. The file test.txt is visible to container2 in the location /temp (correct)
    b. The file test.txt is visible to container2 in the location /cache

    Answer: a.
    Explain: Volume is a section where you can expect many questions and it might be tricky sometimes. There are many options and you will have to know few important one. From the exam perspective, the most important one is emptyDir. You can read more about it in the below documentation:

    https://kubernetes.io/docs/concepts/storage/volumes/#emptydir

    Coming to the question, if you notice the volume mount is same for both the containers. However, they are mounted into different path. Whatever the path they are mounted to, because it is the same volume, container2 will always have a visibility to the file created in that path by container1. The only difference will be the mount path. As for container2, it is mounted to /temp path, the file will be visible in that path. Please see below the result:

    controlplane $ kubectl exec -it test-pd -c container2 -- ls /temp
    test.txt

8. In  a Kubernetes cluster, we have a number of pods running with below labels.

    Question - From the options below, select the correct one which will list pods with label EITHER color=red OR color=yellow

    The output should look like below:

    a. kubectl get pods -l 'color in (red,yellow)' --show-labels (correct)
    b. kubectl get pods -l color=red -l color=yellow --show-labels

    Answer: a.
    Explain: Kuberletes -l or --selector option supports in , notin type of filtering conditions. This is very helpful with situations like this. Below link gives you details of this with example:

    https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#resources-that-support-set-based-requirements

    Try to play around with your own set of conditions to understand it in a better way. This will be helpful during the exam.
9. From the below attributes for a job, which one is used to delete a job from the cluster after a specified time period
    a. activeDeaelineSeconds
    b. ttlSecondsAfterFinished (correct)

    The correct answer is ttlSecondsAfterFinished. It is the attribute to clean up job from the cluster after a specified time period. When the TTL controller cleans up the Job, it will delete the Job cascadingly, i.e. delete its dependent objects, such as Pods, together with the Job.

    Questions on job might be tricky during exam as there are many configurations available to Job and CronJob. please read the documentation properly for this.

    Read more about it in the below link:

    https://kubernetes.io/docs/concepts/workloads/controllers/job/#ttl-mechanism-for-finished-jobs

10. In this question, we will create a config map and a pod using the config map as per below command and pod specification.

Config map created imperatively with kubectl create command as below

kubectl create cm config-map1 --from-literal db-host-name=app-db-host --from-literal db-port-num=6289



Pod created declaratively with below definition

apiVersion: v1
kind: Pod
metadata:
  labels:
    exam: ckad
  name: pod1
spec:
  containers:
  - name: nginx
    image: nginx
    env: 
    - name: DB-HOST-NAME
      valueFrom:
         configMapKeyRef:
           name: config-map1
           key: app-db-host
    - name: DB-PORT
      valueFrom:
         configMapKeyRef:
           name: config-map1
           key: db-port
  dnsPolicy: ClusterFirst
  restartPolicy: Always


Question - What will happen when you try to create the pod with above definition? Choose the correct option.

a. The pod will give error while you try to create it.
b. The pod will be created, however it wont come to running state. The pod status will be CreateContainerConfigError (correct)

The config will be created successfully. However, there is a small problem with the pod definition. If you notice the env declaration and particularly the one with name DB-HOST-NAME, the key referenced here is not a key in the configmap declation.

The config map has key as db-host-name, however, here the key referenced is app-db-host, which is a value in the config map.

So, in this case, the pod will be created, but won't come to the running state. it will come to CreateContainerConfigError state.

After the pod is created, you can see the reason in the pod description as below:

Warning  Failed     9m2s (x8 over 10m)  kubelet            Error: couldn't find key app-db-host in ConfigMap default/config-map1

11. We have a network policy in a cluster as per below definition



apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy-1
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress


Choose from options below the correct usage of above network policy

This will deny all the ingress and egress traffic in the namespace where the policy is created (correct)

This will allow all the ingress and egress traffic in the cluster where the policy is created

Below link explains the default policy.

https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-deny-all-ingress-and-all-egress-traffic


12. In a namespace of a cluster we have below deny all ingress network policy applied.



apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy-1
spec:
  podSelector: {}
  policyTypes:
  - Ingress


We have 3 pods in the namespace. The requirement is to reach the "target-pod" from both the source pods.



Target pod specification:



apiVersion: v1
kind: Pod
metadata:
  labels:
    run: target-pod
  name: target-pod
spec:
  containers:
  - image: nginx
    name: target-con
    ports:
    - containerPort: 80


Source Pods:



apiVersion: v1
kind: Pod
metadata:
  labels:
    run: source-pod-1
  name: source-pod-1
spec:
  containers:
  - image: tomcat
    name: source-con-1


apiVersion: v1
kind: Pod
metadata:
  labels:
    run: source-pod-2
  name: source-pod-2
spec:
  containers:
  - image: tomcat
    name: source-con-2


From the options below, choose the correct network policy to allow traffic from both source pods to target pod.

a. (correct)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-allow-specific
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: target-pod
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              run: source-pod-1
        - podSelector:
            matchLabels:
              run: source-pod-2
      ports:
        - protocol: TCP
          port: 80
b. (wrong) 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-allow-specific
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: target-pod
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
          - matchLabels:
              run: source-pod-1
          - matchLabels:
              run: source-pod-2
      ports:
        - protocol: TCP
          port: 80


Network policies can be tricky and it is very important to understand the use of AND / OR in the podSelector option. If it is defined as multiple items in the array, it is OR. If multiple items defined in a single element of array, it will be AND.

Let us understand with an example:

 ingress:
    - from:
        - podSelector:
            matchLabels:
              run: source-pod-1
        - podSelector:
            matchLabels:
              run: source-pod-2

In the above example, podSelector defined as 2 different elements of array. Hence it will be pods with label run=source-pod-1 OR  pods with label run=source-pod-2 will be allowed to the target pod.



ingress:
    - from:
        - podSelector:
            matchLabels:
              run: source-pod-1
          podSelector:
            matchLabels:
              run: source-pod-2


In the second case, it is 2 items under a single element of array, it will be a pod with both the label in it will be allowed. That is with label run=source-pod-1 AND  pods with label run=source-pod-2


In a Kubernetes cluster, we have 2 namespaces with name service-ns and client-ns. Below resources running in each of this namespaces.

namespace : service-ns

controlplane $ kubectl get all -n service-ns 
NAME              READY   STATUS    RESTARTS   AGE
pod/service-app   1/1     Running   0          10m
 
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/service-app-svc   ClusterIP   10.108.19.147   <none>        80/TCP    10m


namespace : client-ns

controlplane $ kubectl get all -n client-ns  
NAME             READY   STATUS    RESTARTS   AGE
pod/client-app   1/1     Running   0          10m


If we want to reach the service running in the service-ns from the pod running in the client-ns, what will be the correct dns for the service? choose the correct one from the options below:



Assumption: cluster domain name is  - cluster.local

a. service-app.service-ns.svc.cluster.local
b. service-app-svc.service-ns.svc.cluster.local (correct)

Hence, it will be option 1 the correct answer in this case.


https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/


14. In a Kubernetes cluster, we have a pod running in the default namespace with below specification:



apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-app
  name: my-app
spec:
  containers:
  - image: nginx
    name: my-app
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always


If you do a kubectl get pods -o wide, below is the output.



controlplane $ k get pods -o wide
NAME     READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
 
my-app   1/1     Running   0          4m57s   192.168.1.4   node01   <none>           <none>


Now, if you want to access this pod from another pod in the same namespace, using curl, which of options below will work to address the my-app pod?



a. curl my-app (x)
b. curl 192.168.1.4 (O)
c. curl 192-168-1-4.default.pod.cluster.local (O)
d. curl 192.168.1.4.default.pod.cluster.local (x)

Answer: b.c 

There is an option in the pod declaration where you can set the pod hostname and subdomain while creating the pod, like shown below:



spec:
  hostname: my-pod-hostname
  subdomain: my-subdomain
If defines the pod FQDN as

my-pod-hostname.my-subdomain.<my-namespace>.svc.<cluster-domain>.example

Alternatively, if you want to access the pod with IP address DNS, it works as below:

if a Pod in the default namespace has the IP address 172.17.0.3, and the domain name for your cluster is cluster.local, then the Pod has a DNS name:

172-17-0-3.default.pod.cluster.local

Please note the dots in the IP address converts to dash (-)

Again, the Pod can be accessed by only IP address as well. just like 172.17.0.3. However IP address are ephemeral. Hence accessing via service is always suggested

15. In a Kubernetes cluster, we have a pod already running with below configuration:



apiVersion: v1
kind: Pod
metadata:
  labels:
    app: frontend
  name: frontend-pod
spec:
  containers:
  - image: nginx
    name: frontend-pod


Now if we create a deployment, with below specification, how many new pods will be created? Please note the labels of the above pod and the match label condition in the deployment.



apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - image: nginx
        name: nginx
2 new pods as the already running pod match condition for 1 replica

Deployment will fail as there is already another type of resource running with the same label

a. 3 new pods with the existing pod running as it is (correct)
b. Kuberenetes will delete the existing pod and will create 3 new fresh pods

This is interesting. because you will see 4 pods running with the same specification. 3 under the deployment and the independent pod.



Deployment will maintain its own set of pods and will append one extra label to the pods, though they wont be visible as YAML output.



Now if you create a replicaset with similar specification as the above deployment, it will create only 2 new pods. Replicaset checks the cluster for any existing matching pod labels. try below replicaset specification with the above pod created



apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - image: nginx
        name: nginx

        

controlplane ~ ✖ kubectl exec -it nginx1401 -n dev1401 -- netstat -tulnp 80
error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "563a9c4968f3f4ecf53ab69f9ccc20da1b4770564ce417f4ed1c0436059126ad": OCI runtime exec failed: exec failed: unable to start container process: exec: "netstat": executable file not found in $PATH: unknown

controlplane ~ ✖ kubectl exec -it nginx1401 -n dev1401 -- "netstat -tulnp 80"
error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "c73db106d4d259b997bcb4a2a0026d9e90221293d64c6b18db61cb16bc38fd1b": OCI runtime exec failed: exec failed: unable to start container process: exec: "netstat -tulnp 80": executable file not found in $PATH: unknown

controlplane ~ ✖ ^[[200~kubectl exec -it nginx1401 -n dev1401 -- ss -tulnp
-bash: $'\E[200~kubectl': command not found

controlplane ~ ✖ kubectl exec -it nginx1401 -n dev1401 -- ss -tulnp
error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "b88eac33ea71316d292343cc8130f7f5e33a2c701c8373ab4d080d86c89d4c8a": OCI runtime exec failed: exec failed: unable to start container process: exec: "ss": executable file not found in $PATH: unknown

controlplane ~ ✖ kubectl exec -it nginx1401 -n dev1401 -- curl -I http://localhost:8080
kubectl exec -it nginx1401 -n dev1401 -- curl -I http://localhost:9080
error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "5acfc1f32e67e486ea052d734dfd9193e04605c3e0dffb51a0139c15850c5af9": OCI runtime exec failed: exec failed: unable to start container process: exec: "curl": executable file not found in $PATH: unknown
error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "0e5c3f7483c292a2e10969c4ee804ec98da71ce5c8af43d81f3d5ab325b2b37a": OCI runtime exec failed: exec failed: unable to start container process: exec: "curl": executable file not found in $PATH: unknown

controlplane ~ ✖ kubectl exec -it nginx1401 -n dev1401 -- curl -I http://localhost:9080
error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "301b24db8986c48eaf44298647bb85139264c8a1b122f8527cda0d74125fd3ba": OCI runtime exec failed: exec failed: unable to start container process: exec: "curl": executable file not found in $PATH: unknown

controlplane ~ ✖ kubectl exec -it nginx1401 -n dev1401 -- curl -I http://localhost:8080
error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "ae2eaad09b6b6740aa669804cd0de1e940e4f8169c06bc8c640661015b902736": OCI runtime exec failed: exec failed: unable to start container process: exec: "curl": executable file not found in $PATH: unknown


controlplane ~ ✖ kubectl exec -it nginx1401 -n dev1401 -- sh
# apt update && apt install -y net-tools iproute2 curl
Get:1 http://security.debian.org/debian-security buster/updates InRelease [34.8 kB]
Get:2 http://deb.debian.org/debian buster InRelease [122 kB]
Get:3 http://deb.debian.org/debian buster-updates InRelease [56.6 kB]
Get:4 http://security.debian.org/debian-security buster/updates/main amd64 Packages [610 kB]
Get:5 http://deb.debian.org/debian buster/main amd64 Packages [7909 kB]
Get:6 http://deb.debian.org/debian buster-updates/main amd64 Packages [8788 B]
Fetched 8741 kB in 2s (4354 kB/s)                         