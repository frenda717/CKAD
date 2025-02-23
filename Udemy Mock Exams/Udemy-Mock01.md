First Attempt: 
Score: 35% Pass Score: 75%
Correct Answer: 03,04,08,09,13,18,21
難題: 07,10,13,14,17
1.32版本新Concept: StoarageClass, ResourceQuota, EndPoint vs. EndPointSlice (2個Object都要會), jsonPath
我尚未補齊的知識: StatefulStates, DaemonSets,Service(NodePort/ClusterIP/LoadBalander/ExternalName), Helm, CRD, Container Life Cycle

### SECTION: APPLICATION DESIGN AND BUILD

02. Create a storage class with the name banana-sc-ckad08-str as per the properties given below:
    - Provisioner should be kubernetes.io/no-provisioner,
    - Volume binding mode should be WaitForFirstConsumer.
    - **Volume expansion should be enabled**. -> 需配置allowVolumeExpansion 為true

    kubectl config use-context cluster1

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
    **allowVolumeExpansion: true** (少配置這筆)
    volumeBindingMode: WaitForFirstConsumer



07. (Container Life Cycle) In the ckad-multi-containers namespace, create a pod named tres-containers-pod, which has 3 containers matching the below requirements:
    The first container named primero runs busybox:1.28 image and has ORDER=FIRST environment variable.
    The second container named segundo runs nginx:1.17 image and is exposed at port 8080.
    The last container named tercero runs busybox:1.31.1 image and has ORDER=THIRD environment variable.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    NOTE: **All pod containers should be in the running state**.  TRICKY!!!!! 陷阱!!!!!

    我的配置:
    apiVersion: v1
    kind: Pod
    metadata:
      name: tres-containers-pod
      namespace: ckad-multi-containers
    spec:
      containers:
      - name: primero
        image: busybox:1.28
        env:
        - name: ORDER
          value: FIRST
      - name: segundo
        image: nginx:1.17
        ports:
        - containerPort: 8080
      - name: tercero
        image: busybox:1.31.1
        env:
        - name: ORDER
          value: THIRD


    為何正確答案為:
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        run: primero
      name: tres-containers-pod
      namespace: ckad-multi-containers
    spec:
      containers:
      - env:
        - name: ORDER
          value: FIRST
        image: busybox:1.28
        name: primero
        command:
        - /bin/sh
        - -c
        - **sleep 3600**;
        resources: {}
      - image: nginx:1.17
        name: segundo
        ports:
        - containerPort: 8080
        resources: {}
      - env:
        - name: ORDER
          value: THIRD
        image: busybox:1.31.1
        name: tercero
        command:
        - /bin/sh
        - -c
        - **sleep 3600**;
        resources: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    status: {}

    **IMPORTANT**: 
    在 Kubernetes 中，如果容器內執行的主進程（PID 1）結束，容器就會停止（對應的狀態很可能會顯示為 "Completed" 或 "Exited"），而不會保持在 "Running"。

    在我原本的設定中，busybox 容器沒有任何持續運行的命令。busybox 預設行為通常是執行一個短暫命令（或甚至沒有命令），結束後容器就會退出。因此，如果沒有特別指定一個長時間執行的指令，容器就無法一直保持在 "Running" 狀態。

    題目特別要求「所有 Pod 容器都必須處於 Running 狀態」。所以在「正確答案」的範例中，為了讓 busybox 容器不會立刻退出，就額外加上了 sleep 3600

    **busybox： 預設沒有常駐進程，如果不指定 command，就會在執行預設行為後結束**。
    **nginx： 以 daemon 形式啟動伺服器，預設就會常駐，因此不需要額外的 sleep 命令**。

    嚴格來說，Kubernetes 官網並不會針對每種映像 (image) 的「預設進程或行為」做詳細說明，因為那是取決於對**應映像本身的設計和 ENTRYPOINT/CMD 設定**。

    BusyBox 是一個功能精簡的工具箱映像，它預設會執行 /bin/sh，若沒有其他參數或指令就很可能很快退出。
    Nginx 官方映像則在 Dockerfile 內預設將 nginx -g daemon off; 作為 CMD，因此會在前景（foreground）運行並保持活著。
    如果想看到「為什麼 BusyBox 預設不常駐、Nginx 預設會常駐」等資訊，建議可以參考以下來源：

    Docker Hub 官方映像文件

    BusyBox 官方文件
    預設 CMD 通常是 sh (或者是空)，根據 Dockerfile 與使用方式的不同，可能執行完便退出。
    Nginx 官方文件
    預設 CMD 是 nginx -g 'daemon off;'，因此會在前景執行 Nginx。
    Kubernetes 官網 (概念性) 文件

    Define a command and arguments for a Container
    雖然這裡不會特別點名 BusyBox 或 Nginx，但它解釋了如何修改容器的 command 和 args，以及「容器主要進程 (PID 1) 結束後容器就會退出」等基本行為。
    Doc: https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/

    
    Pod Lifecycle
    也沒有針對特定映像，但可理解容器狀態如何隨著主進程退出而改變。
    所以，「BusyBox 預設沒有常駐進程、Nginx 預設在前景運行」 的根本原因，主要是因為它們各自的 Dockerfile (或 EntryPoint/CMD) 設定不同，而非 Kubernetes 的規範所導致。這部分最直接的官方來源是它們在 Docker Hub 上各自的映像介紹與 Dockerfile。

    Doc: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/


