## Certification Domains
Application Design and Build (20%)
Define, build and modify container images
Choose and use the right workload resource (Deployment, DaemonSet, CronJob, etc.)
Understand multi-container Pod design patterns (e.g. sidecar, init and others)
Utilize persistent and ephemeral volumes

Application Deployment (20%)
Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)
Understand Deployments and how to perform rolling updates
Use the Helm package manager to deploy existing packages
Kustomize

Application Observability and Maintenance (15%)
Understand API deprecations
Implement probes and health checks
Use built-in CLI tools to monitor Kubernetes applications
Utilize container logs
Debugging in Kubernetes

Application Environment, Configuration and Security (25%)
Discover and use resources that extend Kubernetes (CRD, Operators)
Understand authentication, authorization and admission control
Understand Requests, limits, quotas
Understand ConfigMaps
Create & consume Secrets
Understand ServiceAccounts
Understand Application Security (SecurityContexts, Capabilities, etc.)

Services and Networking (20%)
Demonstrate basic understanding of NetworkPolicies
Provide and troubleshoot access to applications via services
Use Ingress rules to expose applications



## Certification Tip

1. Imperative Commands
    
        --dry-run

    By default (Without dry-run), as soon as the command is run, the resource will be created. If you simply want to test your command, use the: 
    
        --dry-run=client
    
    使用--dry-run=client 來執行debug，較果如下: 
    controlplane ~ ➜  kubectl apply -f app.yaml --dry-run=client
    error: error parsing app.yaml: error converting YAML to JSON: yaml: line 24: did not find expected key

        
    This will **not create the resource**. Instead, tell you whether the resource can be created and if your command is right.

        
        -o yaml
        
    This will output the resource definition in YAML format on the screen.

    Combine:
    
        kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml

    Examples:
    1) Pods: 
      Generate POD Manifest YAML file (-o yaml) but don't create it(--dry-run)

            kubectl run nginx --image=nginx --dry-run=client -o yaml

    2) Deployment: 
      Generate Deployment YAML file (-o yaml) but don't create it(--dry-run)

            kubectl create deployment --image=nginx nginx --dry-run -o yaml

      Generate Deployment with 4 Replicas
      
            kubectl create deployment nginx --image=nginx --replicas=4
      
      (Can also scale deployment using the "kubectl scale" command)
        
            kubectl scale deployment nginx --replicas=4
      
      (Or can save the YAML definition to a file and modify)
        
            kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
   
    3) Service:
    i. Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379 
   
        kubectl expose pod redis --port=6379 --name redis-service -o yaml
        
       (This will automatically use the pod's labels as selectors)
       
       Or

        kubectl create service clusterip redis --tcp=6379:6379 -o yaml 
            
       (This will not use the pods' labels as selectors; instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So **it does not work well** if your **pod has a different label set**. So generate the file and modify the selectors before creating the service)
   
    ii.  Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes: (Recommended)

        kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml

       (This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

    Or

        kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml

       (This will not use the pods' labels as selectors)

    Both the above commands have their own challenges. While one of it **cannot accept a selector** the other **cannot accept a node port**. Would recommend going with the `**kubectl expose**` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.


    總結：
        
    選擇器處理：
        kubectl expose：自動使用 pod 的標籤作為選擇器，簡化了 Service 與 pod 的關聯。
        kubectl create service：默認使用 app=redis 或 app=nginx 作為選擇器，若 pod 標籤不同需手動調整。
    
        
    節點埠處理：
        kubectl expose：無法直接指定 nodePort，需要生成 YAML 文件後手動新增所需的 nodePort。
        kubectl create service：允許直接指定 nodePort，但需要手動調整選擇器以匹配 pod 的標籤。
    
    建議：
    a優先使用 kubectl expose，因為它會自動選擇與 pod 匹配的標籤作為選擇器。如果需要指定特定的 nod



    Practice:

    4) Deploy a pod named nginx-pod using the nginx:alpine image.
       Use imperative commands only.

        controlplane ~ ➜  k run nginx-pod --image=nginx:alpine
        pod/nginx-pod created

        controlplane ~ ➜  k get pods
        NAME        READY   STATUS    RESTARTS   AGE
        nginx-pod   1/1     Running   0          3s

    5) Deploy a redis pod using the redis:alpine image with the labels set to tier=db. (2 ways)
        Either use imperative commands to create the pod with the labels. Or else use imperative commands to generate the pod definition file, then add the labels before creating the pod using the file.
       
        Selectors & Labels 用法: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/

        a. Use imperative commands to create the pod with the labels
                    
            k run redis --image=redis:alpine --labels="tier=db" (-l tier=db) OR kubectl run redis -l tier=db --image=redis:alpine

        controlplane ~ ➜  k run redis --image=redis:alpine --tier=db
        error: unknown flag: --tier
        See 'kubectl run --help' for usage.

        controlplane ~ ✖ kubectl run redis --image=redis:alpine --labels="tier=db"
        pod/redis created

        controlplane ~ ➜  k get pods
        NAME        READY   STATUS    RESTARTS   AGE
        nginx-pod   1/1     Running   0          32m
        redis       1/1     Running   0          5s


        OR

        b. Use imperative commands to generate the pod definition file, then add the labels before creating the pod using the file

            kubectl run redis --image=redis:alpine --dry-run=client -oyaml > redis-pod.yaml
            vim redis-pod.yaml OR k edit pod redis
        將labels column 替換為:
              labels:
                tier: db
            name: redis
            namespace: default

        controlplane ~ ➜  k edit pod redis
        A copy of your changes has been stored to "/tmp/kubectl-edit-329040364.yaml" ...
                
            kubectl create -f redis-pod.yaml OR k edit pod redis 

    3)  Create a service redis-service to expose the redis application within the cluster on port 6379.
        Use imperative commands.

        Service: redis-service
        Port: 6379
        Type: ClusterIP

            k run redis-pod --image=redis:alpine
            k expose pod redis-pod --port=6379 (default: --type=ClusterIP) (deploy then expose)
        controlplane ~ ✖ k run redis-pod --image=redis:alpine
        pod/redis-pod created

        controlplane ~ ➜  k expose pod redis-pod --port=6379
        service/redis-pod exposed


    4)   Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas.
        Try to use imperative commands only. Do not create definition files.
        Name: webapp
        Image: kodekloud/webapp-color
        Replicas: 3    

            kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3  

        See 2) Deployment.
        
    5)   Create a new pod called custom-nginx using the nginx image and run it on container port 8080.

            k run custom-nginx --image=nginx --port=8080

        ( --port 不會暴露外部訪問，如果需要讓其他資源或用戶訪問該 Pod，需要使用 Service 暴露該埠，例如：)

            kubectl expose pod custom-nginx --type=ClusterIP --port=80 --target-port=8080
        
        controlplane ~ ➜  k run custom-nginx --image=nginx --port=8080
        pod/custom-nginx created

        controlplane ~ ➜  k get pods
        NAME                      READY   STATUS             RESTARTS        AGE
        custom-nginx              1/1     Running            0               3s
        nginx-pod                 1/1     Running            0               29m
        pod                       0/1     CrashLoopBackOff   10 (4m4s ago)   30m
        redis-pod                 1/1     Running            0               18m
        webapp-58bc75696f-7v9zj   1/1     Running            0               4m21s
        webapp-58bc75696f-kc7nr   1/1     Running            0               4m21s
        webapp-58bc75696f-kvhxz   1/1     Running            0               4m21s

        controlplane ~ ✖ kubectl describe pod custom-nginx| grep Port
        Port:           8080/TCP
        Host Port:      0/TCP


    6)   Create a new namespace called dev-ns.
        Use imperative commands.

            k create namespace dev-ns

        controlplane ~ ➜  k create namespace dev-ns
        namespace/dev-ns created

        controlplane ~ ➜  k get ns
        NAME              STATUS   AGE
        default           Active   45m
        dev-ns            Active   3s
        kube-node-lease   Active   45m
        kube-public       Active   45m
        kube-system       Active   45m

    7) Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas.
        Use imperative commands.
        
        controlplane ~ ✖ k create deployment redis-deploy --image=redis --replicas=2 -ns=dev-ns
        error: failed to create deployment: namespaces "s=dev-ns" not found    (**Use --namespace or -n**)

        controlplane ~ ✖ k create deployment redis-deploy --image=redis --replicas=2 --namespace=dev-ns
        deployment.apps/redis-deploy created

        controlplane ~ ➜  k get deploy/deployment
        NAME     READY   UP-TO-DATE   AVAILABLE   AGE
        webapp   3/3     3            3           19m

        controlplane ~ ➜  k get deploy -n=dev-ns
        NAME           READY   UP-TO-DATE   AVAILABLE   AGE
        redis-deploy   2/2     2            2           26s


    8)    Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type       
        ClusterIP by the same name (httpd). The target port for the service should be 80.
        Try to do this with as few steps as possible.

        a. Create Pod then Create Service for this pod
        
            kubectl run httpd --image=httpd:alpine --port=80
            kubectl expose pod httpd --type=ClusterIP --port=80 --target-port=80

        controlplane ~ ✖ kubectl run httpd02 --image=httpd:alpine --port=80
        pod/httpd02 created

        controlplane ~ ➜  kubectl expose pod httpd02 --type=ClusterIP --port=80 --target-port=80
        service/httpd02 exposed



        b. Create Pod & Service in 1 step

            k run httpd --image=httpd:alpine --port=80 --expose

        controlplane ~ ➜  k run httpd --image=httpd:alpine --port=80 --expose
        service/httpd created
        pod/httpd created

        controlplane ~ ➜  k get poods
        error: the server doesn't have a resource type "poods"

        controlplane ~ ✖ k get pods
        NAME                      READY   STATUS             RESTARTS         AGE
        custom-nginx              1/1     Running            0                19m
        httpd                     1/1     Running            0                13s


    See more in: https://kubernetes.io/docs/reference/kubectl/conventions/



