First Attempt: 
Score: 46% Pass Score: 75%
Correct Answer: 01,03,06,08,11,13,15,16,18,19
難題: 12 (ServiceAccount),20
1.32版本新Concept: projected volume (14)
我不熟的topic: Service Account (12) 、Endpiont (17)

### SECTION: APPLICATION DESIGN AND BUILD
02. (RestartPolicy:Never) In the ckad-pod-design namespace, start a ckad-nginx-uahsbcbdkl pod running the nginx:1.17 image.

    Configure the pod with a label:
    TRAINER: KODEKLOUD

    **The pod should not be restarted in any case if it has already exited**.  -> 題目有要求禁止pod重啟!!!

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    我的配置: 幾乎都對，但是**沒有設置restartPolicy為Never**

    Solution: 

    Create a YAML file with the content as below:

    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        TRAINER: KODEKLOUD
    name: ckad-nginx-uahsbcbdkl
    namespace: ckad-pod-design
    spec:
    containers:
    - image: nginx:1.17
        name: ckad-nginx-uahsbcbdkl
        resources: {}
    dnsPolicy: ClusterFirst
    **restartPolicy: Never**
    status: {}

    Then use kubectl apply -f file_name.yaml to create the required object.

04. (CrashLoopBackOff) In the ckad-multi-containers namespace, create a pod named cuda-pod, which has 2 containers matching the below requirements:

    The first container named alpha runs alpine image and has release=stable environment variable.
    The second container named beta runs nginx:1.17 image and is exposed at port 8080.

    NOTE: All pod containers should be in the running state.
    
    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    我的配置: (一新建立pod 便出現CrashLoopBackOff)
    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME       READY        STATUS            RESTARTS        AGE
    cuda-pod   1/2     **CrashLoopBackOff**   20 (5m2s ago)   82m

    student-node ~ ➜  k describe pod -n ckad-multi-containers cuda-pod 
    Name:             cuda-pod
    Namespace:        ckad-multi-containers
    Priority:         0
    Service Account:  default
    Node:             cluster2-node01/192.168.231.175
    Start Time:       Mon, 24 Feb 2025 23:12:26 +0000
    Labels:           run=cuda-pod
    Annotations:      <none>
    Status:           Running
    IP:               10.42.1.5
    IPs:
    IP:  10.42.1.5
    Containers:
    alpha:
        Container ID:   containerd://ede1603cf0eba5c70c685a9806cfce67bf7bac3b80ea525e757bc6449c0506f0
        Image:          alpine
        Image ID:       docker.io/library/alpine@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
        Port:           <none>
        Host Port:      <none>
        State:          Waiting
        **Reason:       CrashLoopBackOff** # cuda-pod 出現 CrashLoopBackOff，根據 kubectl describe pod 的輸出，alpha 容器的 Last State 是 "Completed"，Exit Code 為 0。
        Last State:     Terminated
        **Reason:       Completed**  # 這表示 alpha 容器**成功執行了某些任務後，就立即退出了，而 Kubernetes 會不斷嘗試重啟它，導致 CrashLoopBackOff**。
        Exit Code:    0
        Started:      Tue, 25 Feb 2025 00:34:49 +0000
        Finished:     Tue, 25 Feb 2025 00:34:49 +0000
        Ready:          False
        Restart Count:  21
        Environment:
        release:  stable
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rg8nm (ro)
    beta:
        Container ID:   containerd://f3acd35b2fcfc4816c873d8827cf1f64c99795562228964ba945299da37da7eb
        Image:          nginx:1.17
        Image ID:       docker.io/library/nginx@sha256:6fff55753e3b34e36e24e37039ee9eae1fe38a6420d8ae16ef37c92d1eb26699
        Port:           8080/TCP
        Host Port:      0/TCP
        State:          Running
        Started:      Mon, 24 Feb 2025 23:12:27 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rg8nm (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       False 
    ContainersReady             False 
    PodScheduled                True 
    Volumes:
    kube-api-access-rg8nm:
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
    Type     Reason   Age                   From     Message
    ----     ------   ----                  ----     -------
    Warning  BackOff  3m2s (x371 over 83m)  kubelet  **Back-off restarting failed container alpha** in pod cuda-pod_ckad-multi-containers(fca40a3f-fccd-4091-a95b-5c05e827029a)


    alpha 容器使用了 Alpine 映像檔：
    containers:
    - name: alpha
        image: alpine
        env:
        - name: release
            value: stable
    但 Alpine 默認的 ENTRYPOINT 只是執行一個 shell，沒有長時間運行的進程。例如：

    Alpine **容器可能只是跑了一個短命令（例如 sh），然後立即結束**。
    Pod 內的所有容器必須保持 "Running" 狀態，如果 alpha 退出，Pod 就會進入 CrashLoopBackOff。

    讓 alpha 容器保持運行，最簡單的方法是在 Alpine 容器中運行一個持久的命令，例如：
    containers:
    - name: alpha
        image: alpine
        env:
        - name: release
            value: stable
        command: **[ "sleep", "3600" ]**

    
    (補充: 如何查看Alpine image短暫執行命令便結束了??)

    方法 1：查看 Alpine 官方 Dockerfile
    Alpine 官方的 Dockerfile 可以在 GitHub 或 Docker Hub 上找到。例如：
    Docker Hub: Alpine
    Alpine 官方 GitHub: Alpine Dockerfile
    官方的 Dockerfile（簡化版本）：
    dockerfile

    FROM scratch
    ADD alpine-minirootfs-*.tar.gz /
    CMD ["/bin/sh"]
    CMD ["/bin/sh"]：這表示如果你沒有明確指定 command，容器 預設執行 /bin/sh。
    但 /bin/sh 在非交互模式下會 立即退出，因為它沒有要執行的指令。
    
    方法 2：**手動驗證 Alpine 容器行為**
    你可以直接透過 docker run 來測試：
    **docker run --rm alpine**
    這會：
    啟動一個 alpine 容器，執行預設命令 /bin/sh。
    但由於 /bin/sh 在 非交互模式 下 沒事可做，會立刻退出。
    驗證：
    **docker ps -a**  # 檢查最近的容器
    你會看到 Alpine 容器 立刻進入 Exited 狀態。

    方法 3：**檢查 Alpine 容器的 ENTRYPOINT 和 CMD** # 假如CKAD考試中有需要，建議這個方法(但只需要懂概念，因該是用不太到)
    你可以使用 docker inspect 來查看 ENTRYPOINT 和 CMD：
    **docker inspect alpine | grep -A 5 '"Cmd"'**
    預期輸出：
    "Cmd": [
        "/bin/sh"
    ],
    "ArgsEscaped": true,
    
    這確認了 Alpine 預設只執行 /bin/sh，沒有持續運行的進程，導致容器立即退出。


05. (sh 進入容器) In the ckad-pod-design namespace, we created a pod named custom-nginx that runs the nginx:1.17 image.

    Take appropriate actions to update the index.html page of this NGINX container with below value instead of default NGINX welcome page:
    Welcome to CKAD mock exams!

    NOTE: By default NGINX web server default location is at /usr/share/nginx/html which is located on the default file system of the Linux.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    我忘記進入container 使用-- sh指令即可

    Solution:

    Exec to the pod container and update the index.html file content:
    student-node ~ ➜kubectl exec -it -n ckad-pod-design custom-nginx **-- sh**
    /# echo 'Welcome to CKAD mock exams!' > /usr/share/nginx/html/index.html

    Observe the result:
    student-node ~ ➜  kubectl exec -it -n ckad-pod-design custom-nginx -- cat /usr/share/nginx/html/index.html
    Welcome to CKAD mock exams!


### SECTION: APPLICATION DEPLOYMENT
07. (Helm lint) On the student-node, a Helm chart repository is given under the /opt/ path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. 
    
    The files have some issues. Fix those issues and deploy them with the following specifications: -
    a. The release name should be webapp-color-apd.
    b. All the resources should be deployed on the frontend-apd namespace.
    c. The service type should be node port.
    d. Scale the deployment to 3.
    e. Application version should be 1.20.0.
    
    NOTE: - Remember to make necessary changes in the values.yaml and Chart.yaml files according to the specifications, and, to fix the issues, inspect the template files.

    You can start new terminal to open student-node command.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    Solution:

    Run the following command to change the context: -
    kubectl config use-context cluster2

    In this task, we will use the helm commands. Here are the steps: -
    
    **First, check the given namespace**; if it doesn't exist, we must create it first; otherwise, it will give an error "namespaces not found" while installing the helm chart.
    To check all the namespaces in the cluster2, we would have to run the following command: -
    **kubectl get ns** # b. All the resources should be deployed on the frontend-apd namespace.

    It will list all the namespaces. If the given namespace doesn't exist, then run the following command: -
    kubectl create ns frontend-apd

    Now, on the student-node node and go to the /opt/ directory. We have given the helm chart directory webapp-color-apd that contains templates, values files, and the chart file etc.

    Update the values according to the given specifications as follows: -

    Update the value of the **appVersion to 1.20.0** in the Chart.yaml file. # e. Application version should be 1.20.0.
    Update the value of the **replicaCount to 3** in the values.yaml file.  # d. Scale the deployment to 3.
    Update the value of the **type to NodePort** in the values.yaml file.  # c. The service type should be node port.

    These are the values we have to update.

    Now, we will use the **helm lint command to check the Helm chart because it can identify error**s such as missing or misconfigured values, invalid YAML syntax, and deprecated APIs etc.

    cd /opt/
    **helm lint** ./webapp-color-apd/  # helm lint 用於debug, 類似: k apply -f

    helm -h 
    Usage:
    helm [command]

    Available Commands:
    ...
    **lint        examine a chart for possible issues**

    If there is no misconfiguration, we will see the similar output: -

    helm lint ./webapp-color-apd/
    ==> Linting ./webapp-color-apd/
    [INFO] Chart.yaml: icon is recommended

    1 chart(s) linted, 0 chart(s) failed

    But in our case, there are some issues with the given templates.
        a. Deployment apiVersion needs to be correctly written. It should be apiVersion: apps/v1.
        b. In the service YAML, there is a typo in the template variable {{ .Values.service.name }} because of that, it's not able to reference the value of the name field defined in the values.yaml file for the Kubernetes service that is being created or updated.

    Now run the following command to install the helm chart in the frontend-apd namespace: -

    /# Navigate to the directory having the chart
    cd /opt
    /# Install the helm chart
    **helm install webapp-color-apd -n frontend-apd ./webapp-color-apd**

    Use the helm ls command to list the release deployed using helm.
    **helm ls -n frontend-apd**


### SECTION: SERVICES AND NETWORKING
09. We have deployed several applications in the ns-ckad17-svcn namespace that are exposed inside the cluster via ClusterIP.
    We have deployed several applications in the ns-ckad17-svcn namespace that are exposed inside the cluster via ClusterIP.
    Your task is to create a LoadBalancer type service that will **serve traffic to the applications based on its labels**. Create the resources as follows:
    Service lb1-ckad17-svcn for serving traffic at port 31890 to **pods with labels "exam=ckad, criteria=location**".
    Service lb2-ckad17-svcn for serving traffic at port 31891 to **pods with labels "exam=ckad, criteria=cpu-high**".

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    我的配置: (直接用k expose 具有labels "exam=ckad, criteria=location 的pod，便可以建立基於該pod的service。我直用yaml file生成service,卻並沒有關聯到pods )
    student-node ~ ➜  cat lb1-ckad1.yaml 
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        exam: ckad
        criteria: location
    name: lb1-ckad17-svcn
    namespace: ns-ckad17-svcn
    spec:
    ports:
    - name: "31890"
        port: 31890
        protocol: TCP
        targetPort: 31890
    selector:
        exam: ckad
        criteria: location
    type: LoadBalancer
    status:
    loadBalancer: {}


    student-node ~ ✖ cat lb2-ckad17-svcn.yaml 
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        exam: ckad
        criteria: cpu-high
    name: lb2-ckad17-svcn
    namespace: ns-ckad17-svcn
    spec:
    ports:
    - name: "31891"
        port: 31891
        protocol: TCP
        targetPort: 31891
    selector:
        exam: ckad
        criteria: cpu-high
    type: LoadBalancer
    status:
    loadBalancer: {}


    Solution:
    **To create the loadbalancer for the pods with the specified lables**, first **we need to find the pods with the mentioned lables**.

    To get pods with labels "exam=ckad, criteria=location"
    kubectl -n ns-ckad17-svcn **get pod -l exam=ckad,criteria=location**

    NAME               READY   STATUS    RESTARTS   AGE
    geo-location-app   1/1     Running   0          10m

    Similarly to get pods with labels "exam=ckad,criteria=cpu-high".
    kubectl -n ns-ckad17-svcn **get pod -l exam=ckad,criteria=cpu-high**
    
    NAME           READY   STATUS    RESTARTS   AGE
    cpu-load-app   1/1     Running   0          11m

    Now we know which pods use the labels, we can create the LoadBalancer type service using the imperative command.
    kubectl -n ns-ckad17-svcn **expose pod geo-location-app** --type=LoadBalancer --name=lb1-ckad17-svcn

    Similarly, create the another service.
    kubectl -n ns-ckad17-svcn **expose pod cpu-load-app** --type=LoadBalancer --name=lb2-ckad17-svcn

    Once the services are created, you can edit the services to use the correct nodePorts as per the question using kubectl -n ns-ckad17-svcn edit svc lb2-ckad17-svcn.

    student-node ~ ➜  k get svc -n ns-ckad17-svcn 
    NAME               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
    cpu-load-app       ClusterIP      172.20.139.168   <none>        80/TCP            75m
    geo-location-app   ClusterIP      172.20.131.144   <none>        80/TCP            75m
    lb1-ckad17-svcn    LoadBalancer   172.20.246.34    <pending>     31890:31496/TCP   67m
    lb2-ckad17-svcn    LoadBalancer   172.20.215.245   <pending>     31891:32253/TCP   68m

10. We have created a Network Policy netpol-ckad13-svcn that allows traffic only to specific pods and it allows traffic only from pods with specific labels.
    Your task is to edit the policy so that it **allows traffic from pods with labels access = allowed (這表示pod with access = allowed 為需要添加在ingress-from)**. 
    (**陷阱!!** )Do not change the existing rules in the policy. > 見Mock03-2nd Attempt

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3


    我的配置:
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
    name: netpol-ckad13-svcn
    namespace: default
    spec: 
        # 這裡要加上 **podSelector**:
                    # **matchLabels**:
                        # **app: kk-app** # 這從 k get pod -o wide --show-labels 可得 
        policyTypes:
        - Ingress
        - Egress

    ingress:
    - from:
        - podSelector:
            matchLabels:
            ~~access: allowed~~ # **應為: tier: server** # 這是原本的設置，不應該被刪除
    podSelector:
        matchLabels:
        ~~app: kk-app~~ # **應為: access: allowed** # 這是要新增的label


    
    Solution:
    To edit the existing network policy use the following command:
    kubectl edit netpol netpol-ckad13-svcn

    Edit the policy as follows:
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
    name: netpol-ckad13-svcn
    namespace: default
    spec:
        podSelector:
            matchLabels:
            app: kk-app
    policyTypes:
    - Ingress
    - Egress
    ingress:
    - from:
        - podSelector:
            matchLabels:
                tier: server
    #add the following in the manifest
        - podSelector:
            matchLabels:
                **access: allowed**

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY
12. (難題 Service Account + API Credential) We have a Kubernetes namespace called ckad12-ctm-sa-aecs, which contains a service account and a pod. Your task is to modify the pod so that it uses the service account defined in the same namespace.

    Additionally, you need to ensure that the pod has access to the API credentials associated with the service account by enabling the automounting feature for the credentials.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    Solution:
    Here we will do two things:

    **Remove automountServiceAccountToken**: false field to enable automount of creds and **specifying our service account as: serviceAccountName**: ckad12-my-custom-sa-aecs
    student-node ~ ➜  kubectl config use-context cluster2
    Switched to context "cluster2".

    student-node ~ ➜  kubectl get pods -n ckad12-ctm-sa-aecs ckad12-ctm-nginx-aecs -o yaml > ckad-custom-sa.yaml

    student-node ~ ➜  vim ckad-custom-sa.yaml 

    student-node ~ ➜  cat ckad-custom-sa.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    name: ckad12-ctm-nginx-aecs
    namespace: ckad12-ctm-sa-aecs
    spec:
    /# automountServiceAccountToken: false 不為服務帳戶自動掛載 API 憑證 *removed to enable automount of creds
    containers:
    - image: nginx
        imagePullPolicy: Always
        name: nginx
    **serviceAccountName: ckad12-my-custom-sa-aecs** # using custom sa


    student-node ~ ➜  kubectl replace -f ckad-custom-sa.yaml --force 
    pod "ckad12-ctm-nginx-aecs" deleted
    pod/ckad12-ctm-nginx-aecs replaced

    見doc: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

    這裡有兩個核心概念要理解：

    1)  **為什麼需要指定 serviceAccountName**？
    在 Kubernetes 中，每個 Pod 預設使用命名空間內的 default ServiceAccount。
    但這個題目明確要求你的 Pod 使用命名空間內 特定的 ServiceAccount (ckad12-my-custom-sa-aecs)：
    spec:
    serviceAccountName: ckad12-my-custom-sa-aecs
    透過這樣指定，Pod 就能夠使用該 ServiceAccount 綁定的權限及相應的權限配置來訪問 Kubernetes API 或其他資源。

    2) **automountServiceAccountToken 的意義**
    在 Kubernetes 中：
    預設情況下，每個 Pod 都會自動掛載 ServiceAccount 的 Token 到：
    /var/run/secrets/kubernetes.io/serviceaccount/token
    如果**設置 automountServiceAccountToken: false，會阻止 Pod 掛載這個 token**，導致 Pod 無法使用這個 ServiceAccount 提供的身份與 Kubernetes API 通訊。

    因此：
    automountServiceAccountToken: false 的作用是停用自動掛載 ServiceAccount Token。
    而在本題中，你需要明確讓 Pod 使用 指定的 ServiceAccount，也就是需要該 ServiceAccount 提供的權限及其 Token。




