First Attempt Score:  32%

空題 (來不及寫): 12,14,17,19 
錯題: 1,2,5,7,8,11,13



### SECTION: APPLICATION DESIGN AND BUILD
1. Create a persistent volume called data-pv-ckad02-str with the below properties:
   - Its capacity should be 128Mi.
   - The volume type should be hostpath and path should be /opt/data-pv-ckad02-str.

   Next, create a persistent volume claim called data-pvc-ckad02-str as per below properties:
   - Request 50Mi of storage from data-pv-ckad02-str PV.

   For this question, please set the context to cluster1 by running:
   kubectl config use-context cluster1
  
    student-node ~ ➜  kr data-pv-ckad02-str.yaml --force
    persistentvolume "data-pv-ckad02-str" deleted
    persistentvolume/data-pv-ckad02-str replaced

    student-node ~ ➜  k get pv
    NAME                 CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
    data-pv-ckad02-str   128Mi      RWO            Delete           Available                          <unset>                          4s

    student-node ~ ➜  kr data-pcv-ckad02-str.yaml --force
    persistentvolumeclaim "data-pvc-ckad02-str" deleted
    persistentvolumeclaim/data-pvc-ckad02-str replaced

    student-node ~ ➜  k get pvc
    NAME                  STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
    data-pvc-ckad02-str   Pending                                      local-path     <unset>                 3s

    student-node ~ ➜  vim data-pcv-ckad02-str.yaml 

    student-node ~ ➜  kr data-pcv-ckad02-str.yaml --force
    persistentvolumeclaim "data-pvc-ckad02-str" deleted
    persistentvolumeclaim/data-pvc-ckad02-str replaced

    student-node ~ ➜  k get pvc
    NAME                  STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
    data-pvc-ckad02-str   Pending                                      local-path     <unset>                 3s


    student-node ~ ➜  k get pvc
    NAME                  STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
    data-pvc-ckad02-str   Pending                                      local-path     <unset>                 3s

    student-node ~ ➜  k delete pvc data-pvc-ckad02-str --force
    Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
    persistentvolumeclaim "data-pvc-ckad02-str" force deleted

    student-node ~ ➜  k get pvc
    No resources found in default namespace.

    student-node ~ ➜  kc data-pcv-ckad02-str.yaml 
    persistentvolumeclaim/data-pvc-ckad02-str created

    student-node ~ ➜  k get pvc
    NAME                  STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
    data-pvc-ckad02-str   Pending                                      **local-path**     <unset>                 3s


    Solution:
    Use below YAML to create desired Kubernetes objects:


    apiVersion: v1
    kind: PersistentVolume
    metadata:
    name: data-pv-ckad02-str
    spec:
    capacity:
        storage: 128Mi
    accessModes:
        - ReadWriteOnce
    hostPath:
        path: /opt/data-pv-ckad02-str
    storageClassName: manual

    ---

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
    name: data-pvc-ckad02-str
    spec:
    storageClassName: manual
    volumeName: data-pv-ckad02-str
    accessModes:
        - ReadWriteOnce
    resources:
        requests:
        storage: 50Mi