01.  (空題 Helm) On the student-node, a Helm chart repository is given under the /opt/ path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. The files have some issues. Fix those issues and deploy them with the following specifications: -

    The release name should be webapp-color-apd.
    All the resources should be deployed on the frontend-apd namespace.
    The service type should be node port.
    Scale the deployment to 3.
    Application version should be 1.20.0.
    NOTE: - Remember to make necessary changes in the values.yaml and Chart.yaml files according to the specifications, and, to fix the issues, inspect the template files.

      You can start new terminal to open student-node command.

    For this question, please set the context to cluster2 by running:

    kubectl config use-context cluster2

    
03. (配置正確) In the ckad-job namespace, create a cronjob named simple-node-job to run every 30 minutes to list all the running processes inside a container that used node image (the command needs to be run in a shell).
    
    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    **In Unix-based operating systems**, ps -eaf can be use to list all the running processes.

    k create cronjob simple-node-job -n ckad-job namespace --image=node --schedule="*/30 * * * *" --  ~~ps -eaf~~ (但答案被判斷為正確)
    (因為題目要求在一個 Shell 之中執行 ps -eaf: 故command 應寫為: -- /bin/sh -c "ps -eaf")


08. (配置正確 service??? )  We have deployed a pod pod22-ckad-svcn in the default namespace. Create a service svc22-ckad-svcn that will expose the pod at port 6335.
    
    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    Note: Use the imperative command for the above scenario.
    
    我的配置:

    kubectl expose pod pod22-ckad-svcn \
    --name=svc22-ckad-svcn \
    --port=6335 \
    (**--target-port=6335** \ )   # 當使用 kubectl expose pod 時，如果只指定了 --port 而 沒有 顯式指定 --target-port，Kubernetes 會嘗試：
                                  # 自動偵測 Pod 內部定義的 containerPort (若 Pod 規格有顯式定義 containerPort)，作為 targetPort。
                                  # 或者 直接用 和 --port 相同的值，作為 targetPort。
    --type=NodePort (題目沒有特別要求 NodePort 或 LoadBalancer，可使用預設的 ClusterIP)

    port：
    這個是指 Service 在「Cluster 內部」要對外提供的埠號。
    從集群內的其他 Pod 或 Services 來存取這個 Service 時，需要透過 port 所設定的埠號。
    
    targetPort：
    這個是指 容器（或後端應用）真正在監聽 的埠號。
    Service 會把進來的流量轉送到後端 Pod（或容器）的 targetPort


    student-node ~ ➜  k get svc
    NAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    external-webserver-ckad01-svcn   ClusterIP   172.20.19.20     <none>        80/TCP           80m
    kubernetes                       ClusterIP   172.20.0.1       <none>        443/TCP          140m
    route-apd-svc                    NodePort    172.20.185.40    <none>        8080:31993/TCP   93m
    svc22-ckad-svcn                  NodePort    172.20.211.213   <none>        6335:31348/TCP   82m


