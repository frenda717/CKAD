1.32 
First Attempt: 
Score: 40% Pass Score: 75%
Correct Answer: 01,03,05,06,08,09,12,13,15,18
難題: 01, 14(CRD), 19 (提取欄位)
錯題: 02,04,07,10 ,14,16
寫不完: 17~21
1.32版本新Concept: Configurable container restart delay (02) pod annotation (04) CRD subresource (14)

### SECTION: APPLICATION DESIGN AND BUILD
01. (Container Args vs. Command)
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: ckad-sidecar-pod
    name: ckad-sidecar-pod
    namespace: ckad-multi-containers
    spec:
    containers:
    - image: nginx:1.16
        name: main-container
        resources: {}
        volumeMounts:
        - mountPath: /usr/share/nginx/html
        name: my-vol
    - image: busybox:1.28
        name: sidecar-container
        resources: {}
        args: [/bin/sh, -c,
                'i=0; while true; do echo "$i: $(date)" "Hi I am from Sidecar container" >> /var/log/index.html ; i=$((i+1)); sleep 5; done']
        volumeMounts:
        - mountPath: /var/log
        name: my-vol
    volumes:
    - name: my-vol
        emptyDir: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}


    在 Kubernetes 的 Pod 規格 (Pod Spec) 中，有兩個跟容器啟動時「要執行什麼指令」相關的欄位：command 與 args。
    它們在語意上對應到 Dockerfile 裡的 ENTRYPOINT 與 CMD，概念上是分開的，但運作方式可以大致說明如下：

    **command** (ENTRYPOINT)
    在 Kubernetes 中，如果在 Pod spec 裡設定了 command，就會覆蓋容器映像檔中既有的 ENTRYPOINT。
    command **表示「容器啟動後第一個執行的可執行檔** (executable) 與參數」，也可以理解為主要執行的程式。
    若只指定 command 而沒指定 args，整個 command 會連帶包含參數在內（這時就跟 Dockerfile 的 ENTRYPOINT + CMD 結合的結果相似）。
    
    **args** (CMD)
    如果在 Pod spec 中沒有指定 command，則會直接沿用容器映像檔中的 ENTRYPOINT；同時，args 會去覆蓋容器映像檔中的 CMD，作為該 ENTRYPOINT 的參數。
    若有指定 command，那麼 args 裡的內容會被視為該 command 的參數。
    
    什麼時候該用 command，什麼時候該用 args？
    若你希望**改變容器的「執行主程式」**（例如，把原本映像檔裡的啟動程式整個換掉），就需要使用 command。
    若只是**想修改或新增「啟動程式的參數」**，而映像檔本身的啟動程式是正確的，就可以只指定 args。

    廚房的比喻
    容器映像檔（Image）
    就像是一間「已經準備好基本配方與廚師」的廚房。在 Docker/Kubernetes 世界中，一個映像檔通常有「預設要執行的程式（預設廚師）」以及「預設參數（預設食材或配方）」。

    command → 換掉「廚師」
    當我們在 Pod Spec 中設定 command，就表示把原本映像檔裡的「預設廚師」整個換掉，改由你指定的程式（新廚師）來做菜。所以，如果你只想「調整食材」卻誤用 command，可能會把整個原本的主程式給替換掉，最後完全跟映像檔預設的做法不一樣。

    如果你沒有再指定 args，那「新廚師」就是照著 command 中的所有細節（包含你列的參數）去執行。
    args → 換掉「配方或食材」
    當我們在 Pod Spec 中只設定 args，就表示我們保留了映像檔裡的「預設廚師」（預設程式），只是改用另一份配方或多換一些食材，讓同一位廚師用不同的材料來做菜。

    也就是說：如果映像檔原本已經設定了某個預設程式 (ENTRYPOINT)，那麼 args 會取代預設的參數 (CMD)，當作那個預設程式的新參數來執行。


    student-node ~ ➜  kr ckad-sidecar-pod.yaml --force
    pod "ckad-sidecar-pod" deleted
    pod/ckad-sidecar-pod replaced

    student-node ~ ➜  k exec -it -n ckad-multi-containers ckad-sidecar-pod -c sidecar-container -- cat /var/log/index.html
    0: Sat Feb 22 16:54:45 UTC 2025 Hi I am from Sidecar container
    1: Sat Feb 22 16:54:50 UTC 2025 Hi I am from Sidecar container
    2: Sat Feb 22 16:54:55 UTC 2025 Hi I am from Sidecar container
    3: Sat Feb 22 16:55:00 UTC 2025 Hi I am from Sidecar container

    doc: https://kubernetes.io/docs/concepts/cluster-administration/logging/




