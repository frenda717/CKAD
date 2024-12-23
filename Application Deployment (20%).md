Pod


1. Find where the pod belongs to which node: (2 ways)
   a. 
        
        kubectl get pods: check the column "Node"
   b. 
        
        kubectl get pods **-o wide**:

2. To see the details of a pod (eg. Image, Node...):
   kubectl get pods (Receive all pods name)
   
        kubectl describe pod (pod name 1) (pod name 2)

3. How many containers are part of the pod webapp?
    
        k get pods
    controlplane ~ ➜  k get pods
    NAME            READY   STATUS             RESTARTS   AGE
    newpods-dg5n6   1/1     Running            0          15m
    newpods-fm774   1/1     Running            0          15m
    newpods-ldhm4   1/1     Running            0          15m
    pod             1/1     Running            0          13m
    webapp          1/2     ImagePullBackOff   0          3m14s

        k describe pod webapp

    ...
    Containers:
    nginx:
        Container ID:   containerd://83af519b11589dc3f848b2d39b4d2169efc0d9245c663bda9806e3fa1e98eaad
        Image:          nginx
        Image ID:       docker.io/library/nginx@sha256:fb197595ebe76b9c0c14ab68159fd3c08bd067ec62300583543f0ebda353b5be
        Port:           <none>
        Host Port:      <none>
        State:          Running
        Started:      Mon, 23 Dec 2024 13:26:51 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ntvlq (ro)
    agentx:
        Container ID:   
        Image:          agentx
        Image ID:       
        Port:           <none>
        Host Port:      <none>
        State:          Waiting
        Reason:       ErrImagePull
        Ready:          False
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ntvlq (ro)
    Conditions:
        ....
        (Ignore)

    
    Answer: 2 (define 2 containers called nginx & agentx)