### SECTION: APPLICATION DEPLOYMENT
05. (Blue-Green Deploy) In this task, we have to create two identical environments that are running different versions of the application. 
    The team decided to use the  Blue/green deployment method to deploy a total of 10 application pods which can mitigate common risks such as downtime and rollback capability.

    Also, we have to route traffic in such a way that **30% of the traffic is sent to the green-apd environment** and **the rest is sent to the blue-apd environment**. All the development processes will happen on cluster 3 because it has enough resources for scalability and utility consumption.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

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
      Use the correct service type to **access the application from outside the cluster** and **application should listen on port** 8080. -> port 跟 target-port均設置為8080
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
    
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        type-one: blue
      name: blue-apd
    spec:
      **replicas: 7**   # 我忘記配置replicas
      selector:
        matchLabels:
          **type-one: blue** 
          **version: v1**
      template:
        metadata:
          labels:
            **version: v1**
            **type-one: blue**
        spec:
          containers:
            - image: kodekloud/webapp-color:v1
              name: blue-apd
    
    We will deploy a total of 10 application pods. Also, we have to route 70% traffic to blue-apd and 30% traffic to the green-apd deployment according to the task description.

    Since the service distributes traffic to all pods equally, we have to set the replica count 7 to the blue-apd deployment so that the given service will send ~70% traffic to the deployment pods.

    green-apd deployment should look like this: -
  
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        type-two: green
      name: green-apd
    spec:
      **replicas: 3** # 我忘記配置replicas
      selector:
        matchLabels:
          **type-two: green**
          **version: v1**
      template:
        metadata:
          labels:
            **type-two: green**
            **version: v1**
        spec:
          containers:
            - image: kodekloud/webapp-color:v2
              name: green-apd  
  
    route-apd-svc service should look like this: -

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
    kubectl create -f <FILE-NAME-1>.yaml -f <FILE-NAME-2>.yaml -f <FILE-NAME-3>.yaml (k create 指令可以同時啟用多個yaml file)

06. On cluster3, in the dev-001 namespace, one of the interns deployed one web application called news-apd.
    After successfully deploying on the worker node, we start getting alerts about the pod crashing.
    We want you to inspect the dev-001 namespace and fix those issues.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    
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
    kubectl logs -n dev-001 <POD-NAME> <CONTAINER-NAME>  # 這步我有檢察到

      Here, <POD-NAME> is the name of the Pod in which the container is running, and <CONTAINER-NAME> is the name of the container whose logs we want to retrieve. If the Pod has only one container, we can omit the <CONTAINER-NAME> argument.

      In the logs, we will see that there is a typo in the sleep command which needs to be correct. Use the kubectl edit command as follows: -
      kubectl edit deploy -n dev-001 news-apd # 考試時已修正此問題

      After fixing the typo, press the ESC button and type :wq. This command will save the changes to the file and then quit vi editor.

      Run the kubectl get command again to check the pod's status.
      kubectl get pods -n dev-001 # 已執行以上所有步驟，考試時應k get pods 再次檢查pod是否已重啟

      The pod should be running.