2. Linux Commnad:

    If you paste a block of code from the doc but the fomat did'nt change but the indentation is not automatically aligned:
    command:

        press Shift + V  then scorll down to select the columns you want by (Shift +V 的時候，屬標需要在欲選取的第一行，接著放掉後，用PgDn 鍵往下選取)
        倘若按到數字鍵盤的dot(也就是delete鍵，將會全數刪除!)

        press Shift + dot(鍵盤裡的: > 鍵)
        >：將選中的行向右縮排。
        <：將選中的行向左縮排 

    Jump to the top or bottom of the page:

        gg: jump to the top
        G: jump to the bottom
        M: 跳至當前畫面的中間行

    Find a specific word:

        /keyword + Enter: Search for keywords downward from the current cursor position (當前光標以上)
        ?keyword + Enter: Search for keywords upward from the current cursor position (當前光標以下)

        n: jump to the next keyword
        N: jump to the previous keyword

    Highlight all keywords:

        /keyword + Enter: search first
        :set hlsearch: highlight
        :noh : remobe highlight
    
    Alternative keywords:

        :%s/old_keyword/new_keyword/g

        %: Represents the entire file
        s: Indicates a replacement operation
        g: Indicates global replacement, that is, replace all matching keywords in each line


    Ask for confimation for each keyword: 

        y/n/a/q/l/^E/^Y
    
        Enter y: Replace the current match.
        Enter n: Skip the current match.
        Enter a: Replace all remaining matches.
        Type q: Exit replace mode.

        apiVersion: v1
        kind: Pod
        metadata:
        creationTimestamp: "2025-01-09T02:15:44Z"
        labels:
            name: test
        name: test
        namespace: elastic-stack
        resourceVersion: "582"
        uid: 5f31ab29-557b-41a3-8593-ccf80714be10
        spec:
        containers:
        - name: test
            image: kodekloud/event-simulator
            imagePullPolicy: Always
            resources: {}
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
            volumeMounts:
            - mountPath: /log
            name: log-volume
            - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
            name: kube-api-access-f65df
            readOnly: true
        - name: sidecar
            image: kodekloud/filebeat-configured
            volumeMounts:
        replace with app (y/n/a/q/l/^E/^Y)? 
        
        5 substitutions on 5 lines  (提示總共做了哪幾個變更)    

    在 Vim 中启用自动缩进：

        :set ai
    OR 直接启用 YAML 语法缩进：

        :set filetype=yaml

    确保使用合适的空格：
        
        :set tabstop=2 shiftwidth=2 expandtab
    这将确保 YAML 文件中的缩进使用 2 个空格，而不是 Tab。