14. (Projected Volume) In the ckad14-sa-projected namespace, configure the ckad14-api-pod Pod to include a projected volume named vault-token.

    Mount the service account token to the container at /var/run/secrets/tokens, with an expiration time of 7000 seconds.

    Additionally, set the intended audience for the token to vault and path to vault-token.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3
    我的配置:

    spec:
    containers:
    - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: ~~/var/run/secrets/kubernetes.io/serviceaccount~~ # 應設置為: /var/run/secrets/tokens
        name: vault-token
        readOnly: true
    ....
    securityContext: {}
    serviceAccount: ckad14-sa
    serviceAccountName: ckad14-sa
    ....
    volumes:
      - name: vault-token # 這塊應直接新增
        projected:
          defaultMode: 420
          sources:
          - serviceAccountToken:
              expirationSeconds: 7000
              #path: ~~/var/run/secrets/tokens~~ # volume的path路徑是在volumeMounts內設置，而非此處
              **path: vault-token**
              **audience: vault # 沒有設置這項**
          - configMap:
              items:
              - key: ca.crt
                path: ca.crt
              name: kube-root-ca.crt
          - downwardAPI:
              items:
              - fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
                path: namespace


    
    Solution:

    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  k get pod -n ckad14-sa-projected ckad14-api-pod -o yaml > ckad-pro-vol.yaml

    student-node ~ ➜  cat ckad-pro-vol.yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: ckad14-api-pod
    namespace: ckad14-sa-projected 
    spec:
    containers:
    - image: nginx
        imagePullPolicy: Always
        name: nginx
    .
    .
    .
    volumeMounts:                              # Added
        - mountPath: /var/run/secrets/tokens       # Added
        name: vault-token                        # Added
    .
    .
    .
    serviceAccount: ckad14-sa
    serviceAccountName: ckad14-sa
    volumes:
    - name: vault-token                   # Added
        projected:                          # Added
        sources:                          # Added
        - serviceAccountToken:            # Added
            path: vault-token             # Added
            expirationSeconds: 7000       # Added
            audience: vault               # Added

    student-node ~ ➜  k replace -f ckad-pro-vol.yaml --force 
    pod "ckad14-api-pod" deleted
    pod/ckad14-api-pod replaced



### SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE
17. (EndPoint) A pod named ckad-nginx-pod-aom is deployed and exposed with a service ckad-nginx-service-aom, but it seems the service is not configured properly and is not selecting the correct pod.
    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    Make the required changes to service and ensure the endpoint is configured for service.
    Check for service endpoint by using kubectl describe svc ckad-nginx-service-aom

    Solution:
    we can see below output as below

    kubectl describe svc ckad-nginx-service-aom
    Name:              ckad-nginx-service-aom
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          app=ngnix
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                10.43.134.231
    IPs:               10.43.134.231
    Port:              80-80  80/TCP
    TargetPort:        80/TCP
    **Endpoints:         <none>**
    Session Affinity:  None
    Events:            <none>

    doc:
    Endpoint:  https://kubernetes.io/docs/reference/kubernetes-api/service-resources/endpoints-v1/
    Debug Service: https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/
    
    
    we can see there endpoint value as none. Let's debug this we can see selector as app=ngnix . Lets check label value in pod.

    解决方案调试步骤
    1. 检查 Service 选择器是否正确
    kubectl get svc ckad-nginx-service-aom -o yaml | grep selector

    2. **检查 Pod 的 Labels 是否匹配**
    **kubectl get pod ckad-nginx-pod-aom --show-labels**

    OR 
    kubectl get pod ckad-nginx-pod-aom -o json | jq -r .metadata.labels
    "app": "nginx"

    3. 编辑 Service 并修正 selector
    kubectl edit svc ckad-nginx-service-aom
    /# 修改 `selector` 部分，使其与 Pod 标签匹配

    4. 确保 Endpoint 正常创建
    kubectl get endpoints ckad-nginx-service-aom



    kubectl get pod ckad-nginx-pod-aom -o json | jq -r .metadata.labels
    "app": "nginx"

    we can see here selector is mis-spelled in service. so edit service and check for endpoint value.

    kubectl get ep ckad-nginx-service-aom
    NAME                     ENDPOINTS                   AGE
    ckad-nginx-service-aom   10.42.2.4:80,10.42.2.5:80   4m44s