2. (CronoJob: command設成了arg!) In the ckad-job namespace, create a cron job called my-alarm that prints current datetime. It must be scheduled to run every Sunday at midnight.
    In case the container in pod failed for any reason, it should be restarted automatically.
    Use busybox:1.28 image to create job.

    Sample output:
    Sun Mar  12 00:00:00 UTC 2023
    For this question, please set the context to cluster1 by running
    kubectl config use-context cluster1

    student-node ~ ➜  kd create cronjob my-alarm -n ckad-job --image=busybox:1.28 --schedule="0 0 * * sun" -o yaml > my-alarm.yaml

    student-node ~ ➜  vim my-alarm.yaml 

    student-node ~ ➜  kc my-alarm.yaml 
    cronjob.batch/my-alarm created

    student-node ~ ➜  k get cronjob -n ckad-job 
    NAME       SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
    my-alarm   0 0 * * sun   False     0        <none>          6s

    student-node ~ ➜  k describe cronjobs.batch -n ckad-job 
    Name:                          my-alarm
    Namespace:                     ckad-job
    Labels:                        <none>
    Annotations:                   <none>
    Schedule:                      0 0 * * sun
    Concurrency Policy:            Allow
    Suspend:                       False
    Successful Job History Limit:  3
    Failed Job History Limit:      1
    Starting Deadline Seconds:     <unset>
    Selector:                      <unset>
    Parallelism:                   <unset>
    Completions:                   <unset>
    Pod Template:
    Labels:  <none>
    Containers:
    my-alarm:
        Image:      busybox:1.28
        Port:       <none>
        Host Port:  <none>
        ~~Args~~:
        /bin/sh
        -c
        echo $(date)
        Environment:     <none>
        Mounts:          <none>
    Volumes:           <none>
    Node-Selectors:    <none>
    Tolerations:       <none>
    Last Schedule Time:  <unset>
    Active Jobs:         <none>
    Events:              <none>

    student-node ~ ➜  cat my-alarm.yaml 
    apiVersion: batch/v1
    kind: CronJob
    metadata:
    creationTimestamp: null
    name: my-alarm
    namespace: ckad-job
    spec:
    jobTemplate:
        metadata:
        creationTimestamp: null
        name: my-alarm
        spec:
        template:
            metadata:
            creationTimestamp: null
            spec:
            containers:
            - image: busybox:1.28
                name: my-alarm
                args: [/bin/sh, -c, echo $(date)]
                resources: {}
            restartPolicy: OnFailure
    schedule: 0 0 * * sun
    status: {}

    
    O Is cronjob my-alarm created?
    O Is the container image busybox:1.28?
    X Does cronjob run the date command?
    O Does cronjob runs every Sunday at midnight?

    Solution: Use below YAML to create cronjob:

    apiVersion: batch/v1
    kind: CronJob
    metadata:
    name: my-alarm
    namespace: ckad-job
    spec:
    schedule: "0 0 * * 0"
    jobTemplate:
        spec:
        template:
            spec:
            containers:
            - name: hello
                image: busybox:1.28
                imagePullPolicy: IfNotPresent
                command:
                - /bin/sh
                - -c
                - date;
            restartPolicy: OnFailure

    