3. Setting alias:

        alias k="kubectl"
        alias ka="kubectl apply -f"
        alias ka= "kubectl apply" (設置了新的alias, 將會覆蓋原有的)
        alias kg="kubectl get"
        alias kd="kubectl describe"
        alias ke="kubectl edit"

    示範如下:
    controlplane ~ ➜ alias ka= "kubectl apply" 
    controlplane ~ ➜  ka -f app.yaml --namespace=elastic-stack
    error: error parsing app.yaml: error converting YAML to JSON: yaml: line 24: did not find expected key    

    OR
    controlplane ~ ➜ alias ka="kubectl apply -f"
    controlplane ~ ➜  ka app.yaml --namespace=elastic-stack

    View all aliases:

        alias
        alias <alias-anme>
            eg. alias ka, alias ke

    查看alias 的來源:

        grep 'alias' ~/.bashrc ~/.zshrc /etc/bashrc /etc/profile

    通過grep 快速查找以上有可能存放alias 配置的位置


    Tips tutor gave:
    1) https://www.linkedin.com/pulse/my-ckad-exam-experience-atharva-chauthaiwale/

    2) https://medium.com/@harioverhere/ckad-certified-kubernetes-application-developer-my-journey-3afb0901014

        **Resist the urge to answer the questions sequentially.**

    3) https://github.com/lucassha/CKAD-resources