4. What does the READY column in the output of the kubectl get pods command indicate?
    controlplane ~ ➜  k get pods
    NAME            READY   STATUS             RESTARTS       AGE
    newpods-dg5n6   1/1     Running            1 (6m5s ago)   22m
    newpods-fm774   1/1     Running            1 (6m5s ago)   22m
    newpods-ldhm4   1/1     Running            1 (6m5s ago)   22m
    pod             1/1     Running            0              20m
    webapp          1/2     ImagePullBackOff   0              10m

    Answer: Running Containers in POD / Total Containers in POD`


5. Create a new pod with the name redis and with the image redis123: (2 ways)
   (Recommand to use a pod-definition YAML file.)
    a. create a YAML file (manualy, type: apiVersion, kinds, metadata and spec...)
        
        1. create a redis.yaml file, and type as below:
        vim redis.yaml

        apiVersion: v1
        kind: Pod
        metadata:
        name: redis (======>Make the pod name as redis)
        labels:
            app: myapp (define by yourself)
            type: front-end (define by yourself)

        spec:
        containers:
            - name: nginx-container (define by yourself)
            image: redis123 (======> and Make the pod imgage as redis123)

        kubectl create -f redis.yaml
        (pod/myapp-pod created)
    b. kubectl run redis -image=redis123 --**dry-run -o yaml**:

        kubectl run redis --image=redis123 --**dry-run=client -o yaml** > redis.yaml
        kubectl create -f redis.yaml

6. Now change the image on this pod to redis. Once done, the pod should be in a running state.
    a. manualy change the YAML file and then use apply command to update:
            
            kubectl apply -f redis.yaml
            kubectl get pods
    controlplane ~ ➜  k apply -f redis.yaml
    pod/redis configured

    b. directly edit the generated pod:
        
        kubectl edit pod redis (Modify the generated pod directly!!)
    controlplane ~ ➜  k edit pod redis
    pod/redis edited

    """

"""
A Note on Editing Existing Pods
In any of the practical quizzes, if you are asked to edit an existing POD, please note the following:

If you are given a pod definition file, edit that file and use it to create a new pod.

If you are not given a pod definition file, you may extract the definition to a file using the below command:

kubectl get pod <pod-name> -o yaml > pod-definition.yaml

Then edit the file to make the necessary changes, delete, and re-create the pod.

To modify the properties of the pod, you can utilize the kubectl edit pod <pod-name> command. Please note that only the properties listed below are editable.

spec.containers[*].image

spec.initContainers[*].image

spec.activeDeadlineSeconds

spec.tolerations

spec.terminationGracePeriodSeconds


"""


ReplicaSet

1. How many PODS are DESIRED in the new-replica-set?
   
        k get replicaset new-replica-set (= k get rs new-replica-set)
        k describe replicaset new-replica-set (= k describe rs new-replica-set)
    
    controlplane ~ ➜  k get rs new-replica-set 
    NAME              DESIRED   CURRENT   READY   AGE
    new-replica-set   4         4         0       2m9s

    controlplane ~ ➜  k describe rs new-replica-set 
    Name:         new-replica-set
    Namespace:    default
    Selector:     name=busybox-pod
    Labels:       <none>
    Annotations:  <none>
    Replicas:     4 current / 4 desired (======> To know how many pods are under this replicaset, check the "desired" number, 4)
    Pods Status:  0 Running / 4 Waiting / 0 Succeeded / 0 Failed
    Pod Template:
    Labels:  name=busybox-pod
    Containers:
    busybox-container:
        Image:      busybox777
        Port:       <none>
        Host Port:  <none>
        Command:
        sh
        -c
        echo Hello Kubernetes! && sleep 3600
        Environment:   <none>
        Mounts:        <none>
    Volumes:         <none>
    Node-Selectors:  <none>
    Tolerations:     <none>
    Events:
    ....
    (Ignore)

    Answer: 4 pods


2. How many PODs are READY in the new-replica-set?

    WROWN ANSWER:
        k describe rs new-replica-set (x)
        "describe" 有點像是用來查看該資源底下的子資源狀態，像是用k describe pod pod01 -> 可以用來查看含有幾個container

    ... 
    (Ignore)
    Events: (Check from the events is WRWON!!!!!)
    Type    Reason            Age    From                   Message
    ----    ------            ----   ----                   -------
    Normal  SuccessfulCreate  2m23s  replicaset-controller  Created pod: new-replica-set-59vnm
    Normal  SuccessfulCreate  2m23s  replicaset-controller  Created pod: new-replica-set-x5g7j
    Normal  SuccessfulCreate  2m23s  replicaset-controller  Created pod: new-replica-set-8jvkx
    Normal  SuccessfulCreate  2m23s  replicaset-controller  Created pod: new-replica-set-sjxt8

    Answer: 4 (WRWON!!!!)

    CORRECT ANSWER:

        k get rs new-replica-set
        (欲看該replicaset 本身的狀態，使用k get)

    controlplane ~ ✖ k get rs new-replica-set 
    NAME              DESIRED   CURRENT   READY   AGE
    new-replica-set   4         4         0       9m42s

    Answer: 0 (CORRECT!!!)

3. Why do you think the PODs are not ready?
    If the replicaset cannot run smoothly, check the inner pod status and also the replicaset configuration file 

        Check the configuration file
        k describe pods

    Events:
    Type     Reason     Age                 From               Message
    ----     ------     ----                ----               -------
    Normal   Scheduled  15m                 default-scheduler  Successfully assigned default/new-replica-set-x5g7j to controlplane
    Normal   Pulling    13m (x4 over 15m)   kubelet            Pulling image "busybox777"
    Warning  Failed     13m (x4 over 15m)   kubelet            Failed to pull image "busybox777": failed to pull and unpack image "docker.io/library/busybox777:latest": failed to resolve reference "docker.io/library/busybox777:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
    Warning  Failed     13m (x4 over 15m)   kubelet            Error: ErrImagePull
    Warning  Failed     13m (x6 over 15m)   kubelet            Error: ImagePullBackOff
    Normal   BackOff    10s (x64 over 15m)  kubelet            Back-off pulling image "busybox777"

    Answer: The image BUSYBOX777 doesn't exist


4. Fix the original replica set new-replica-set to use the correct busybox image.
   
        K edit rs new-replica-set
        k delete pod (pod name))
   controlplane ~ ➜  k edit rs new-replica-set 
    replicaset.apps/new-replica-set edited

    controlplane ~ ➜  k delete pods
    error: resource(s) were provided, but no name was specified

    controlplane ~ ✖ k get pods 
    NAME                    READY   STATUS             RESTARTS   AGE
    new-replica-set-4xmdm   0/1     ImagePullBackOff   0          13m
    new-replica-set-9m76c   0/1     ImagePullBackOff   0          13m
    new-replica-set-k6dkn   0/1     ImagePullBackOff   0          10m
    new-replica-set-qgszd   0/1     ImagePullBackOff   0          13m

    controlplane ~ ➜  k delete pod new-replica-set-*
    \Error from server (NotFound): pods "new-replica-set-*" not found

    controlplane ~ ✖ k delete pod new-replica-set-4xmdm new-replica-set-9m76c new-replica-set-k6dkn new-replica-set-qgszd 
    pod "new-replica-set-4xmdm" deleted
    pod "new-replica-set-9m76c" deleted
    pod "new-replica-set-k6dkn" deleted
    pod "new-replica-set-qgszd" deleted

    controlplane ~ ➜  k get pods
    NAME                    READY   STATUS    RESTARTS   AGE
    new-replica-set-229sm   1/1     Running   0          6s
    new-replica-set-565cf   1/1     Running   0          6s
    new-replica-set-qpjr6   1/1     Running   0          6s
    new-replica-set-z6g92   1/1     Running   0          6s

5. Scale the RepicaSet to 5 Pods:

    controlplane ~ ✖ kubectl scale --replicas=5 rs/new-replica-set 
    replicaset.apps/new-replica-set scaled
    controlplane ~ ➜ ** k scale --help**
    
    a. kubectl scale:

        k scale --replicas=5

    controlplane ~ ➜  kubectl scale --replicas=5 
    error: You must provide one or more resources by argument or filename.
    Example resource specifications include:
    '-f rsrc.yaml'
    '--filename=rsrc.json'
    '<resource> <name>'
    '<resource>'

        
    controlplane ~ ➜  kubectl scale replicaset new-replica-set --replicas=3 replicaset.apps/new-replica-set scaled

    b. kubectl edit replicaset new-replica-set, vi replica column 

        k edit rs new-replicaset


6. Now scale the ReplicaSet down to 2 PODs.


Use the kubectl scale command or edit the replicaset using kubectl edit replicaset.

controlplane ~ ➜  kubectl scale replicaset new-replica-set --replicas=2
replicaset.apps/new-replica-set scaled


6. Create a ReplicaSet using the replicaset-definition-1.yaml file located at /root/.
    controlplane ~ ➜  k create rs -f replicaset-definition-1.yaml 
    error: Unexpected args: [rs]
    See 'kubectl create -h' for help and examples

    Above shows that it should be something wrong in the yaml file. 
    vim replicaset-definition-1.yaml

    Run the command: You can check for apiVersion of replicaset by command 
    
        kubectl api-resources | grep replicaset
        
        kubectl explain replicaset | grep VERSION 
        
        and correct the apiVersion for ReplicaSet.

    Then run the command: kubectl create -f /root/replicaset-definition-1.yaml

    controlplane ~ ➜  vim  replicaset-definition-1.yaml 
    controlplane ~ ➜  kubectl explain replicaset | grep VERSION
    VERSION:    v1
    controlplane ~ ➜  kubectl create -f /root/replicaset-definition-1.yaml
    replicaset.apps/replicaset-1 created


Deployment

1. Create a new Deployment with the below attributes using your own deployment definition file.
    Name: httpd-frontend;
    Replicas: 3;
    Image: httpd:2.4-alpine

        k create -f deploy.yaml (new definition file)

    controlplane ~ ➜  ls
    deployment-definition-1.yaml  sample.yaml

    controlplane ~ ➜  cat deployment-definition-1.yaml > deploy.yaml

    controlplane ~ ➜  vim deploy.yaml 

    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: httpd-frontend
    spec:
    replicas: 3
    selector:
        matchLabels:
        name: httpd-frontend
    template:
        metadata:
        labels:
            name: httpd-frontend
        spec:
        containers:
        - name: httpd-frontend
            image: httpd:2.4-alpine
            command:
            - sh
            - "-c"
            - echo Hello Kubernetes! && sleep 3600
  
    controlplane ~ ➜  k create -f deploy.yaml 
    deployment.apps/httpd-frontend created

        k get deployment (or k get deploy)

    controlplane ~ ➜  k get deploy 
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    deployment-1     0/2     2            0           8m47s
    httpd-frontend   3/3     3            3           14s
    new-deployment   0/3     3            0           6m25s


Namespace
1. How many Namespaces exist on the system?
        
        k get ns

    controlplane ~ ✖ k get ns
    NAME              STATUS   AGE
    default           Active   6m47s
    dev               Active   3m46s
    finance           Active   3m46s
    kube-node-lease   Active   6m47s
    kube-public       Active   6m47s
    kube-system       Active   6m47s
    manufacturing     Active   3m46s
    marketing         Active   3m46s
    prod              Active   3m46s
    research          Active   3m46s

    Answer: 10

2. How many pods exist in the research namespace?

        k get pods --namespace=research

   controlplane ~ ✖ kubectl get pods --namespace=research
    NAME    READY   STATUS             RESTARTS       AGE
    dna-1   0/1     CrashLoopBackOff   6 (4m2s ago)   9m49s
    dna-2   0/1     CrashLoopBackOff   6 (4m2s ago)   9m49s

    Answer: 2

3. Create a POD in the finance namespace.
    Use the spec given below.
    Name: redis
    Image name: redis

        k run redis --image=redis -n finance

    controlplane ~ ➜  kubectl run redis --image=redis -n finance
    pod/redis created

    controlplane ~ ➜  kubectl run nginx  --image=nginx -n finance
    pod/nginx created

    controlplane ~ ➜  kubectl get pods --namespace=finance
    NAME      READY   STATUS    RESTARTS   AGE
    nginx     1/1     Running   0          10s
    payroll   1/1     Running   0          12m
    redis     1/1     Running   0          23s

4. Which namespace has the blue pod in it?
    
        k get pods --all-namespaces

    controlplane ~ ➜  k get pods
    No resources found in default namespace.

    controlplane ~ ➜  k get ns
    NAME              STATUS   AGE
    default           Active   18m
    dev               Active   15m
    finance           Active   15m
    kube-node-lease   Active   18m
    kube-public       Active   18m
    kube-system       Active   18m
    manufacturing     Active   15m
    marketing         Active   15m
    prod              Active   15m
    research          Active   15m

    controlplane ~ ➜  k get pods --all-namespaces
    NAMESPACE       NAME                                      READY   STATUS             RESTARTS        AGE
    dev             redis-db                                  1/1     Running            0               15m
    finance         nginx                                     1/1     Running            0               2m55s
    finance         payroll                                   1/1     Running            0               15m
    finance         redis                                     1/1     Running            0               3m8s
    kube-system     coredns-5dd589bf46-k7wmn                  1/1     Running            0               18m
    kube-system     helm-install-traefik-499js                0/1     Completed          1               18m
    kube-system     helm-install-traefik-crd-fbc2c            0/1     Completed          0               18m
    kube-system     local-path-provisioner-846b9dcb6c-bn6t4   1/1     Running            0               18m
    kube-system     metrics-server-5dc58b587c-grb7j           1/1     Running            0               18m
    kube-system     svclb-traefik-c05f8a9b-cq7sh              2/2     Running            0               18m
    kube-system     traefik-7f4b44bf74-jx9zz                  1/1     Running            0               18m
    manufacturing   red-app                                   1/1     Running            0               15m
    marketing       blue                                      1/1     Running            0               15m
    marketing       redis-db                                  1/1     Running            0               15m
    research        dna-1                                     0/1     CrashLoopBackOff   7 (4m38s ago)   15m
    research        dna-2                                     0/1     CrashLoopBackOff   7 (4m38s ago)   15m

    Answer: marketing

5. Access the Blue web application using the link above your terminal!!
From the UI you can ping other services.

6. What DNS name should the Blue application use to access the database db-service in its own namespace - marketing?
You can try it in the web application UI. Use port 6379.


Answer: db-service

7. What DNS name should the Blue application use to access the database db-service in the dev namespace?
You can try it in the web application UI. Use port 6379.

Ansewr: db-service.dev.svc.cluster.local