02. (container Lifecycle)In the ckad-multi-containers namespace, create pod named dos-containers-pod which has 2 containers matching the below requirements:
    The first container named alpha runs the nginx:1.17 image and has the ROLE=SERVER ENV variable configured.
    The second container named beta, runs busybox:1.28 image. This container will print message Hello multi-containers (command needs to be run in shell).

    NOTE: **all containers should be in a running state to pass the validation**. --> 
    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1


    student-node ~ ➜  k run dos-containers-pod -n ckad-multi-containers --image=nginx:1.17 --dry-run=client -o yaml > dos-containers-pod.yaml

    student-node ~ ➜  vim dos-containers-pod.yaml 

    student-node ~ ➜  kc dos-containers-pod.yaml 
    pod/dos-containers-pod created

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                 READY   STATUS             RESTARTS     AGE
    ckad-sidecar-pod     2/2     Running            0            4m3s
    dos-containers-pod   1/2     CrashLoopBackOff   1 (3s ago)   7s

    student-node ~ ➜  k describe pod -n ckad-multi-containers dos-containers-pod 
    Name:             dos-containers-pod
    Namespace:        ckad-multi-containers
    Priority:         0
    Service Account:  default
    Node:             cluster1-node02/192.168.140.146
    Start Time:       Sat, 22 Feb 2025 16:58:39 +0000
    Labels:           run=dos-containers-pod
    Annotations:      <none>
    Status:           Running
    IP:               10.42.2.4
    IPs:
    IP:  10.42.2.4
    Containers:
    alpha:
        Container ID:   containerd://8c2d199dd4314cf6f07abc180b1f4ae7e7c522ded19b91d0fa3cbd01a6f23840
        Image:          nginx:1.17
        Image ID:       docker.io/library/nginx@sha256:6fff55753e3b34e36e24e37039ee9eae1fe38a6420d8ae16ef37c92d1eb26699
        Port:           <none>
        Host Port:      <none>
        State:          Running
        Started:      Sat, 22 Feb 2025 16:58:42 +0000
        Ready:          True
        Restart Count:  0
        Environment:
        ROLE:  SERVER
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-58lkx (ro)
    beta:
        Container ID:  containerd://c15185b2d1934d1d55dc3b23bdb053730eeffeb4f947815ae18c60799172788f
        Image:         busybox:1.28
        Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
        Port:          <none>
        Host Port:     <none>
        Command:
        /bin/sh
        -c
        echo "Hello multi-containers"
        State:          Waiting
        Reason:       CrashLoopBackOff
        Last State:     Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Sat, 22 Feb 2025 16:58:43 +0000
        Finished:     Sat, 22 Feb 2025 16:58:43 +0000
        Ready:          False
        Restart Count:  1
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-58lkx (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       False 
    ContainersReady             False 
    PodScheduled                True 
    Volumes:
    kube-api-access-58lkx:
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
    Type     Reason     Age                From               Message
    ----     ------     ----               ----               -------
    Normal   Scheduled  14s                default-scheduler  Successfully assigned ckad-multi-containers/dos-containers-pod to cluster1-node02
    Normal   Pulling    14s                kubelet            Pulling image "nginx:1.17"
    Normal   Pulled     12s                kubelet            Successfully pulled image "nginx:1.17" in 2.668s (2.668s including waiting)
    Normal   Created    12s                kubelet            Created container alpha
    Normal   Started    12s                kubelet            Started container alpha
    Normal   Pulling    12s                kubelet            Pulling image "busybox:1.28"
    Normal   Pulled     11s                kubelet            Successfully pulled image "busybox:1.28" in 233ms (233ms including waiting)
    Normal   Pulled     11s                kubelet            Container image "busybox:1.28" already present on machine
    Normal   Created    11s (x2 over 11s)  kubelet            Created container beta
    Normal   Started    11s (x2 over 11s)  kubelet            Started container beta
    Warning  BackOff    9s (x2 over 10s)   kubelet            Back-off restarting failed container beta in pod dos-containers-pod_ckad-multi-containers(9d7cfa64-caaa-45f8-8fc5-906d3c659b95)

    student-node ~ ➜  k logs -n ckad-multi-containers dos-containers-pod -c beta
    Hello multi-containers

    Solution:
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: dos-containers-pod
    name: dos-containers-pod
    namespace: ckad-multi-containers
    spec:
    containers:
    - image: nginx:1.17
        env:
        - name: ROLE
        value: SERVER
        name: alpha
        resources: {}
    - command:
        - /bin/sh
        - -c
        - echo Hello multi-containers; **sleep 4000;** # 由於題目: all containers should be in a running state (同mock 01第7題)
        image: busybox:1.28
        name: beta
        resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    **busybox： 預設沒有常駐進程，如果不指定 command: sleep 4000，就會在執行預設行為後結束**。
    **nginx： 以 daemon 形式啟動伺服器，預設就會常駐，因此不需要額外的 sleep 命令**。

    Then use kubectl apply -f file_name.yaml to create the required object.

    In the exam, you can take advantage of dry-run flag to generate the YAML and modify it as question requirements.

    kubectl run dos-containers-pod -n ckad-multi-containers --image=nginx:1.17 **--env ROLE=SERVER** --dry-run=client -o yaml > file_name.yaml

    Next, use vim to modify the created YAML file, follow the question requirements.

    vi file_name.yaml



03. student-node ~ ➜  k create job alpine-job --image=alpine -n ckad-job
    job.batch/alpine-job created

    student-node ~ ➜  k create job alpine-job --image=alpine -n ckad-job --dry-run=client -o yaml > alpine-job.yaml
    student-node ~ ➜  vim alpine-job.yaml 
    student-node ~ ➜  kr alpine-job.yaml 

    student-node ~ ➜  k get pod -n ckad-job 
    NAME               READY   STATUS    RESTARTS   AGE
    alpine-job-z44bf   1/1     Running   0          3m4s


    student-node ~ ➜  k logs -n ckad-job alpine-job-z44bf

    Mem: 118353868K used, 13638708K free, 204796K shrd, 8632416K buff, 71554568K ca
    CPU:   5% usr   6% sys   0% nic  87% idle   0% io   0% irq   0% sirq
    Load average: 4.64 5.76 6.77 5/15631 56
    PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND

04. In the ckad-pod-design namespace, start a ckad-redis-wiipwlznjy pod running the redis image; the container should be named redis-custom-annotation.

    Configure a custom annotation to that pod as below:
    KKE: https://engineer.kodekloud.com/

    For this question, please set the context to cluster2 by running:

    kubectl config use-context cluster2

    
    student-node ~ ➜  k run ckad-redis-wiipwlznjy -n ckad-pod-design --image=redis --dry-run=client -o yaml > ckad-redis-wiipwlznjy.yaml

    student-node ~ ➜  vim ckad-redis-wiipwlznjy.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: ckad-redis-wiipwlznjy
    name: ckad-redis-wiipwlznjy
    namespace: ckad-pod-design
    annotations:
        ~~imageregistry: "KKE: https://engineer.kodekloud.com/"~~ # 配置錯誤，應為:KKE: https://engineer.kodekloud.com/
                                        
    spec:
    containers:
    - image: redis
        name: redis-custom-annotation
        resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    student-node ~ ➜  kc ckad-redis-wiipwlznjy.yaml 
    pod/ckad-redis-wiipwlznjy created

    student-node ~ ➜  k get pod -n ckad-pod-design 
    NAME                    READY   STATUS    RESTARTS   AGE
    ckad-redis-wiipwlznjy   1/1     Running   0          8s

    Soution:
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: ckad-redis-wiipwlznjy
    name: ckad-redis-wiipwlznjy
    namespace: ckad-pod-design
    annotations: 
        **KKE: https://engineer.kodekloud.com/** # doc: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/
    spec:
    containers:
    - image: redis
        name: redis-custom-annotation
        resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}



    Then use kubectl apply -f file_name.yaml to create the required object.
    Alternatively, you can use this command for similar outcome:

    kubectl run ckad-redis-wiipwlznjy -n ckad-pod-design --image=redis **--annotations KKE=https://kodekloud.com**