18. View the metrics (CPU and Memory) of the node cluster2-node01 and copy the output to the /root/node-metrics file in clustername,CPU and memory.

    View the metrics (CPU and Memory) of the node cluster2-node01 and copy the output to the /root/node-metrics file in clustername,CPU and memory.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    student-node ~ ➜  k describe node cluster2-node01 |grep -A6 "Capacity:"
    Capacity:
    cpu:                32
    ephemeral-storage:  1546531076Ki
    hugepages-1Gi:      0
    hugepages-2Mi:      0
    memory:             131992568Ki
    pods:               110

    student-node ~ ➜  k describe node cluster2-node01 |grep -A6 "Allocatable:"
    Allocatable:
    cpu:                32
    ephemeral-storage:  1504465429553
    hugepages-1Gi:      0
    hugepages-2Mi:      0
    memory:             131992568Ki
    pods:               110


    student-node ~ ✖ k get nodes
    NAME                    STATUS   ROLES                  AGE    VERSION
    cluster2-controlplane   Ready    control-plane,master   158m   v1.29.0+k3s1
    cluster2-node01         Ready    <none>                 158m   v1.29.0+k3s1

    student-node ~ ➜  k get nodes cluster2-node01 ^C

    student-node ~ ✖ echo "cluster2-node01" > /root/node-metrics

    student-node ~ ➜  k describe node cluster2-node01 |grep -A6 "Capacity:" >> /root/node-metrics 

    student-node ~ ➜  k describe node cluster2-node01 |grep -A6 "Allocatable:" >> /root/node-metrics 

    student-node ~ ➜  cat /root/node-metrics 
    cluster2-node01
    Capacity:
    cpu:                32
    ephemeral-storage:  1546531076Ki
    hugepages-1Gi:      0
    hugepages-2Mi:      0
    memory:             131992568Ki
    pods:               110
    Allocatable:
    cpu:                32
    ephemeral-storage:  1504465429553
    hugepages-1Gi:      0
    hugepages-2Mi:      0
    memory:             131992568Ki
    pods:               110