4. (正確) In the ckad-multi-containers namespace, create a pod named static-web-server, which consists of 2 containers. One main container and one init-container both are running alpine image.

    Init container should print this message Getting main application ready! and then sleep for 10 seconds.
    Main container should print this message Main application is running and then sleep for 3600 seconds.


    student-node ~ ➜  cat static-web-server.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: static-web-server
    name: static-web-server
    namespace: ckad-multi-containers
    spec:
    initContainers:
    - image: alpine 
        name: init-container
        command: ["/bin/sh","-c",echo "Getting main application ready!", "sleep 10"]
        resources: {}
    containers:
    - image: alpine
        name: main-container
        command: ["/bin/sh","-c",echo "Main application is running", "sleep 3600"]
        resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Never
    status: {}


    student-node ~ ➜  kr static-web-server.yaml --force
    pod/static-web-server replaced

    student-node ~ ➜  k describe pod -n ckad-multi-containers 
    Name:             static-web-server
    Namespace:        ckad-multi-containers
    Priority:         0
    Service Account:  default
    Node:             cluster2-node01/192.168.81.167
    Start Time:       Mon, 10 Mar 2025 01:12:49 +0000
    Labels:           run=static-web-server
    Annotations:      <none>
    Status:           Succeeded
    IP:               10.42.1.6
    IPs:
    IP:  10.42.1.6
    Init Containers:
    init-container:
        Container ID:  containerd://c1c694e677ad4a15d090aed70a8a0d33bc4ea9b1ad1d6aba9c186dd10f13ee43
        Image:         alpine
        Image ID:      docker.io/library/alpine@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
        Port:          <none>
        Host Port:     <none>
        Command:
        /bin/sh
        -c
        echo "Getting main application ready!"
        sleep 10
        State:          Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Mon, 10 Mar 2025 01:12:50 +0000
        Finished:     Mon, 10 Mar 2025 01:12:50 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4ntqq (ro)
    Containers:
    main-container:
        Container ID:  containerd://cd804dbac70491a780a4c68804266e8c3816395465a6f1a8f0d344c00fa0dc1a
        Image:         alpine
        Image ID:      docker.io/library/alpine@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
        Port:          <none>
        Host Port:     <none>
        Command:
        /bin/sh
        -c
        echo "Main application is running"
        sleep 3600
        State:          Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Mon, 10 Mar 2025 01:12:51 +0000
        Finished:     Mon, 10 Mar 2025 01:12:51 +0000
        Ready:          False
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4ntqq (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   False 
    Initialized                 True 
    Ready                       False 
    ContainersReady             False 
    PodScheduled                True 
    Volumes:
    kube-api-access-4ntqq:
        Type:                    Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds:  3607
        ConfigMapName:           kube-root-ca.crt
        ConfigMapOptional:       <nil>
        DownwardAPI:             true
    QoS Class:                   BestEffort
    Node-Selectors:              <none>
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
    Type    Reason     Age   From               Message
    ----    ------     ----  ----               -------
    Normal  Scheduled  5s    default-scheduler  Successfully assigned ckad-multi-containers/static-web-server to cluster2-node01
    Normal  Pulling    4s    kubelet            Pulling image "alpine"
    Normal  Pulled     4s    kubelet            Successfully pulled image "alpine" in 161ms (161ms including waiting)
    Normal  Created    4s    kubelet            Created container init-container
    Normal  Started    4s    kubelet            Started container init-container
    Normal  Pulling    4s    kubelet            Pulling image "alpine"
    Normal  Pulled     4s    kubelet            Successfully pulled image "alpine" in 166ms (166ms including waiting)
    Normal  Created    4s    kubelet            Created container main-container
    Normal  Started    3s    kubelet            Started container main-container

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                READY   STATUS      RESTARTS   AGE
    static-web-server   0/1     Completed   0          10s

5. In the ckad-multi-containers namespace, create a pod named static-web-server, which consists of 2 containers. One main container and one init-container both are running alpine image.

    Init container should print this message Getting main application ready! and then sleep for 10 seconds.
    Main container should print this message Main application is running and then sleep for 3600 seconds.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    我的配置:

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                READY   STATUS      RESTARTS   AGE
    static-web-server   0/1     Completed   0          94m

    student-node ~ ➜  cat static-web-server.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: static-web-server
    name: static-web-server
    namespace: ckad-multi-containers
    spec:
    initContainers:
    - image: alpine 
        name: init-container
        command: ["/bin/sh","-c",echo "Getting main application ready!", "sleep 10"]
        resources: {}
    containers:
    - image: alpine
        name: main-container
        command: ["/bin/sh","-c",echo "Main application is running", "sleep 3600"]
        resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Never
    status: {}



    Solution:
    Use below YAML to create required pod:


    apiVersion: v1
    kind: Pod
    metadata:
    name: static-web-server
    namespace: ckad-multi-containers
    labels:
        app.kubernetes.io/name: static-web-server
    spec:
    containers:
        - name: server-container
        image: alpine
        command:
            - sh
            - -c
            - echo Main application is running && sleep 3600
    initContainers:
        - name: init-myservice
        image: alpine
        command:
            - sh
            - -c
            - echo Getting main application ready! && sleep 10

### SECTION: APPLICATION DEPLOYMENT
6. (helm 正確) Our new client wants to deploy the resources through the popular Helm tool. In the initial phase, our team lead wants to deploy nginx, a very powerful and versatile web server software that is widely used to serve static content, reverse proxying, load balancing, from the bitnami helm chart on the cluster3-controlplane node.

    The chart URL and other specifications are as follows: -

    1. The chart URL link - https://charts.bitnami.com/bitnami
    2. The chart repository name should be polar.
    3. The release name should be nginx-server.
    4. All the resources should be deployed on the cd-tool-apd namespace.

    NOTE: - You have to perform this task from the student-node.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    student-node ~ ➜  helm search repo polar|grep nginx
    polar/nginx                                     19.0.1          1.27.4          NGINX Open Source is a web server that can be a...
    polar/nginx-ingress-controller                  11.6.11         1.12.0          NGINX Ingress Controller is an Ingress controll...
    polar/nginx-intel                               2.1.15          0.4.9           DEPRECATED NGINX Open Source for Intel is a lig...

    student-node ~ ➜  helm search repo polar|grep nginx
    polar/nginx                                     19.0.1          1.27.4          NGINX Open Source is a web server that can be a...
    polar/nginx-ingress-controller                  11.6.11         1.12.0          NGINX Ingress Controller is an Ingress controll...
    polar/nginx-intel                               2.1.15          0.4.9           DEPRECATED NGINX Open Source for Intel is a lig...


    student-node ~ ➜  helm install nginx-server polar/nginx  -n cd-tool-apd
    Error: INSTALLATION FAILED: create: failed to create: namespaces "cd-tool-apd" not found

    student-node ~ ✖ k create ns cd-tool-apd
    namespace/cd-tool-apd created

    student-node ~ ➜  helm install nginx-server polar/nginx  -n cd-tool-apd
    NAME: nginx-server
    LAST DEPLOYED: Mon Mar 10 01:22:05 2025
    NAMESPACE: cd-tool-apd
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    CHART NAME: nginx
    CHART VERSION: 19.0.1
    APP VERSION: 1.27.4
    ...

    student-node ~ ➜  helm list -n cd-tool-apd
    NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART              APP VERSION
    nginx-server    cd-tool-apd     1               2025-03-10 01:22:05.654528469 +0000 UTC deployed        nginx-19.0.1       1.27.4     

7. One of the Kubernetes developers from the team has deployed a test application named test-v1-apd to implement the blue/green deployment methodology. 

    Now, the team wants to send 50% of the traffic to the new test application, but since the main developer is away from the office, the team wants you to create a new deployment that can take the traffic from the pre-configured service. We have details of the application, which will help you create a new deployment: -

    The new deployment name should be test-v2-apd.
    Add label to a deployment stage = test02. 
    Use the image kodekloud/webapp-color:v2 and the container name test-v2.
    Scale a deployment in a way that the pre-configured service can send 50% of the traffic to this new deployment.

    NOTE: - We do not need to modify the test-v1-apd deployment.

    You can test the application from the terminal by running the curl command with the following syntax: -
    curl http://cluster3-controlplane:NODE-PORT

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  k get svc
    NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    kubernetes     ClusterIP   172.20.0.1      <none>        443/TCP          111m
    test-apd-svc   NodePort    172.20.243.73   <none>        8080:30155/TCP   25s

    student-node ~ ➜  k get deploy
    NAME          READY   UP-TO-DATE   AVAILABLE   AGE
    test-v1-apd   3/3     3            3           28s

    student-node ~ ➜  k describe deploy test-v1-apd 
    Name:                   test-v1-apd
    Namespace:              default
    CreationTimestamp:      Mon, 10 Mar 2025 01:24:17 +0000
    Labels:                 stage=test01
                            test-app=v1
    Annotations:            deployment.kubernetes.io/revision: 1
    Selector:               test-app=v1
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
    Labels:  test-app=v1
    Containers:
    test-v1:
        Image:         kodekloud/webapp-color:v1
        Port:          <none>
        Host Port:     <none>
        Environment:   <none>
        Mounts:        <none>
    Volumes:         <none>
    Node-Selectors:  <none>
    Tolerations:     <none>
    Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   test-v1-apd-668fc5b564 (3/3 replicas created)
    Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  38s   deployment-controller  Scaled up replica set test-v1-apd-668fc5b564 to 3

    student-node ~ ✖ kd create deploy test-v2-apd --image=kodekloud/webapp-color:v2 -o yaml > test-v2-apd.yaml

    student-node ~ ➜  vim test-v2-apd.yaml 
    student-node ~ ➜  cat test-v2-apd.yaml 
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    creationTimestamp: null
    labels:
        test-app: v2
        app: test-v2-apd
    name: test-v2-apd
    spec:
    replicas: 2
    selector:
        matchLabels:
            test-app: v2
    strategy: {}
    template:
        metadata:
        creationTimestamp: null
        labels:
            test-app: v2
        spec:
        containers:
        - image: kodekloud/webapp-color:v2
            name: webapp-color
            resources: {}
    status: {}


    student-node ~ ➜  k get deploy
    NAME          READY   UP-TO-DATE   AVAILABLE   AGE
    test-v1-apd   3/3     3            3           19m
    test-v2-apd   2/2     2            2           6m17s

    student-node ~ ➜  k describe deploy test-v2-apd 
    Name:                   test-v2-apd
    Namespace:              default
    CreationTimestamp:      Mon, 10 Mar 2025 01:37:22 +0000
    Labels:                 app=test-v2-apd
                            test-app=v2
    Annotations:            deployment.kubernetes.io/revision: 1
    Selector:               test-app=v2
    Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
    Labels:  test-app=v2
    Containers:
    webapp-color:
        Image:         kodekloud/webapp-color:v2
        Port:          <none>
        Host Port:     <none>
        Environment:   <none>
        Mounts:        <none>
    Volumes:         <none>
    Node-Selectors:  <none>
    Tolerations:     <none>
    Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   test-v2-apd-867c589fcf (2/2 replicas created)
    Events:
    Type    Reason             Age    From                   Message
    ----    ------             ----   ----                   -------
    Normal  ScalingReplicaSet  6m25s  deployment-controller  Scaled up replica set test-v2-apd-867c589fcf to 2

    student-node ~ ➜  k get svc
    NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    kubernetes     ClusterIP   172.20.0.1      <none>        443/TCP          131m
    test-apd-svc   NodePort    172.20.243.73   <none>        8080:30155/TCP   19m

    student-node ~ ➜  k describe svc test-apd-svc
    Name:                     test-apd-svc
    Namespace:                default
    Labels:                   tag=test-apd-svc
    Annotations:              <none>
    Selector:                 test-app=v2
    Type:                     NodePort
    IP Family Policy:         SingleStack
    IP Families:              IPv4
    IP:                       172.20.243.73
    IPs:                      172.20.243.73
    Port:                     <unset>  8080/TCP
    TargetPort:               8080/TCP
    NodePort:                 <unset>  30155/TCP
    Endpoints:                172.17.1.8:8080,172.17.1.9:8080
    Session Affinity:         None
    External Traffic Policy:  Cluster
    Events:                   <none>


    Solution
    Run the following command to change the context: -


    kubectl config use-context cluster3



    In this task, we will use the kubectl command. Here are the steps: -



    Use the kubectl get command to list all the given resources: -

    kubectl get deploy,svc -A



    Here -A option lists the resources of all the namespaces.

    Identify the deployment and service names.

    Now, use the kubectl create command to create a deployment manifest file as follows: -


    kubectl create deploy test-v2-apd --image=kodekloud/webapp-color:v2 --dry-run=client -oyaml > <FILE-NAME-1>.yaml



    Open the file with any text editor such as vi or nano and update it as per the specifications. Check the labels for the pod template from the service. It should look like this: -

    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    labels:
        stage: test02
    name: test-v2-apd
    spec:
    replicas: 3
    selector:
        matchLabels:
        test-app: v1
    template:
        metadata:
        labels:
            test-app: v1
        spec:
        containers:
        - image: kodekloud/webapp-color:v2
            name: test-v2



    We got the instructions not to modify the test-v1-apd deployment, we must add the replica count to 3 for the test-v2-apd deployment. So we will deploy a total of 6 application pods for both deployments.

    Since the service distributes traffic to all pods equally, we have to set the replica count 3 to the test-v2-apd deployment so that the given service will send 50% traffic to the deployment pods.

    Now, create a deployment by using the kubectl create -f command: -

    kubectl create -f <FILE-NAME-1>.yaml




8. On the cluster2, the team has installed multiple helm charts on a different namespace. By mistake, those deployed resources include one of the vulnerable images called kodekloud/webapp-color:v1. Find out the release name and uninstall it.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    student-node ~ ➜  helm list -A 
    NAME                    NAMESPACE               REVISION        UPDATED                                 STATUS     CHART                           APP VERSION
    atlanta-page-apd        atlanta-page-04         1               2025-03-10 01:45:29.089962328 +0000 UTC deployed   atlanta-page-apd-0.1.0          1.16.0     
    digi-locker-apd         digi-locker-02          1               2025-03-10 01:45:27.993635738 +0000 UTC deployed   digi-locker-apd-0.1.0           1.16.0     
    security-alpha-apd      security-alpha-01       1               2025-03-10 01:45:27.461324897 +0000 UTC deployed   security-alpha-apd-0.1.0        1.16.0     
    traefik                 kube-system             1               2025-03-09 23:33:12.604180561 +0000 UTC deployed   traefik-25.0.2+up25.0.0         v2.10.5    
    traefik-crd             kube-system             1               2025-03-09 23:32:56.186904986 +0000 UTC deployed   traefik-crd-25.0.2+up25.0.0     v2.10.5    
    web-dashboard-apd       web-dashboard-03        1               2025-03-10 01:45:28.522906987 +0000 UTC deployed   web-dashboard-apd-0.1.0         1.16.0     

    student-node ~ ✖ helm get valus atlanta-page-apd -n atlanta-page-04 |grep "image"

    student-node ~ ➜  helm get values atlanta-page-apd -n atlanta-page-04
    USER-SUPPLIED VALUES:
    null

    student-node ~ ➜  helm get values atlanta-page-apd -n atlanta-page-04 -o yaml
    null

    Solution
    Run the following command to change the context: -

    kubectl config use-context cluster2



    In this task, we will use the helm commands and jq tool. Here are the steps: -



    Run the helm ls command with -A option to list the releases deployed on all the namespaces using helm.
    helm ls -A



    We will use the jq tool to extract the image name from the deployments.
    kubectl get deploy -n <NAMESPACE> <DEPLOYMENT-NAME> -o json | jq -r '.spec.template.spec.containers[].image'



    Replace <NAMESPACE> with the namespace and <DEPLOYMENT-NAME> with the deployment name, which we get from the previous commands.

    After finding the kodekloud/webapp-color:v1 image, use the helm uninstall to remove the deployed chart that are using this vulnerable image.


    helm uninstall <RELEASE-NAME> -n <NAMESPACE> 





### SECTION: SERVICES AND NETWORKING
11. Please use the namespace nginx-depl-svcn for the following scenario.

    Create a deployment with name nginx-ckad10-svcn using nginx image with 2 replicas. Also expose the deployment via ClusterIP service .i.e. nginx-ckad10-service-svcn on port 80. Use the label app=nginx-ckad for both resources.

    Now, create a NetworkPolicy .i.e. netpol-ckad-allow-svcn so that only pods with label criteria: allow can access the deployment and apply it.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  k create deploy nginx-ckad10-svcn -n  nginx-depl-svcn --image=nginx --replicas=2
    deployment.apps/nginx-ckad10-svcn created

    student-node ~ ➜  k get deploy -n nginx-depl-svcn 
    NAME                READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-ckad10-svcn   2/2     2            2           4s

    student-node ~ ➜  k expose deploy -n nginx-depl-svcn nginx-ckad10-svcn --port=80 --name=nginx-ckad10-service-svcn --labels app=nginx-ckad
    service/nginx-ckad10-service-svcn exposed

    student-node ~ ✖ k get deploy -n nginx-depl-svcn 
    NAME                READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-ckad10-svcn   2/2     2            2           2m2s

    student-node ~ ➜  k get deploy -n nginx-depl-svcn nginx-ckad10-svcn -o yaml > nginx-depl-svcn.yaml

    student-node ~ ➜  vim nginx-depl-svcn.yaml 

    student-node ~ ➜  kr nginx-depl-svcn.yaml --force
    deployment.apps "nginx-ckad10-svcn" deleted
    deployment.apps/nginx-ckad10-svcn replaced

    student-node ~ ➜  k get deploy -n nginx-depl-svcn 
    NAME                READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-ckad10-svcn   0/2     2            0           2s

    student-node ~ ➜  k describe deploy -n nginx-depl-svcn nginx-ckad10-svcn 
    Name:                   nginx-ckad10-svcn
    Namespace:              nginx-depl-svcn
    CreationTimestamp:      Mon, 10 Mar 2025 02:05:36 +0000
    Labels:                 app=nginx-ckad10-svcn
    Annotations:            deployment.kubernetes.io/revision: 1
    Selector:               app=nginx-ckad
    Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
    Labels:  app=nginx-ckad
    Containers:
    nginx:
        Image:         nginx
        Port:          <none>
        Host Port:     <none>
        Environment:   <none>
        Mounts:        <none>
    Volumes:         <none>
    Node-Selectors:  <none>
    Tolerations:     <none>
    Conditions:
    Type           Status  Reason
    /----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-ckad10-svcn-8476cdff5c (2/2 replicas created)
    Events:
    Type    Reason             Age   From                   Message
    /----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  8s    deployment-controller  Scaled up replica set nginx-ckad10-svcn-8476cdff5c to 2

    student-node ~ ➜  k get deploy -n nginx-depl-svcn 
    NAME                READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-ckad10-svcn   2/2     2            2           15s

    student-node ~ ✖ vim netpol-ckad-allow-svcn.yaml
    student-node ~ ➜  cat netpol-ckad-allow-svcn.yaml
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
    name: netpol-ckad-allow-svcn
    namespace: nginx-depl-svcn
    spec:
    podSelector:
        matchLabels:
        app: nginx-ckad    
    policyTypes:
    - Ingress
    ingress:
    - from:
        - podSelector:
            matchLabels:
            criteria: allow

    student-node ~ ➜  kc netpol-ckad-allow-svcn.yaml
    networkpolicy.networking.k8s.io/netpol-ckad-allow-svcn created

    student-node ~ ➜  k get netpol -n nginx-depl-svcn 
    NAME                     POD-SELECTOR     AGE
    netpol-ckad-allow-svcn   app=nginx-ckad   11s

    student-node ~ ✖ k port-forward -n nginx-depl-svcn svc/nginx-ckad10-service-svcn 80:80
    error: timed out waiting for the condition

    student-node ~ ✖ k get pod -n nginx-depl-svcn 
    NAME                                 READY   STATUS    RESTARTS   AGE
    nginx-ckad10-svcn-8476cdff5c-9kfnz   1/1     Running   0          9m41s
    nginx-ckad10-svcn-8476cdff5c-t5qjg   1/1     Running   0          9m41s

    student-node ~ ➜  k exec -it -n nginx-depl-svcn nginx-ckad10-svcn-8476cdff5c-9kfnz -- sh
    /# curl 172.20.52.7:80
    curl: (7) Failed to connect to 172.20.52.7 port 80 after 466 ms: Couldn't connect to server
    /#


    Solution:

    Use the following to create the deployment and the service.

    kubectl apply -n nginx-depl-svcn -f - <<eof
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: nginx-ckad10-svcn
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: nginx-ckad
    template:
        metadata:
        labels:
            app: nginx-ckad
        spec:
        containers:
            - name: nginx
            image: nginx
            ports:
                - containerPort: 80
    
    apiVersion: v1
    kind: Service
    metadata:
    name: nginx-ckad10-service-svcn
    spec:
    selector:
        app: nginx-ckad
    ports:
        - name: http
        port: 80
        targetPort: 80
    type: ClusterIP
    eof



    To create a NetworkPolicy that only allows pods with labels criteria: allow to access the deployment, you can use the following YAML definition:


    kubectl apply -n nginx-depl-svcn -f - <<eof
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
    name: netpol-ckad-allow-svcn
    spec:
    podSelector:
        matchLabels:
        app: nginx-ckad
    ingress:
        - from:
            - podSelector:
                matchLabels:
                criteria: allow
        ports:
            - protocol: TCP
            port: 80
    eof


12. (空題) We have deployed an application in the ns-new-ckad namespace. We also configured services, namely frontend-ckad-svcn and backend-ckad-svcn.

    However, there are some issues:
    backend-ckad-svcn is not able to access backend pods
    frontend-ckad-svcn is not accessible from backend pods

    Inspect them and troubleshoot the issues.
    Note: You can modify the resources.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    student-node ~ ➜  k get all -n ns-new-ckad 
    NAME                READY   STATUS    RESTARTS   AGE
    pod/backend-pods    1/1     Running   0          4m22s
    pod/frontend-pods   1/1     Running   0          4m21s
    pod/testpod         1/1     Running   0          4m21s

    NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    service/backend-ckad-svcn    ClusterIP   172.20.60.141   <none>        80/TCP    4m22s
    service/frontend-ckad-svcn   ClusterIP   172.20.55.123   <none>        80/TCP    4m21s



13. (粗心) Deploy a pod with name messaging-ckad04-svcn using the redis:alpine image with the label tier=msg.

    Now, Create a service messaging-service-ckad04-svcn to expose the pod messaging-ckad04-svcn application within the cluster on port 6379.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    student-node ~ ➜  k run messaging-ckad04-svcn --image=redis:alpine -l tier=msg (正確)
    pod/messaging-ckad04-svcn created

    student-node ~ ➜  k expose pod messaging-ckad04-svcn --port=6379 (沒有設置name! --name messaging-service-ckad04-svcn)
    service/messaging-ckad04-svcn exposed

    student-node ~ ➜  k get svc
    NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    kubernetes              ClusterIP   10.43.0.1      <none>        443/TCP    166m
    messaging-ckad04-svcn   ClusterIP   10.43.65.214   <none>        6379/TCP   1s

    
    Solution

    Switch to cluster1 :
    kubectl config use-context cluster1

    On student-node, use the command kubectl run messaging-ckad04-svcn --image=redis:alpine -l tier=msg

    Now run the command: kubectl expose pod messaging-ckad04-svcn --port=6379 --name messaging-service-ckad04-svcn.

14. (空題)


### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY

16. Create a service account named ckad23-sa-aecs in the namespace ckad23-nssa-aecs.

    Grant the service account get and list permissions to access all resources within the namespace using a Role named wide-access-aecs.

    Also bind the Role to the service account using a RoleBinding named wide-access-rb-aecs, restricting the access to the ckad23-nssa-aecs namespace only.
    Note: If the resources do not exist, please create them as well.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  k create sa ckad23-sa-aecs -n ckad23-nssa-aecs
    error: failed to create serviceaccount: namespaces "ckad23-nssa-aecs" not found

    student-node ~ ✖ k create ns ckad23-nssa-aecs
    namespace/ckad23-nssa-aecs created

    student-node ~ ➜  k create sa ckad23-sa-aecs -n ckad23-nssa-aecs
    serviceaccount/ckad23-sa-aecs created

    student-node ~ ✖ k create role wide-access-aecs --verb=get,list --resource=* -n ckad23-nssa-aecs 
    role.rbac.authorization.k8s.io/wide-access-aecs created     


    student-node ~ ✖ kubectl create rolebinding wide-access-rb-aecs --role=wide-access-aecs  --group=system:serviceaccounts:ckad23-sa-aecs -n ckad23-nssa-aecs 
    rolebinding.rbac.authorization.k8s.io/wide-access-rb-aecs created

    student-node ~ ➜  k get rolebindings.rbac.authorization.k8s.io -n ckad23-nssa-aecs 
    NAME                  ROLE                    AGE
    wide-access-rb-aecs   Role/wide-access-aecs   7s


    Solution
    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  kubectl create ns ckad23-nssa-aecs
    namespace/ckad23-nssa-aecs created

    student-node ~ ➜  kubectl create serviceaccount ckad23-sa-aecs -n ckad23-nssa-aecs
    serviceaccount/ckad23-sa-aecs created

    student-node ~ ➜  kubectl create role wide-access-aecs --namespace=ckad23-nssa-aecs --verb=get,list --resource=* 
    role.rbac.authorization.k8s.io/wide-access-aecs created

    student-node ~ ➜  kubectl create rolebinding wide-access-rb-aecs \
    --role=wide-access-aecs \
    --serviceaccount=ckad23-nssa-aecs:ckad23-sa-aecs \
    --namespace=ckad23-nssa-aecs
    rolebinding.rbac.authorization.k8s.io/wide-access-rb-aecs created

19. (空題)

### SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE (全對)