### SECTION: APPLICATION DEPLOYMENT

06. 
    student-node ~ ➜  k create deploy ocean-apd --image=kodekloud/webapp-color:v1 --replicas=2 --dry-run=client -o yaml > ocean-apd.yaml
    student-node ~ ➜  cat ocean-apd.yaml 
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    creationTimestamp: null
    labels:
        app: ocean-apd
    name: ocean-apd
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: ocean-apd
    strategy: {}
    template:
        metadata:
        creationTimestamp: null
        labels:
            app: ocean-apd
        spec:
        containers:
        - image: kodekloud/webapp-color:v1
            name: webapp-color
            resources: {}
    strategy:
        type: RollingUpdate
        rollingUpdate:
        maxUnavailable: 45%
        maxSurge: 55%
    status: {}

    student-node ~ ➜  kc ocean-apd.yaml 
    deployment.apps/ocean-apd created

    student-node ~ ➜  k set image deployment/ocean-apd kodekloud/webapp-color:v2
    error: the server doesn't have a resource type "kodekloud"

    student-node ~ ✖ k set image deployment/ocean-apd image=kodekloud/webapp-color:v2
    error: unable to find container named "image"

    student-node ~ ➜  k describe pod ocean-apd-f665c6949-5xgjg |grep pod
                    pod-template-hash=f665c6949
                    cni.projectcalico.org/podIP: 172.17.1.3/32
                    cni.projectcalico.org/podIPs: 172.17.1.3/32

    student-node ~ ➜  k describe pod ocean-apd-f665c6949-5xgjg |grep image
    Normal  Pulling    3m49s  kubelet            Pulling image "kodekloud/webapp-color:v1"
    Normal  Pulled     3m46s  kubelet            Successfully pulled image "kodekloud/webapp-color:v1" in 139ms (2.384s including waiting)

    student-node ~ ➜  k set image deployment/ocean-apd webapp-color=kodekloud/webapp-color:v2
    deployment.apps/ocean-apd image updated

    student-node ~ ➜  k rollout history deploy ocean-apd > /opt/ocean-revision-count.txt

    student-node ~ ➜  cat /opt/ocean-revision-count.txt
    deployment.apps/ocean-apd 
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>


    student-node ~ ➜  k rollout undo deployment ocean-apd 
    deployment.apps/ocean-apd rolled back

    student-node ~ ➜  k rollout history deployment ocean-apd 
    deployment.apps/ocean-apd 
    REVISION  CHANGE-CAUSE
    2         <none>
    3         <none>