### SECTION: SERVICE NETWORKING

20. (難題) Part I: Create a ClusterIP service .i.e. service-3421-svcn in the spectra-1267 ns which should expose the pods namely pod-23 and pod-21 with port set to 8080 and targetport to 80.

    Part II: Store the pod names and their ip addresses from the spectra-1267 ns at /root/pod_ips_cka05_svcn where the output is sorted by their IP's.

    Please ensure the format as shown below:
    POD_NAME        IP_ADDR
    pod-1           ip-1
    pod-3           ip-2
    pod-2           ip-3
    ...

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3


    Solution:

    Switching to cluster3:
    kubectl config use-context cluster3

    Part I: 
    **The easiest way to route traffic to a specific pod is by the use of labels and selectors** . List the pods along with their labels:

    student-node ~ ➜  kubectl get pods **--show-labels** -n spectra-1267
    NAME     READY   STATUS    RESTARTS   AGE     LABELS
    pod-12   1/1     Running   0          5m21s   env=dev,mode=standard,type=external
    pod-34   1/1     Running   0          5m20s   env=dev,mode=standard,type=internal
    pod-43   1/1     Running   0          5m20s   env=prod,mode=exam,type=internal
   **pod-23**   1/1  Running   0          5m21s   env=dev,mode=exam,type=external
    pod-32   1/1     Running   0          5m20s   env=prod,mode=standard,type=internal
    **pod-21**   1/1 Running   0          5m20s   env=prod,mode=exam,type=external

    Looks like there are a lot of pods created to confuse us. But we are only concerned with the labels of pod-23 and pod-21.

    As we can see both the required pods have labels mode=exam,type=external in common. Let's confirm that using kubectl too:
    student-node ~ ➜  kubectl get pod **-l mode=exam,type=external** -n spectra-1267                                    
    NAME     READY   STATUS    RESTARTS   AGE
    pod-23   1/1     Running   0          9m18s
    pod-21   1/1     Running   0          9m17s


    Nice!! Now as we have figured out the labels, we can proceed further with the creation of the service:
    student-node ~ ➜  kubectl create service clusterip service-3421-svcn -n spectra-1267 --tcp=8080:80 --dry-run=client -o yaml > service-3421-svcn.yaml

    Now modify the service definition with selectors as required before applying to k8s cluster:
    student-node ~ ➜  cat service-3421-svcn.yaml 
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        app: service-3421-svcn
    name: service-3421-svcn
    namespace: spectra-1267
    spec:
    ports:
    - name: 8080-80
        port: 8080
        protocol: TCP
        targetPort: 80
    selector:
        ~~app: service-3421-svcn~~  # delete 
        **mode: exam**    # add
        **type: external**  # add
    type: ClusterIP
    status:
    loadBalancer: {}


    Finally let's apply the service definition:

    student-node ~ ➜  kubectl apply -f service-3421-svcn.yaml
    service/service-3421 created

    student-node ~ ➜  k get ep service-3421-svcn -n spectra-1267
    NAME           ENDPOINTS                     AGE
    service-3421   10.42.0.15:80,10.42.0.17:80   52s

    Part II: 
    To store all the pod name along with their IP's , we could use imperative command as shown below:

    student-node ~ ➜  kubectl get pods -n spectra-1267 **-o=custom-columns**='**POD_NAME:metadata.name**,**IP_ADDR:status.podIP**' --sort-by=.status.podIP

    POD_NAME   IP_ADDR
    pod-12     10.42.0.18
    pod-23     10.42.0.19
    pod-34     10.42.0.20
    pod-21     10.42.0.21
    ...

    /# store the output to /root/pod_ips
    student-node ~ ➜  kubectl get pods -n spectra-1267 -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP > /root/pod_ips_cka05_svcn