4. 官網文件找不到的終極大法:

    k explain <resource> --recursive 
    k explain <resource> --recursive |grep <colum options>
    k explain <resource>.spec.hostpath

    controlplane ~ ➜  k create -f log-volume.yaml 
    The PersistentVolume "log-volume" is invalid: spec: Required value: must specify a volume type


    controlplane ~ ➜  kubectl explain persistentvolume --recursive |grep -A5 hostPath
        hostPath    <HostPathVolumeSource>
        path      <string> -required-
        type      <string>
        enum: "", BlockDevice, CharDevice, Directory, ....
        iscsi       <ISCSIPersistentVolumeSource>
        chapAuthDiscovery <boolean>


    controlplane ~ ➜  vim log-volume.yaml 

        apiVersion: v1
        kind: PersistentVolume
        metadata:
        name: log-volume
        spec:
            capacity:
                storage: 1Gi
            volumeMode: Filesystem
            accessModes:
                - ReadWriteMany
            persistentVolumeReclaimPolicy: Retain
            storageClassName: manual

            hostPath: (用expain指令查詢完畢之後，變得知需hostPath下的path 為必須設置的選項)
                path: /opt/volume/nginx



    controlplane ~ ➜  k explain deploy.spec.template.spec.containers.resources --recursive
    GROUP:      apps
    KIND:       Deployment
    VERSION:    v1

    FIELD: resources <ResourceRequirements>


    DESCRIPTION:
        Compute Resources required by this container. Cannot be updated. More info:
        https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
        ResourceRequirements describes the compute resource requirements.
        
    FIELDS:
    claims        <[]ResourceClaim>
        name        <string> -required-
        request     <string>
    limits        <map[string]Quantity>
    **requests**      <map[string]Quantity>


    controlplane ~ ✖ k explain deploy.spec.template.spec.containers.resources.requests --recursive
    GROUP:      apps
    KIND:       Deployment
    VERSION:    v1

    FIELD: requests <map[string]Quantity>


    DESCRIPTION:
        Requests describes the minimum amount of compute resources required. If
        Requests is omitted for a container, it defaults to Limits if that is
        explicitly specified, otherwise to an implementation-defined value. Requests
        cannot exceed Limits. More info:
        https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
        Quantity is a fixed-point representation of a number. It provides convenient
        marshaling/unmarshaling in JSON and YAML, in addition to String() and
        AsInt64() accessors.
        
        The serialization format is:
        
        ``` <quantity>        ::= <signedNumber><suffix>
        
            (Note that <suffix> may be empty, from the "" case in <decimalSI>.)
        
        <digit>           ::= 0 | 1 | ... | 9 <digits>          ::= <digit> |
        <digit><digits> <number>          ::= <digits> | <digits>.<digits> |
        <digits>. | .<digits> <sign>            ::= "+" | "-" <signedNumber>    ::=
        <number> | <sign><number> <suffix>          ::= <binarySI> |
        <decimalExponent> | <decimalSI> <binarySI>        ::= Ki | Mi | Gi | Ti | Pi
        | Ei
        
            (International System of units; See:
        http://physics.nist.gov/cuu/Units/binary.html)
        
        <decimalSI>       ::= m | "" | k | M | G | T | P | E
        
            (Note that 1024 = 1Ki but 1000 = 1k; I didn't choose the capitalization.)
        
        <decimalExponent> ::= "e" <signedNumber> | "E" <signedNumber> ```
        
        No matter which of the three exponent forms is used, no quantity may
        represent a number greater than 2^63-1 in magnitude, nor may it have more
        than 3 decimal places. Numbers larger or more precise will be capped or
        rounded up. (E.g.: 0.1m will rounded up to 1m.) This may be extended in the
        future if we require larger or smaller quantities.
        
        When a Quantity is parsed from a string, it will remember the type of suffix
        it had, and will use the same type again when it is serialized.
        
        Before serializing, Quantity will be put in "canonical form". This means
        that Exponent/suffix will be adjusted up or down (with a corresponding
        increase or decrease in Mantissa) such that:
        
        - No precision is lost - No fractional digits will be emitted - The exponent
        (or suffix) is as large as possible.
        
        The sign will be omitted unless the number is negative.
        
        Examples:
        
        - 1.5 will be serialized as "1500m" - 1.5Gi will be serialized as "1536Mi"
        
        Note that the quantity will NEVER be internally represented by a floating
        point number. That is the whole point of this exercise.
        
        Non-canonical values will still parse as long as they are well formed, but
        will be re-emitted in their canonical form. (So always use canonical form,
        or don't diff.)
        
        This format is intended to make it difficult to use these numbers without
        writing some sort of special handling code in the hopes that that will cause
        implementors to also use a fixed point implementation.

5. --restart=Never 建立debug pod:
   
    如下，倘若pod 因不明原因導致Error或是崩潰 CrashLoopBackOff，可建立debug pod 來進入容器內，進一步排查問題。
    可以嘗試以 **--restart=Never** 的方式 **手動啟動一個新的 Debug Pod**。

    controlplane ~ ➜  kubectl exec -it time-check -n dvl1987 -- env | grep TIME_FREQ
    error: Internal error occurred: unable to upgrade connection: container not found ("time-check")


    建立debug pod 來排查原因:
            
    kubectl run debug-shell --rm -i -t --image=busybox --restart=Never --namespace=dvl1987 -- sh

    進入容器 intit mode:
    controlplane ~ ✖ kubectl run debug-shell --rm -i -t --image=busybox --restart=Never --namespace=dvl1987 -- sh
    If you don't see a command prompt, try pressing enter.
    / # env | grep TIME_FREQ (檢查pod是否正確傳遞環境變量)
    / # 


    倘若此方法有限，則使用 kubectl logs 來查看崩潰日誌
    
    kubectl logs time-check -n dvl1987

6. 適合使用k replace的場合
    controlplane ~ ✖ k apply -f redis-deploy.yaml 
    Warning: resource deployments/redis is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
    Error from server (Conflict): error when applying patch:
    {"metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"apps/v1\",\"kind\":\"Deployment\",\"metadata\":{\"annotations\":{},\"creationTimestamp\":\"2025-02-05T22:47:31Z\",\"generation\":1,\"labels\":{\"app\":\"redis\"},\"name\":\"redis\",\"namespace\":\"default\",\"resourceVersion\":\"5748\",\"uid\":\"8137484f-04b9-47f1-9817-400d439cbe15\"},\"spec\":{\"progressDeadlineSeconds\":600,\"replicas\":1,\"revisionHistoryLimit\":10,\"selector\":{\"matchLabels\":{\"app\":\"redis\"}},\"strategy\":{\"rollingUpdate\":{\"maxSurge\":\"25%\",\"maxUnavailable\":\"25%\"},\"type\":\"RollingUpdate\"},\"template\":{\"metadata\":{\"creationTimestamp\":null,\"labels\":{\"app\":\"redis\"}},\"spec\":{\"containers\":[{\"image\":\"redis:alpine\",\"imagePullPolicy\":\"IfNotPresent\",\"name\":\"redis\",\"ports\":[{\"containerPort\":6379}],\"resources\":{\"requests\":{\"cpu\":\"0.2\"}},\"terminationMessagePath\":\"/dev/termination-log\",\"terminationMessagePolicy\":\"File\",\"volumeMounts\":[{\"mountPath\":\"/redis-master-data\",\"name\":\"data\"}]}],\"dnsPolicy\":\"ClusterFirst\",\"restartPolicy\":\"Always\",\"schedulerName\":\"default-scheduler\",\"securityContext\":{},\"terminationGracePeriodSeconds\":30,\"volumes\":null}}},\"status\":{}}\n"},"resourceVersion":"5748"},"spec":{"template":{"spec":{"$setElementOrder/containers":[{"name":"redis"}],"containers":[{"name":"redis","ports":[{"containerPort":6379}],"resources":{"requests":{"cpu":"0.2"}},"volumeMounts":[{"mountPath":"/redis-master-data","name":"data"}]}],"volumes":null}}}}
    to:
    Resource: "apps/v1, Resource=deployments", GroupVersionKind: "apps/v1, Kind=Deployment"
    Name: "redis", Namespace: "default"
    for: "redis-deploy.yaml": error when patching "redis-deploy.yaml": Operation cannot be fulfilled on deployments.apps "redis": the object has been modified; please apply your changes to the latest version and try again


    **什麼時候需要更新 Deployment 卻不刪除 Pod？**
    通常，當我們更新 Kubernetes **Deployment** 時，Pod 會根據 **Rolling Update** 策略逐步重新啟動。但某些情況下，我們希望 **只更新 Deployment 定義，而不影響現有的 Pod**。這時可以使用 `kubectl replace` 來 **更新 Deployment 本身，但不立即影響 Pod**。

    ---

    **✅ 什麼情況下需要這樣做？**
    **1️⃣ 更新 Deployment Metadata，但不影響 Pod**
    如果你只想更新 Deployment 的 **標籤 (Labels)**、**註釋 (Annotations)** 或 **其他不影響 Pod 的設定**，你可以使用：
    ```sh
    kubectl replace -f redis-deploy.yaml
    ```
    這樣 **Deployment 的 metadata 會更新，但不會觸發新的 Pod 創建或刪除**。

    📌 適用情境**
    - 增加 **Labels** 來更好地管理 Deployment：
        ```yaml
        metadata:
        labels:
            environment: production
        ```
    - 增加或修改 **Annotations**，例如增加監控標籤：
        ```yaml
        metadata:
        annotations:
            monitoring: "enabled"
        ```
    - 調整 **Rolling Update** 策略：
        ```yaml
        strategy:
        type: RollingUpdate
        rollingUpdate:
            maxSurge: "50%"
            maxUnavailable: "25%"
        ```
        → **Pod 仍然保持不變，只有 Deployment 設定更新。**

    ---

    **2️⃣ 修正 `Deployment` 的 YAML 配置，但不影響現有 Pod**
    如果你發現 `Deployment` YAML **有錯誤**，但 **現有的 Pod 運作正常**，你可能只想修正 YAML，而不想讓 Kubernetes 重新建立 Pod。

    **📌 例如**
    - 你不小心 **漏掉 `kubectl.kubernetes.io/last-applied-configuration` 註解**，導致 `kubectl apply` 失敗：
        ```yaml
        metadata:
        annotations:
            kubectl.kubernetes.io/last-applied-configuration: '{"apiVersion":"apps/v1", ... }'
        ```
        **解決方法**
        ```sh
        kubectl replace -f redis-deploy.yaml
        ```
        **Pod 不會受到影響，但 `Deployment` 會更新。**

    ---

    **3️⃣ 避免影響線上業務**
    如果你有一個正在運行的 `Deployment`，但希望暫時修改 Deployment 設定，**又不想讓 Pod 重新啟動**（例如高流量時段），可以用 `kubectl replace` 來 **更新 `Deployment` 設定，然後等到低流量時再進行 `kubectl rollout restart`**。

    **📌 適用情境**
    - 線上環境的 **高峰時段**，不希望影響業務。
    - **只想先修改 Deployment，稍後再重啟 Pod**。

    **流程**
    1. **使用 `kubectl replace` 更新 Deployment，但不影響現有 Pod**
        ```sh
        kubectl replace -f redis-deploy.yaml
        ```
    2. **等到低流量時，手動滾動更新**
        ```sh
        kubectl rollout restart deployment redis
        ```

    這樣可以 **分開「更新 Deployment」和「重啟 Pod」的時間點，避免影響業務**。

    ---

    **❌ 什麼時候** `kubectl replace` **不適用？**
    - **當你需要立刻更新 Pod（例如改變環境變數或 Image）**
        - `kubectl replace` **不會觸發 Rolling Update**，Pod 仍然保持舊的狀態。
        - 如果你改變了 `image` 或 `env` 但只使用 `kubectl replace`，Pod **不會自動重新啟動**。
        - **正確做法**：
        ```sh
        kubectl apply -f redis-deploy.yaml
        ```
        - 或者手動重啟：
        ```sh
        kubectl rollout restart deployment redis
        ```

    - **當你想觸發滾動更新**
        - `kubectl replace` **不會觸發 Rolling Update**，但 `kubectl apply` 會根據變更的內容來決定是否需要滾動更新。
        - **如果你希望 Pod 立即更新，應該用 `kubectl apply` 或 `kubectl set image`。**

    ---

    **📌 總結**
    | 指令 | 更新 Deployment | 影響 Pod |
    |------|--------------|---------|
    | `kubectl apply` | 是 | 會觸發 Rolling Update |
    | `kubectl replace` | 是 | **不影響** Pod |
    | `kubectl edit deployment` | 是 | 可能影響 Pod（取決於變更內容） |
    | `kubectl rollout restart` | 否 | 重新啟動所有 Pod |

    **適合使用 `kubectl replace` 的情境**
    ✅ 更新 **標籤 (Labels)**、**註釋 (Annotations)** 或 **策略 (Strategy)**，但不希望影響 Pod。  
    ✅ 修正 **Deployment 的 YAML 格式錯誤**，但現有 Pod 是健康的。  
    ✅ **高流量時段**，先更新 `Deployment`，稍後再手動重啟 Pod。

    你可以試試 **`kubectl replace -f redis-deploy.yaml`**，然後執行 `kubectl get pods` 看看 Pod 是否仍然保持不變！ 🚀