07. (Helm)This deployment will be facilitated through the kubernetes Helm chart on the cluster1-controlplane node to fulfill the requirements of our new client

    The chart URL and other specifications are as follows: -
    The chart URL link - https://kubernetes.github.io/dashboard/
    The chart repository name should be kubernetes-dashboard.
    The release name should be kubernetes-dashboard-server.
    All the resources should be deployed on the cd-tool-apd namespace.

    NOTE: - You have to perform this task from the student-node.

    student-node ~ ➜  helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
    "kubernetes-dashboard" has been added to your repositories

    student-node ~ ✖ helm search repo kubernetes-dashboard
    NAME                                            CHART VERSION   APP VERSION     DESCRIPTION                                   
    kubernetes-dashboard/kubernetes-dashboard       7.10.4                          General-purpose web UI for Kubernetes clusters

    student-node ~ ✖ helm install   **kubernetes-dashboard-server**  **kubernetes-dashboard/kubernetes-dashboard**  --namespace cd-tool-apd --create-namespace
    NAME: kubernetes-dashboard-server
    LAST DEPLOYED: Sun Feb 23 00:04:53 2025
    NAMESPACE: cd-tool-apd
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    ...
 

### SECTION: SERVICES AND NETWORKING

09.  
    student-node ~ ➜  k run pod21-ckad-svcn --image=nginx:alpine
    pod/pod21-ckad-svcn created

    student-node ~ ➜  k expose pod pod21-ckad-svcn --port=80
    service/pod21-ckad-svcn exposed

    student-node ~ ➜  k get pod
    NAME                        READY   STATUS    RESTARTS   AGE
    ocean-apd-f665c6949-2l92b   1/1     Running   0          118s
    ocean-apd-f665c6949-6bbw9   1/1     Running   0          118s
    pod21-ckad-svcn             1/1     Running   0          18s

    student-node ~ ➜  k get svc
    NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
    kubernetes        ClusterIP   172.20.0.1    <none>        443/TCP   62m
    pod21-ckad-svcn   ClusterIP   172.20.18.6   <none>        80/TCP    6s


10. We have already deployed an application that consists of frontend, backend, and database pods in the app-ckad namespace. Inspect them.
    Your task is to create:
    A service frontend-ckad-svcn to expose the frontend pods outside the cluster on port 31100.
    A service backend-ckad-svcn to make backend pods to be accessible within the cluster.
    A policy database-ckad-netpol to **limit access to database pods only to backend pods**. -> 僅將資料庫 pod 的存取權限限制為後端 pod

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    我的配置:
    student-node ~ ➜  k get pod -n app-ckad 
    NAME                READY   STATUS    RESTARTS   AGE
    backend-pod-svcn    1/1     Running   0          14m
    database-pod-svcn   1/1     Running   0          14m
    frontend-pod-svcn   1/1     Running   0          14m

    student-node ~ ➜  k create svc nodeport frontend-ckad-svcn -n app-ckad --tcp=31100
    service/frontend-ckad-svcn created

    student-node ~ ➜  k get svc -n app-ckad 
    NAME                 TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
    frontend-ckad-svcn   NodePort   172.20.23.250   <none>        31100:31753/TCP   3s


    student-node ~ ➜  k get pod -n app-ckad 
    NAME                READY   STATUS    RESTARTS   AGE
    backend-pod-svcn    1/1     Running   0          14m
    database-pod-svcn   1/1     Running   0          14m
    frontend-pod-svcn   1/1     Running   0          14m

    student-node ~ ✖ k describe pod -n app-ckad backend-pod-svcn |grep Port
    Port:           80/TCP
    Host Port:      0/TCP

    student-node ~ ➜  k describe pod -n app-ckad database-pod-svcn |grep Port
    Port:           80/TCP
    Host Port:      0/TCP

    student-node ~ ➜  k create svc nodeport frontend-ckad-svcn -n app-ckad --tcp=31100
    service/frontend-ckad-svcn created

    student-node ~ ➜  k get svc -n app-ckad 
    NAME                 TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
    frontend-ckad-svcn   NodePort   172.20.23.250   <none>        31100:31753/TCP   3s

    student-node ~ ➜  k get pod -n app-ckad backend-pod-svcn -o yaml > backend-pod-svcn.yaml

    student-node ~ ➜  vim backend-pod-svcn.yaml 

    student-node ~ ➜  k create svc nodeport frontend-ckad-svcn -n app-ckad --tcp=31100^C

    student-node ~ ✖ k expose pod -n app-ckad backend-pod-svcn --port=80
    service/backend-pod-svcn exposed


    student-node ~ ➜  k get svc -n app-ckad 
    NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
    backend-pod-svcn     ClusterIP   172.20.124.175   <none>        80/TCP            46s
    frontend-ckad-svcn   NodePort    172.20.23.250    <none>        31100:31753/TCP   2m46s

    student-node ~ ➜  k get pod -n app-ckad --show-labels 
    NAME                READY   STATUS    RESTARTS   AGE   LABELS
    backend-pod-svcn    1/1     Running   0          23m   app=backend
    database-pod-svcn   1/1     Running   0          23m   app=database
    frontend-pod-svcn   1/1     Running   0          23m   app=frontend

    student-node ~ ➜  vim database-ckad-netpol.yaml
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
    name: database-ckad-netpol
    namespace: app-ckad
    spec:
        **podSelector**:  # 口訣: 使用 spec.podSelector 來**鎖定「想要受政策保護的 Pod」**
            matchLabels:
            ~~app: backend~~ # 應為:app: database
        ingress:
            - from:
              - **podSelector**: # 口訣: 「**允許哪些來源 Pod 可以對目標 Pod進行存取**」
                  matchLabels:
                      ~~app: database~~ 應為: app: backend

    policyTypes:
    - Ingress
    
    student-node ~ ✖ kc database-ckad-netpol.yaml
    networkpolicy.networking.k8s.io/database-ckad-netpol created

    student-node ~ ➜  k get netpol -n app-ckad 
    NAME                   POD-SELECTOR   AGE
    database-ckad-netpol   app=backend    5s


    Solution:

    A service frontend-ckad-svcn to expose the frontend pods outside the cluster on port 31100. # 這步我設置正確
    apiVersion: v1
    kind: Service
    metadata:
    name: frontend-ckad-svcn
    namespace: app-ckad
    spec:
    selector:
        app: frontend
    type: NodePort
    ports:
    - name: http
        port: 80
        targetPort: 80
        nodePort: 31100

    A service backend-ckad-svcn to make backend pods to be accessible within the cluster. # 這步我設置正確
    apiVersion: v1
    kind: Service
    metadata:
    name: backend-ckad-svcn
    namespace: app-ckad
    spec:
    selector:
        app: backend
    ports:
    - name: http
        port: 80
        targetPort: 80

    
    A policy database-ckad-netpol to limit access of database pods only to backend pods. # 這步我設置正確
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
    name: database-ckad-netpol
    namespace: app-ckad
    spec:
        podSelector:
            matchLabels:
            **app: database**
        ingress:
        - from:
            - podSelector:
                matchLabels:
                **app: backend**

                