### SECTION: SERVICES AND NETWORKING
10. (空題 EndPoint) We have an external webserver running on student-node which is exposed at port 9999.

    We have also created a service called external-webserver-ckad01-svcn that can connect to our local webserver from within the cluster3 but, at the moment, it is not working as expected.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    Fix the issue so that other pods within cluster3 can use external-webserver-ckad01-svcn service to access the webserver.

    """在 Kubernetes 裡面，Service 本質上是用來將叢集內外的網路流量，轉發到後端的實際 Pod 或外部服務。
    然而，Service 要知道「要轉發到哪裡」時，會透過 Endpoints（或在新版 Kubernetes 中透過 EndpointSlice）來知道可用的 IP 與 Port。

    當 describe svc external-webserver-ckad01-svcn 顯示 Endpoints: <none> 時，就代表這個 Service 並沒有 綁定任何「後端目標」。

    對於「ClusterIP、NodePort、LoadBalancer 等類型的 Service」：**若 Endpoints 為空，流量就無處可去，無法連線到實際的外部或後端服務**。
    也就是說，從叢集內其他 Pod 嘗試存取這個 Service（例如 curl external-webserver-ckad01-svcn:port）時，Service 找不到任何後端 IP，最終請求會失敗或是沒有任何回應。
    因此，沒有 Endpoints 會導致：

    Service 無法完成流量轉發

    Service 是個抽象層，若底層沒有對應的 Endpoint，就沒有路徑連到最終的目標服務。
    應用程式無法使用該 Service 名稱

    即便 DNS 可以解析到 Service ClusterIP，因為沒有 Endpoints，連過去後還是沒有 Pod 或外部服務可以回應。
    整個存取路徑等同斷路

    從叢集內部看起來像是「Service 存在，卻無法對應到任何實際服務」。
    在本題中，**你的外部 webserver 是跑在 student-node:9999，但 Service 卻沒有將該 IP 與 Port 建立成 endpoints，導致 Service 無法幫忙代理或轉發流量到那個外部地址**。
    解法就是手動為這個外部服務建立 endpoints，把 student-node:9999 註冊進去。如此一來，你就能在叢集內部用 external-webserver-ckad01-svcn 來存取該外部服務。
    """
    
    
    Solution:

    Let's **check if the webserver is working or not**:
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
    **Endpoints**:         <none> # there are no endpoints for the service
    ...

    As we can see there is no endpoints specified for the service, hence we won't be able to get any output. Since we can not destroy any k8s object, **let's create the endpoint manually** for this service as shown below:

    student-node ~ ➜  export IP_ADDR=$(ifconfig eth0 | grep inet | awk '{print $2}')

    student-node ~ ➜ kubectl --context cluster3 apply -f - <<EOF
    apiVersion: v1
    kind: Endpoints
    metadata:
    /# the name here should match the name of the Service
      name: external-webserver-ckad01-svcn
    subsets:
      - addresses:
          - ip: **$IP_ADDR**
        ports:
          - port: 9999
    EOF

    (使用k explain Endpoints來查看有哪些property)

    Finally check if the curl test works now: (--rm 建立臨時debug container來測試cluster3 內的pod是否可以curl external-webserver-ckad01-svcn 服務)
    student-node ~ ➜  kubectl --context cluster3 run --rm  -i test-curl-pod --image=curlimages/curl --restart=Never -- curl -m 2 external-webserver-ckad01-svcn
    ...
    <title>Welcome to nginx!</title>
    ...


    kubectl *--context cluster3* run **--rm**  -i test-curl-pod --image=curlimages/curl **--restart=Never** -- curl -m 2 external-webserver-ckad01-svcn

    --context cluster3: 
    指定要對哪一個 Kubernetes 環境執行操作。
    如果你的系統中同時設定了多個 kubectl Context，透過 --context 可以臨時切換到特定的 context（例如 cluster3），而不需要先執行 kubectl config use-context cluster3 再執行指令。

    --rm:
    指令執行結束後會自動將此臨時 Pod 刪除（相當於 Docker 中的 --rm 參數）。
    這樣做可以避免留下不需要的 Pod 資源在叢集中，尤其常在「測試用的臨時容器」時使用。

    -restart=Never:
    預設情況下，使用 kubectl run 建立的 Pod 可能會受到重啟策略（Restart Policy）影響，例如 Always、OnFailure 等。
    設為 Never 代表這個 Pod 不會被重新啟動，一旦容器的流程結束（成功或失敗），Pod 就結束，不會自動重啟。
    配合 --rm 一起使用，通常用於一次性測試或偵錯


    -i (亦可寫成 --interactive) 的參數，主要是允許與容器互動，也就是「Attach 到容器的 STDIN」的功能。
    如果不加 -i，在有些情況下指令可能無法正確執行互動式流程，或直接結束而無法看到輸出。

    -i：表示以互動模式運行容器，並將 STDIN (標準輸入) 保持開啟。
    （常見另一個參數是 -t 或 --tty，可讓你獲得類似終端機的介面，但這裡的例子並沒有用到。）

    (補充)

    Endpoints 物件 (v1)
    角色：
    早期 Kubernetes 使用 Endpoints 物件，紀錄某個 Service 的所有後端 Pod（即 “endpoint”）的 IP 與 Port。
    資料結構限制：
    所有 endpoint 資訊都存放在同一個 Endpoints 物件中。
    若某個 Service 後端 Pod 較多，單一物件會變得很大，導致 API 伺服器在 watch、更新時的負載上升。
    可擴展性 (scalability) 限制：
    在大型叢集裡，如果某個 Service 有數百甚至數千個 Pod，Endpoints 物件會非常龐大，不容易管理與更新。

    EndpointSlice 物件 (discovery.k8s.io/v1)
    新一代設計：
    Kubernetes 從 v1.17 開始引入了 EndpointSlice，用來取代或增強現有的 Endpoints。在較新的版本中也會自動產生並使用 EndpointSlice 來進行端點資訊管理。
    分片 (Sharding) 概念：
    將同一個 Service 的所有 endpoint 拆分成多個較小的 EndpointSlice 物件（預設每個 slice 最多 100 個 endpoint，可配置）。
    這樣可以大幅減少單一物件過於龐大的問題，也減輕了 API Server 在 watch、更新時的負擔。
    可攜帶更多屬性：
    EndpointSlice 提供更靈活的結構，可以對 endpoint 進行額外標記（例如 Topology、Zone、Hostname 等）。
    也支援同時記載 TCP/UDP/其他協定的 Port 設定。
    與其他功能整合：
    像是 Service Topology、Multi-Cluster Service，也多是基於 EndpointSlice 來進行設計或延伸。

    為什麼要從 Endpoints 轉向 EndpointSlice？
    擴展性 (Scalability): 避免單一 Endpoints 物件過大、更新頻繁帶來的 API Server 負載。
    彈性 (Flexibility): EndpointSlice 結構更容易擴充功能，支援更多屬性、協定。
    細粒度管理: 可以對不同分片進行更細緻的操作或更新，而不必一次更新巨大清單。