11. For this scenario, create a deployment named hr-web-app-ckad05-svcn using the image kodekloud/webapp-color with 2 replicas.
    Expose the hr-web-app-ckad05-svcn with service named hr-web-app-service-ckad05-svcn on port 30082 on the nodes of the cluster.
    Note: The web application listens on port 8080.
    
    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    我的配置:

    student-node ~ ➜  k create deploy hr-web-app-ckad05-svcn --image=kodekloud/webapp-color --replicas=2
    deployment.apps/hr-web-app-ckad05-svcn created

    student-node ~ ➜  k expose deploy hr-web-app-ckad05-svcn --port=30082 --dry-run=client -o yaml > hr-web-app-ckad05-svcn.yaml

    student-node ~ ➜  vim hr-web-app-ckad05-svcn.yaml 
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        app: hr-web-app-ckad05-svcn
    name: hr-web-app-service-ckad05-svcn
    spec:
    type: NodePort
    ports:
    - port: 30082
        protocol: TCP
        targetPort: 30082
    selector:
        app: hr-web-app-ckad05-svcn
    status:
    loadBalancer: {}


    student-node ~ ➜  kc hr-web-app-ckad05-svcn.yaml 
    service/hr-web-app-service-ckad05-svcn created


    student-node ~ ➜  k get svc
    NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
    frontend-ckad-svcn               NodePort    172.20.28.203   <none>        31100:31203/TCP   29m
    hr-web-app-service-ckad05-svcn   NodePort    172.20.83.13    <none>        30082:32227/TCP   6s
    kubernetes                       ClusterIP   172.20.0.1      <none>        443/TCP           92m
    pod21-ckad-svcn                  ClusterIP   172.20.18.6     <none>        80/TCP            30m


    Solution: 

    Switch to cluster3 :
    kubectl config use-context cluster3

    On student-node, use the command: kubectl create deployment  hr-web-app-ckad05-svcn --image=kodekloud/webapp-color --replicas=2

    Now we can run the command: kubectl expose deployment hr-web-app-ckad05-svcn --type=NodePort --port=8080 --name=hr-web-app-service-ckad05-svcn --dry-run=client -o yaml > hr-web-app-service-ckad05-svcn.yaml to generate a service definition file.

    Now, in generated service definition file add the nodePort field with the given port number under the ports section and create a service.

12. 
    student-node ~ ➜  k create deploy nginx-ckad11 -n nginx-deployment --image=nginx --replicas=2 --dry-run=client -o yaml > nginx-ckad11.yaml

    student-node ~ ➜  vim nginx-ckad11.yaml 


    student-node ~ ➜  k create deploy nginx-ckad11 -n nginx-deployment --image=nginx --replicas=2
    deployment.apps/nginx-ckad11 created

    student-node ~ ➜  k edit deployments.apps -n nginx-deployment nginx-ckad11  (修改label 為:app: nginx-ckad)
    deployment.apps/nginx-ckad11 edited 

    student-node ~ ➜  k expose deploy -n nginx-deployment nginx-ckad11 --port=80 --dry-run=client -o yaml > nginx-ckad11-svn.yaml 

    student-node ~ ➜  vim nginx-ckad11-svn.yaml 

    student-node ~ ➜  kc nginx-ckad11-svn.yaml 
    service/nginx-ckad11-service created

    student-node ~ ✖ vim ckad-allow.yaml
        apiVersion: networking.k8s.io/v1
        kind: NetworkPolicy
        metadata:
        name: ckad-allow
        namespace: nginx-deployment
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
            ports:
            - protocol: TCP
            port: 80

    student-node ~ ➜  kc ckad-allow.yaml
    networkpolicy.networking.k8s.io/ckad-allow created

    student-node ~ ➜  k get all -n nginx-deployment 
    NAME                                READY   STATUS    RESTARTS   AGE
    pod/test-pod                        1/1     Running   0          7m50s
    pod/nginx-ckad11-689d888dc5-qkmn6   1/1     Running   0          6m27s
    pod/nginx-ckad11-689d888dc5-265mc   1/1     Running   0          6m27s

    NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
    service/nginx-ckad11-service   ClusterIP   10.43.114.91   <none>        80/TCP    3m44s

    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx-ckad11   2/2     2            2           6m27s

    NAME                                      DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-ckad11-689d888dc5   2         2         2       6m27s


13. (SecurityContext)
    student-node ~ ➜  k get pod -n ckad05-securityctx-aecs ckad06-cap-aecs -o yaml > ckad06-cap-aecs.yaml

    student-node ~ ➜  vim ckad06-cap-aecs.yaml 
    spec:
    containers:
    - command:
    - sleep
    - "4800"
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu-sleeper
    resources: {}
    securityContext:
    runAsUser: root # Add
    capabilities:
        add:
        - SYS_TIME
        - NET_ADMIN # Add


    student-node ~ ➜  kr ckad06-cap-aecs.yaml 
    Error from server (BadRequest): error when replacing "ckad06-cap-aecs.yaml": Pod in version "v1" cannot be handled as a Pod: json: cannot unmarshal string into Go struct field SecurityContext.spec.containers.securityContext.runAsUser of type int64
    student-node ~ ➜  vim ckad06-cap-aecs.yaml 
    securityContext:
    **runAsUser: 0 # root改為0** runAsUser 必須是一個整數 (int64) 格式的參數，代表要以哪個 User ID (UID) 執行該容器的進程。
    因此，你不能直接填寫 "root" 這樣的字串，而必須填寫 0 來表示要以 root (UID=0) 身份運行。

    student-node ~ ✖ kr ckad06-cap-aecs.yaml --force
    pod "ckad06-cap-aecs" deleted
    pod/ckad06-cap-aecs replaced

    student-node ~ ➜  k get pod -n ckad05-securityctx-aecs 
    NAME              READY   STATUS    RESTARTS   AGE
    ckad06-cap-aecs   1/1     Running   0          42s


### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY
14. (CRD 難題!)Define a Kubernetes custom resource definition (CRD) for a new resource kind called Foo (plural form - foos) in the samplecontroller.k8s.io group.
    This CRD should have a version of v1alpha1 with a schema that includes two properties as given below:
    deploymentName (a string type) and replicas (an integer type with minimum value of 1 and maximum value of 5).

    It should also include a status subresource which enables retrieving and updating the status of Foo object, including the availableReplicas property, which is an integer type.
    The Foo resource should be namespace scoped.

    Note: We have provided a template /root/foo-crd-aecs.yaml for your ease. There are few issues with it so please make sure to incorporate the above requirements before deploying on cluster.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    我的配置:

    student-node ~ ➜  k explain CustomResourceDefinition.spec.versions.subresources
    GROUP:      apiextensions.k8s.io
    KIND:       CustomResourceDefinition
    VERSION:    v1

    FIELD: subresources <CustomResourceSubresources>


    DESCRIPTION:
        subresources specify what subresources this version of the defined custom
        resource have.
        CustomResourceSubresources defines the status and scale subresources for
        CustomResources.
        
    FIELDS:
    scale <CustomResourceSubresourceScale>
        scale indicates the custom resource should serve a `/scale` subresource that
        returns an `autoscaling/v1` Scale object.

    status        <CustomResourceSubresourceStatus>
        status indicates the custom resource should serve a `/status` subresource.
        When enabled: 1. requests to the custom resource primary endpoint ignore
        changes to the `status` stanza of the object. 2. requests to the custom
        resource `/status` subresource ignore changes to anything other than the
        `status` stanza of the object.

        
    student-node ~ ✖ vim Foo-crd.yaml

    student-node ~ ➜  kc Foo-crd.yaml
    error: error parsing Foo-crd.yaml: error converting YAML to JSON: yaml: line 31: did not find expected key

    student-node ~ ✖ vim Foo-crd.yaml (沒有配置完 差一點點)

    student-node ~ ➜  cat Foo-crd.yaml
    apiVersion: samplecontroller.k8s.io
    kind: CustomResourceDefinition
    metadata:
    name: foos.stable.example.com
    spec:
    group: stable.example.com
    scope: Namespaced
    names:
        plural: foos
        singular: foo
        kind: Foo
    versions:
    - name: v1alpha1
        served: true
        storage: true
        schema:
            openAPIV3Schema:
            type: object
            properties:
            spec:
                type: object
                properties:
                    deploymentName:
                    type: string
                    replicas:
                    type: integer
                    minimum: 1
                    maximum: 5
            # 這裡應為:
            **status**:
            **type**: object
            **properties**:
                availableReplicas:
                    type: integer
                ~~properties:~~
                ~~availableReplicas:~~
                    ~~type: integer~~
        subresources: # 這裡對了
            # status enables the status subresource.
            status: {}


    Solution:
    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  vim foo-crd-aecs.yaml

    student-node ~ ➜  cat foo-crd-aecs.yaml 
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
    name: foos.samplecontroller.k8s.io
    annotations:
        "api-approved.kubernetes.io": "unapproved, experimental-only"
    spec:
    group: samplecontroller.k8s.io
    scope: Namespaced
    names:
        kind: Foo
        plural: foos
    versions:
        - name: v1alpha1
        served: true
        storage: true
        schema:
            # schema used for validation
            openAPIV3Schema:
            type: object
            properties:
                spec:
                # Spec for schema goes here !
                type: object
                properties:
                    deploymentName:
                    type: string
                    replicas:
                    type: integer
                    minimum: 1
                    maximum: 5
                status:
                type: object
                properties:
                    availableReplicas:
                    type: integer
        # subresources for the custom resource
        subresources:
            # enables the status subresource
            status: {}

    student-node ~ ➜  kubectl apply -f foo-crd-aecs.yaml
    customresourcedefinition.apiextensions.k8s.io/foos.samplecontroller.k8s.io created