11. Deploy a pod with name webapp-svcn using the kodekloud/webapp-color image with the label tier=msg.
    Now, Create a service webapp-service-svcn to expose the pod webapp-svcn application within the cluster on port 6379.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2



    Solution:
    On student-node, use the command kubectl run webapp-svcn --image=kodekloud/webapp-color **-l tier=msg**

    Now run the command: kubectl expose pod webapp-svcn --port=6379 --name webapp-service-svcn.

12. For this scenario, create a Service called ckad12-service that routes traffic to an external IP address.
    Please note that service should listen on port 53 and be of type ExternalName. Use the external IP address 8.8.8.8
    Create the service in the default namespace.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    Solution:

    Create the service using the following manifest:
    apiVersion: v1
    kind: Service
    metadata:
      name: ckad12-service
    spec:
      **type: ExternalName**
      **externalName: 8.8.8.8**
      ports:
        - name: http
          port: 53
          targetPort: 53


### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY

13. (配置正確) Create a pod named ckad17-qos-aecs-3 in namespace ckad17-nqoss-aecs with image nginx and container name ckad17-qos-ctr-3-aecs.
    Define other fields such that the Pod is configured to use the Quality of Service (QoS) class of Burstable.
    Also retrieve the name and QoS class of each Pod in the namespace ckad17-nqoss-aecs in the below format and save the output to a file named qos_status_aecs in the /root directory.


    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1


    Format:

    NAME    QOS
    pod-1   qos_class
    pod-2   qos_class

    我的配置" 
    echo "NAME    QOS" > /root/qos_status_aecs

    kubectl get pods -n ckad17-nqoss-aecs \
      -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.qosClass}{"\n"}{end}' \
    >> /root/qos_status_aecs

    (見: https://kubernetes.io/docs/reference/kubectl/jsonpath/)

    -o jsonpath: 告訴 kubectl 以 JSONPath 的方式輸出結果。
    JSONPath 表達式（{range .items[*]}...{end}）則告訴 kubectl 如何在回傳的 JSON 結構裡取出或遍歷我們需要的欄位。

    {range .items[*]} ... {end}
    .items: Kubernetes 以 kubectl get pods 取得的 JSON 結果中，一般會有一個 items 陣列，裡面存放所有 Pod 的資訊。
    {range .items[*]}: 表示「遍歷 .items 這個陣列中的每一個元素」。也就是「對清單裡的每一個 Pod 進行後續動作」。
    {end}: 結束整個 range（迭代）區段的標記。
    因此，{range .items[*]} ... {end} 結構中的「...」會針對清單裡的每一個 Pod執行一次

    {.metadata.name}{"\t"}{.status.qosClass}{"\n"}
    在 range 的迭代區塊中，對每個 Pod 都會輸出以下內容：

    {.metadata.name}
    透過 JSONPath 取到當前 Pod 的 .metadata.name 欄位，也就是 Pod 名稱。例如：ckad17-qos-aecs-1。
    {"\t"}
    這個是把「Tab 字元」(\t) 當作字串字面值插入輸出，用來做欄位對齊或分隔。
    {"字串"} 在 kubectl JSONPath 中可以視為「直接輸出此字串」，其中 \t 是制表符。
    {.status.qosClass}
    取得當前 Pod 的 .status.qosClass 欄位，也就是 QoS 等級（如 BestEffort、Burstable 或 Guaranteed）。
    {"\n"}
    這裡同樣是把換行字元 \n 當作「字串」插入，讓每個 Pod 的輸出結果換行

    組合起來，對每個 Pod 輸出的格式就是：
    PodName    QoSClass

    Doc: https://kubernetes.io/docs/reference/kubectl/jsonpath/

    cat /root/qos_status_aecs
    NAME    QOS
    ckad17-qos-aecs-2    BestEffort
    ckad17-qos-aecs-1    BestEffort
    ckad17-qos-aecs-3    BestEffort


    參考其他方法: (應該不會考)

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

14.  (CRD) Create a custom resource my-anime of kind Anime with the below specifications:
      Name of Anime: Death Note
      Episode Count: 37

      For this question, please set the context to cluster2 by running:
      kubectl config use-context cluster2

      TIP: You may find the respective CRD with anime substring in it.

      student-node ~ ➜  kubectl config use-context cluster2
      Switched to context "cluster2".

      Solution:

      因為題目: "Create a custom resource my-anime of kind Anime" ，故應先查詢anime resource，然後用題目定義好的anime resource,複製其properties然後用來創建my-anime物件
      查詢 CRD 列表，找出與 anime 相關的 CRD
      student-node ~ ➜  kubectl get crd | grep -i anime
      animes.animes.k8s.io

      查看 CRD 的結構與規範（尤其是 spec 部分） 這是題目給的方式，可以直接k get crd animes 然後複製欄位

      student-node ~ ➜  k get crd Anime
      Error from server (NotFound): customresourcedefinitions.apiextensions.k8s.io "Anime" not found

      student-node ~ ✖ k get crd animes.animes.k8s.io 
      NAME                   CREATED AT
      animes.animes.k8s.io   2025-02-21T16:49:40Z

      student-node ~ ➜  **kubectl get crd** animes.animes.k8s.io \
                      -o json \
                      | jq .spec.versions[].schema.openAPIV3Schema.properties.spec.properties
      {
        "**animeName**": {
          "type": "string"
        },
        "**episodeCount**": {
          "maximum": 52,
          "minimum": 24,
          "type": "integer"
        }
      }

      OR:
      student-node ~ ➜  k explain animes.spec
      GROUP:      animes.k8s.io
      KIND:       Anime
      VERSION:    v1alpha1

      FIELD: spec <Object>


      DESCRIPTION:
          <empty>
      FIELDS:
        animeName     <string>
          <no description>

        episodeCount  <integer>
          <no description>



      透過 kubectl api-resources 瞭解 Resource 名稱、ShortName 以及 Group/Version
      student-node ~ ➜  k api-resources | grep anime
      animes                            an           animes.k8s.io/v1alpha1                 true         Anime

      可得:
      Resource 名稱：animes
      ShortName：an
      Group/Version：animes.k8s.io/v1alpha1
      Kind：Anime
      
      建立自定義資源 (Custom Resource) 物件
      student-node ~ ➜  cat << YAML | kubectl apply -f -
      apiVersion: animes.k8s.io/v1alpha1
      kind: Anime
      metadata:
        name: my-anime
      spec:
        **animeName**: "Death Note"
        **episodeCount**: 37
      YAML
      anime.animes.k8s.io/my-anime created

      驗證自定義資源是否建立成功 (an 是前面看到 CRD 對應資源的 shortName。)
      student-node ~ ➜  k get an my-anime 
      NAME       AGE
      my-anime   23s

      student-node ~ ➜  k get animes.animes.k8s.io 
      NAME       AGE
      my-anime   98s

      
      注意!! 

      配置時不要配錯!:
      apiVersion: ~~apiextensions.k8s.io/v1~~
      kind: ~~CustomResourceDefinition~~ # 這是定義Anime CRD物件時會用到的kind
      metadata:
        name: my-anime
      spec:
        animeName: "Death Note"
        episodeCount: 37

      apiVersion: animes.k8s.io/v1alpha1
      kind: Anime  # 這是在anime物件已經創建好之後，使用Anime 這個kind來創建my-anime object
      metadata:
        name: my-anime
      spec:
        animeName: "Death Note"
        episodeCount: 37


15. Create a ConfigMap named ckad04-config-multi-env-files-aecs in the default namespace from the environment(env) files provided at /root/ckad04-multi-cm directory.
    
    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    Solution:

    student-node ~ ➜  kubectl config use-context cluster1
    Switched to context "cluster1".

    student-node ~ ➜  k create cm ckad04-config-multi-env-files-aecs --from-
    **--from-env-file**  (Specify the path to a file to read lines of key=val pairs to create a con…)
    --from-file      (Key file can be specified using its file path, in which case file basenam…)
    --from-literal   (Specify a key and literal value to insert in configmap (i.e. mykey=someva…)

    student-node ~ ➜  kubectl create configmap ckad04-config-multi-env-files-aecs \
            --from-env-file=/root/ckad04-multi-cm/file1.properties \
            --from-env-file=/root/ckad04-multi-cm/file2.properties
    configmap/ckad04-config-multi-env-files-aecs created

    (考試時我寫成: --from-env-file~~=key1~~=/root/ckad04-multi-cm/file1.properties  --from-env-file~~=key2~~=/root/ckad04-multi-cm/file.properties)

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

16. Create a new secret named ckad01-db-scrt-aecs with the data given below.
    Secret Name: ckad01-db-scrt-aecs
    Secret 1: DB_Host=sql01
    Secret 2: DB_User=root
    Secret 3: DB_Password=password123
    We have already deployed the required pods and services in the namespace ckad01-db-sec.
    Configure ckad01-mysql-server to load environment variables from the newly created secret, where the keys from the secret should become the environment variable name in the Pod.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3


    Solution:

    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  k get all -n ckad01-db-sec
    NAME                         READY   STATUS    RESTARTS   AGE
    pod/ckad01-mysql-server      1/1     Running   0          3m13s
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

    student-node ~ ➜  k get -n ckad01-db-sec pod ckad01-mysql-server -o yaml > ckad01-mysql-server.yaml

    student-node ~ ➜  vim ckad01-mysql-server.yaml

    student-node ~ ➜  cat ckad01-mysql-server.yaml 
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
        - **secretRef**: # 配置文件沒有，用k explain Pod.spec.containers.envFrom 搜
            name: ckad01-db-scrt-aecs

    (使用replace --force指令強制更新pod yaml file)
    student-node ~ ➜  k replace -f ckad01-db-server.yaml  (不使用--force參數如下)
    The Pod "ckad01-mysql-server" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`,`spec.initContainers[*].image`,`spec.activeDeadlineSeconds`,`spec.tolerations` (only additions to existing tolerations),`spec.terminationGracePeriodSeconds` (allow it to be set to 1 if it was previously negative)
      core.PodSpec{
            Volumes:        {{Name: "kube-api-access-72wch", VolumeSource: {Projected: &{Sources: {{ServiceAccountToken: &{ExpirationSeconds: 3607, Path: "token"}}, {ConfigMap: &{LocalObjectReference: {Name: "kube-root-ca.crt"}, Items: {{Key: "ca.crt", Path: "ca.crt"}}}}, {DownwardAPI: &{Items: {{Path: "namespace", FieldRef: &{APIVersion: "v1", FieldPath: "metadata.namespace"}}}}}}, DefaultMode: &420}}}},
            InitContainers: nil,
            Containers: []core.Container{
                    {
                            ... // 4 identical fields
                            WorkingDir: "",
                            Ports:      nil,
    -                       EnvFrom:    nil,
    +                       EnvFrom: []core.EnvFromSource{
    +                               {
    +                                       SecretRef: &core.SecretEnvSource{LocalObjectReference: core.LocalObjectReference{...}},
    +                               },
    +                       },
                            Env:       nil,
                            Resources: {},
                            ... // 15 identical fields
                    },
            },
            EphemeralContainers: nil,
            RestartPolicy:       "Always",
            ... // 28 identical fields
      }

    student-node ~ ➜  kubectl replace -f ckad01-mysql-server.yaml --force 
    pod "ckad01-mysql-server" deleted
    pod/ckad01-mysql-server replaced

    student-node ~ ➜  kubectl exec -n ckad01-db-sec ckad01-mysql-server -- printenv | egrep -w 'DB_Password=password123|DB_User=root|DB_Host=sql01'
    DB_Password=password123
    DB_User=root
    DB_Host=sql01

    egrep 是與 grep -E 等效（啟用擴展正則表示式的 grep）。
    -w 參數表示「只匹配完整的單字（word）」，也就是必須符合字串的邊界 (word boundary)，而不會只匹配部分字串。

    (直接-- printenv 即可，考試時不用這麼麻煩)

17. (ResourceQuata) Create a ResourceQuota called ckad16-rqc in the namespace ckad16-rqc-ns and enforce a limit of one ResourceQuota for the namespace.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2


    Solution:

    student-node ~ ➜  kubectl config use-context cluster2
    Switched to context "cluster2".

    student-node ~ ➜  kubectl create namespace ckad16-rqc-ns
    namespace/ckad16-rqc-ns created

    student-node ~ ➜  cat << EOF | kubectl apply -f - (直接vim 再apply 也行)
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: ckad16-rqc
      namespace: ckad16-rqc-ns
    spec:
      **hard:**
        **resourcequotas: "1"**
    EOF

    見：https://kubernetes.io/docs/concepts/policy/resource-quotas/　（object count quota table)
    resourcequota/ckad16-rqc created

    student-node ~ ➜  k get resourcequotas -n ckad16-rqc-ns
    NAME              AGE   REQUEST               LIMIT
    ckad16-rqc   20s   resourcequotas: 1/1


### SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE
19. Update the newly created pod simple-webapp-aom with a readinessProbe using the given specifications 
    Configure an HTTP readiness probe to the existing pod simple-webapp with path value set to /ready and port number to access container is 8080.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

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

20. Pod manifest file is already given under the /root/ directory called ckad-pod-busybox.yaml.
    There is error with manifest file correct the file and create resource.
  
    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1


    我的配置: 
      apiVersion: v1
      kind: Pod
      metadata:
        name: ckad-pod-busybox
      spec:
        containers:
          - ~~command: ["sleep 3600"]~~
            image: busybox
            name: pods-simple-container


    你的配置：
    command: ["sleep 3600"]
    這其實是一個陣列中只有一個字串──字串本身是 "sleep 3600"。**容器會試圖把這整個字串當成一個可執行檔的名稱**（例如 sleep 3600 這樣的檔名）。在多數情況下，系統找不到名為 sleep 3600 的執行檔，因此可能導致解析錯誤或執行失敗。

    Solution (官方/解答) 配置：
    command:
      - sleep
      - "3600"
    這表示 command 是一個陣列，第一個元素是 sleep，第二個元素是 "3600"。換句話說，系統會執行 sleep 這個指令，並傳入 3600 當作參數。
    Kubernetes（以及 Docker）對 command (或 ENTRYPOINT) 和 args 參數的解析是**「陣列每個元素都會被視為一個獨立的執行引數」**。
    因此，如果只用 ["sleep 3600"]，等於把 sleep 3600 視為一個單獨的可執行檔名稱（包含空格），也就可能導致找不到該檔案、出現 BadRequest 或執行失敗。

    而在官方解答中，
    command:
      - sleep
      - "3600"
    則是執行 sleep 指令，帶入 "3600" 做為引數，這樣就能正確執行並讓容器維持 3600 秒不退出

    也可以使用另一種寫法，例如：
    **command: ["/bin/sh", "-c", "sleep 3600"]**
    這樣相當於**在容器內執行 /bin/sh -c "sleep 3600", 同樣能達到預期效果**。
    所以問題不在於大方向設定錯，而是 command 陣列的拆分方式導致指令被誤判為一個名字(可執行檔)而非指令＋參數。

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