15. ()
    
    我的配置:
    student-node ~ ➜  

    student-node ~ ➜  kubectl config use-context cluster1
    Switched to context "cluster1".

    student-node ~ ➜  k get cm -n ckad02-mult-cm-cfg-aecs 
    NAME                  DATA   AGE
    kube-root-ca.crt      1      15s
    ckad02-config1-aecs   1      14s
    ckad02-config2-aecs   1      14s

    student-node ~ ➜  k get pod -n ckad02-mult-cm-cfg-aecs ckad02-test-pod-aecs -o yaml > ^C

    student-node ~ ✖ k get pod -n ckad02-mult-cm-cfg-aecs 
    NAME                   READY   STATUS    RESTARTS   AGE
    ckad02-test-pod-aecs   1/1     Running   0          77s

    student-node ~ ➜  k describe cm -n ckad02-mult-cm-cfg-aecs ckad02-config1-aecs 
    Name:         ckad02-config1-aecs
    Namespace:    ckad02-mult-cm-cfg-aecs
    Labels:       <none>
    Annotations:  <none>

        Data
        ====
        greetings.how:
        ----
        HI

        BinaryData
        ====

        Events:  <none>

    student-node ~ ➜  k describe cm -n ckad02-mult-cm-cfg-aecs ckad02-config2-aecs 
    Name:         ckad02-config2-aecs
    Namespace:    ckad02-mult-cm-cfg-aecs
    Labels:       <none>
    Annotations:  <none>

        Data
        ====
        man:
        ----
        HANDSOME

        BinaryData
        ====

        Events:  <none>



    student-node ~ ➜  k get pod -n ckad02-mult-cm-cfg-aecs ckad02-test-pod-aecs -o yaml > ckad02-test-pod-aecs.yaml

    student-node ~ ➜  vim ckad02-test-pod-aecs.yaml
    spec:
    containers:
    - command:
        - /bin/sh
        - -c
        - while true; do env | egrep 'GREETINGS|WHO'; sleep 10; done
        env:
        - name: GREETINGS
        valueFrom:
            configMapKeyRef:
            name: ckad02-config1-aecs
            key: greetings.how
        - name: WHO
        valueFrom:
            configMapKeyRef:
            name: ckad02-config2-aecs
            key: man


    student-node ~ ✖ kr ckad02-test-pod-aecs.yaml --force
    pod "ckad02-test-pod-aecs" deleted
    pod/ckad02-test-pod-aecs replaced


16. (原配置項未刪除導致配置被覆蓋) Create a Kubernetes Pod named ckad15-memory, with a container named ckad15-memory running the polinux/stress image, and         configure it to use the following specifications:
    Command: stress
    Arguments: ["--vm", "1", "--vm-bytes", "10M", "--vm-hang", "1"]
    Requested memory: 10Mi
    Memory limit: 10Mi

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    我的配置:

    student-node ~ ➜  vim ckad15-memory.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: ckad15-memory
    name: ckad15-memory
    spec:
    containers:
    - image: polinux/stress
        name: ckad15-memory
        command: ["/bin/sh","-c","stress"]
        args: ["--vm", "1", "--vm-bytes", "10M", "--vm-hang", "1"]
        resources:
        requests:
            memory: "10Mi"
        limits:
            memory: "10Mi"
        resources: {} # **多了這筆!! 原本的配置應刪除，以避免被覆蓋**!!!
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    student-node ~ ✖ k create -f  ckad15-memory.yaml 
    pod/ckad15-memory created


    student-node ~ ➜  kubectl config use-context cluster2
    Switched to context "cluster2".

    Solution:

    student-node ~ ➜  cat << EOF | kubectl apply -f -
    apiVersion: v1
    kind: Pod
    metadata:
    name: ckad15-memory
    spec:
    containers:
    - name: ckad15-memory
        image: polinux/stress
        resources:
        requests:
            memory: "10Mi"
        limits:
            memory: "10Mi"
        command: ["stress"]
        args: ["--vm", "1", "--vm-bytes", "10M", "--vm-hang", "1"]
    EOF

    pod/ckad15-memory created

17. Create a role configmap-updater in the ckad17-auth1 namespace granting the update and get permissions on configmaps resources but restricted to only the ckad17-config-map instance of the resource.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3


    student-node ~ ➜  vim configmap-updater.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
    namespace: ckad17-auth1
    name: configmap-updater
    rules:
    - apiGroups: [""] # "" indicates the core API group
    resources: ["configmaps"]
    resourceNames: ["ckad17-config-map"]
    verbs: ["get", "update"]


    student-node ~ ➜  ^Configmap-updater.yaml

    student-node ~ ✖ alias kc="k create -f"

    student-node ~ ➜  kc configmap-updater.yaml 
    role.rbac.authorization.k8s.io/configmap-updater created

    student-node ~ ➜  k get cm -n ckad17-auth1 
    NAME                DATA   AGE
    ckad17-config-map   2      4m21s
    kube-root-ca.crt    1      4m21s

### SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE


19. (難題!) Identify the kube api-resources that use api_version=storage.k8s.io/v1 using kubectl command line interface and store them in /root/api-version.txt on student-node.




20. A manifest file located at root/ckad-aom.yaml on cluster3-controlplane. Which can be used to create a multi-containers pod. There are issues with the manifest file, preventing resource creation. Identify the errors, fix them and create resource.
    You can access controlplane by ssh cluster3-controlplane if required.

    student-node ~ ✖ ssh cluster3-controlplane
    Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-1072-gcp x86_64)

    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/pro

    This system has been minimized by removing packages and content that are
    not required on a system that users do not log into.

    To restore this content, you can run the 'unminimize' command.

    cluster3-controlplane ~ ➜  cat /root/ckad-aom.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    name: ckad-aom
    spec:
    containers:
        - name: nginx
        image: nginx
        volumeMounts:
            - name: flash-logs
            mountPath: /var/log
        - name: busybox
        image: busybox
        command:
            - /bin/sh
            - -c
            - sleep 10000
        volumeMounts:
            - name: flash-logs
            mountPath: /usr/src
    volumes:
        - name: flash-logs
        emptyDir: {}


    cluster3-controlplane ~ ✖ k logs ckad-aom -c nginx
    /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
    /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
    10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
    10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
    /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
    /docker-entrypoint.sh: Configuration complete; ready for start up
    nginx: [alert] could not open error log file: open() "/var/log/nginx/error.log" failed (2: No such file or directory)
    2025/02/23 00:26:07 [emerg] 1#1: open() "/var/log/nginx/error.log" failed (2: No such file or directory)

    cluster3-controlplane ~ ➜  vim /root/ckad-aom.yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: ckad-aom
    spec:
    containers:
        - name: nginx
        image: nginx
            #volumeMounts:
            #- name: flash-logs
            #  mountPath: /var/log
        - name: busybox
        image: busybox
        command:
            - /bin/sh
            - -c
            - sleep 10000
        volumeMounts:
            - name: flash-logs
            mountPath: /usr/src
    volumes:
        - name: flash-logs
        emptyDir: {}

    cluster3-controlplane ~ ✖ k replace -f /root/ckad-aom.yaml --force
    pod "ckad-aom" deleted
    pod/ckad-aom replaced

    cluster3-controlplane ~ ✖ k exec ckad-aom -c nginx -- touch /var/log/nginx/error.log

    cluster3-controlplane ~ ➜  k exec ckad-aom -c nginx -- ls /var/log/nginx
    access.log
    error.log

    cluster3-controlplane ~ ➜  k replace -f /root/ckad-aom.yaml --force
    pod "ckad-aom" deleted
    pod/ckad-aom replaced

    cluster3-controlplane ~ ➜  k logs ckad-aom -c nginx
    /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
    /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
    10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
    10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
    /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
    /docker-entrypoint.sh: Configuration complete; ready for start up
    nginx: [alert] could not open error log file: open() "/var/log/nginx/error.log" failed (2: No such file or directory)
    2025/02/23 00:33:04 [emerg] 1#1: open() "/var/log/nginx/error.log" failed (2: No such file or directory)


    改在command欄添加touch error.log文件:
    apiVersion: v1
    kind: Pod
    metadata:
    name: ckad-aom
    spec:
    containers:
        - name: nginx
        image: nginx
        command: ["/bin/sh","-c","touch /var/log/nginx/error.log"]
        volumeMounts:
            - name: flash-logs
            mountPath: /var/log
        - name: busybox
        image: busybox
        command:
            - /bin/sh
            - -c
            - sleep 10000
        volumeMounts:
            - name: flash-logs
            mountPath: /usr/src
    volumes:
        - name: flash-logs
        emptyDir: {}

    卻仍顯示找不到:
    cluster3-controlplane ~ ✖ k replace -f  /root/ckad-aom.yaml --force
    pod "ckad-aom" deleted
    pod/ckad-aom replaced

    cluster3-controlplane ~ ➜  k logs ckad-aom -c nginx
    touch: cannot touch '/var/log/nginx/error.log': No such file or directory


21. A manifest file located at root/ckad-aom.yaml on cluster3-controlplane. Which can be used to create a multi-containers pod. There are issues with the manifest file, preventing resource creation. Identify the errors, fix them and create resource.
    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3
    You can access controlplane by ssh cluster3-controlplane if required.
    
    我的配置:

    student-node ~ ➜  k apply -f /root/resourcedefinition.yaml
    error: resource mapping not found for name: "mongodbs.mongodb.com" namespace: "" from "/root/resourcedefinition.yaml": no matches for kind "CustomResourceDefinition" in version "k8s.io/v1"
    ensure CRDs are installed first

    student-node ~ ✖ k api-resources |grep crd
    customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1           false        CustomResourceDefinition


    apiVersion: apiextensions.k8s.io/ #k8s.io/v1 變更
    kind: CustomResourceDefinition
    metadata:
    name: mongodbs.mongodb.com
    spec:
    ...
