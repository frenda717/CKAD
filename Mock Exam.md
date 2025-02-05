### 01. Application Deployment (20%)

1. 

### 02. Application Environment, Configuration and Security (25%)  

1. Pod + ConfigMap + Volume
    Create a pod called time-check in the dvl1987 namespace. 
    This pod should run a container called time-check that uses the busybox image.

    Create a config map called time-config with the data TIME_FREQ=10 in the same namespace.
    The time-check container should run the command: while true; do date; sleep $TIME_FREQ;done and write the result to the location /opt/time/time-check.log.
    The path /opt/time on the pod should mount a volume that lasts the lifetime of this pod.

    我的配置:

    controlplane ~ ➜  k get pod time-check -n dvl1987 -o yaml > time-check-pod.yaml

    controlplane ~ ➜  k delete pod -n dvl1987 time-check 
    pod "time-check" deleted


    controlplane ~ ➜  vim time-check-pod.yaml 
    
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: dapi-test-pod
      namespace: dvl1987
    spec:
      containers:
        - name: test-container
          image: registry.k8s.io/busybox
          command: [ "/bin/sh", "-c", "while true; do date; sleep $TIME_FREQ;done >  /opt/time/time-check.log" ]
          env:
            # Define the environment variable
            - name: TIME_FREQ
              valueFrom:
                configMapKeyRef:
                  # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
                  name: time-config
                  # Specify the key associated with the value
                  key: TIME_FREQ
        volumeMounts:
          - name: log-volume
          mountPath: /opt/time

    volumes:
    - name: log-volume
      ~~configMap:~~
        ~~name: time-config~~

    ---
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: time-config
      namespace: dvl1987
    data:
      TIME_FREQ: "10"
    
    ---

    controlplane ~ ➜  k apply -f time-check-pod.yaml 
    pod/time-check created

    controlplane ~ ➜  k get pod
    NAME           READY   STATUS    RESTARTS   AGE
    logger         1/1     Running   0          28m
    secure-pod     1/1     Running   0          25m
    webapp-color   1/1     Running   0          45m



      Solution:
        ---
        apiVersion: v1
        kind: Pod
        metadata:
        labels:
            run: time-check
        name: time-check
        namespace: dvl1987
        spec:
        **volumes**:
        - name: log-volume
            **emptyDir**: {} (見IMPORTANT -1)
        containers:
        - image: busybox
            name: time-check
            **env**:
            - name: TIME_FREQ
            **valueFrom**:
                    **configMapKeyRef**:
                    name: time-config
                    key: TIME_FREQ
            **volumeMounts**:
            - mountPath: /opt/time
            name: log-volume
            command:
            - "**/bin/sh**" (見IMPORTANT -2)
            - "**-c**"
            - "while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log"
            (OR:
            command: ["**/bin/sh**","**-c**","cat /etc/config/keys" ])

      **IMPORTANT -1**:
      Solution給的:
        volumes:
        - name: log-volume
            **emptyDir**: {} # {} 表示該對象沒有任何額外的配置（使用預設值）

      controlplane ~ ➜  k explain pod.spec.volumes.emptyDir --recursive
      KIND:       Pod
      VERSION:    v1

      FIELD: emptyDir <EmptyDirVolumeSource>


      DESCRIPTION:
          emptyDir represents a temporary directory that shares a pod's lifetime. More
          info: https://kubernetes.io/docs/concepts/storage/volumes#emptydir
          Represents an empty directory for a pod. Empty directory volumes support
          ownership management and SELinux relabeling.
          
      FIELDS:
        medium        <string>
        sizeLimit     <Quantity>

      這等同於：

      volumes:
        - name: config
          emptyDir:
            medium: ""    # 預設為空字符串，表示使用磁碟存儲
            sizeLimit: "" # 預設為空，表示不限制大小


      我設置的:
        volumes:
        - name: log-volume
        ~~configMap:~~
        ~~name: time-config~~
      
      為何 emptyDir: {} 會被用於 volumes？
      在 Kubernetes 中，volumes 的設置決定了 Pod 內部如何存儲和共享數據。你的疑問是 為何 emptyDir: {} 被用於 volumes，而不是 configMap？

      a. emptyDir 的作用
      **emptyDir 是一種臨時性（Ephemeral）的 Volume**。
      這種 Volume 會在 Pod 存在期間保持存儲數據，但 一旦 Pod 被刪除，數據就會丟失。 (符合題目: mount a volume that lasts the lifetime of this pod)
      emptyDir 在 Pod 內部的所有容器之間是共享的，也就是說，如果有多個容器，它們可以同時讀寫 emptyDir 掛載的目錄。
      適用情境
        日誌文件存儲（log files）
        臨時緩存數據（temporary cache）
        容器之間共享數據（data sharing between containers）
      
      b. 為何不使用 configMap 作為 volumes？
      你在 YAML 文件中寫了：
      volumes:
        - name: log-volume
          configMap:
            name: time-config
      這是不正確的，因為：

      **configMap 主要用來提供靜態的 Key-Value 配置，而 不能用來存儲變動的日誌文件**。
      configMap 只能存放靜態文件，而 while true; do date; sleep $TIME_FREQ; done > /opt/time/time-check.log 會不斷產生新數據，這並不符合 configMap 的用途。
      **configMap 是只讀的**，它的內容不能在容器內部被動態修改。
      什麼時候用 configMap 作為 volumes？
        只用來存放 靜態配置文件（例如 config.yaml）。
        不會變更的數據（例如 SSH 金鑰、TLS 憑證）。

    emptyDir 在此處的作用
    在 solution 中，這段：

    volumes:
      - name: log-volume
        emptyDir: {}
    
    表示：
    Kubernetes 會給這個 Pod 分配一塊臨時存儲空間，用來存放 /opt/time/time-check.log。
    這塊存儲在 Pod 存在期間都可以使用，但 一旦 Pod 重新啟動或刪除，數據就會消失。
    它允許容器寫入 /opt/time/time-check.log，而不會遇到只讀問題。
    所以： 
    ✅ 用 emptyDir: {}，因為它適用於 Pod 內的臨時數據存儲（如日誌）。
    ❌ 不能用 configMap，因為 configMap 無法動態存儲變更的數據。


    Volume 類型的快速決策:
    | 需求                               | 適合的 Volume 類型              |
    |:-----------------------------------|:-------------------------------|
    | 存儲可修改的臨時數據                 | emptyDir |
    | 持久化數據，即使 Pod 重新啟動	       |  PersistentVolumeClaim (PVC)   |
    | 只讀的靜態配置                      | configMap 或 secret |
    | 直接存取宿主機的檔案                 | hostPath  |
      

      **IMPORTANT -2**:
        /bin/sh 是什麼？
        容器的基礎映像是 busybox，它是一個輕量級的 Linux 系統，並沒有 bash，但有 /bin/sh（類似 ash 或 dash）。
        在 busybox 或大多數 Linux 環境中，當你要執行一個較為複雜的 Shell 指令（如 while 迴圈），你需要一個 Shell 解釋器來執行它。
        直接執行 while true; do date; sleep $TIME_FREQ; done > /opt/time/time-check.log 可能無法正常解析，因為 command 需要執行的是一個具體的二進制程式，而 while true 這類指令是 Shell 指令。
        
        -c 的作用
        -c 參數的作用是讓 Shell 執行後面接的字串作為 Shell 命令。
        command 參數列表的完整格式：
        command:
        - "/bin/sh"
        - "-c"
        - "while true; do date; sleep $TIME_FREQ; done > /opt/time/time-check.log"
        這相當於手動執行：
        /bin/sh -c "while true; do date; sleep $TIME_FREQ; done > /opt/time/time-check.log"
        
        如果不加 -c，Shell 會將 "while true; do date; sleep $TIME_FREQ; done > /opt/time/time-check.log" 當作一個二進制可執行文件來執行，但這顯然不是可執行的程序，因此會報錯。  

      **IMPORTANT -3**:
      controlplane ~ ➜  kubectl exec -it time-check -n dvl1987 -- env | grep TIME_FREQ
      error: Internal error occurred: unable to upgrade connection: container not found ("time-check")


      倘若pod因為container內部配置有問題導致無限重啟，可建立debug pod 來排查原因:
        
          kubectl run debug-shell --rm -i -t --image=busybox --restart=Never --namespace=dvl1987 -- sh


      controlplane ~ ✖ kubectl run debug-shell --rm -i -t --image=busybox --restart=Never --namespace=dvl1987 -- sh
      If you don't see a command prompt, try pressing enter.
      / # env | grep TIME_FREQ (檢查pod是否正確傳遞環境變量)
      / # 
      / # 

      回到配置文件:
      spec:
        containers:
        - image: busybox
          imagePullPolicy: Always
          name: time-check
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          command:
            - "/bin/sh"
            - "-c"
            - "while true; do date; sleep ${TIME_FREQ:-5};done > /opt/time/time-check.log"
          env:
          - name: TIME_FREQ
            valueFrom:
              configMapKeyRef:
                name: time-config
                key: TIME_FREQ
          volumeMounts:
            - name: config
              mountPath: /opt/time
              **readOnly: true** # **表示開起唯獨模式，導致因為無法寫入/opt/time而報錯!!!**
        volumes:
        - name: config
          emptyDir: {}

      controlplane ~ ➜  vim time-check-pod.yaml 
      刪除readOnly設置。

      controlplane ~ ➜  k delete pod time-check -n dvl1987 --force
      Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
      pod "time-check" force deleted

      controlplane ~ ➜  kc time-check-pod.yaml 
      pod/time-check created

      controlplane ~ ➜  k get pod -n dvl1987
      NAME         READY   STATUS    RESTARTS   AGE
      time-check   1/1     Running   0          8s



   
2. Persistent Volume + PVC + Pod Mounting
  
    Create a Persistent Volume called log-volume. It should make use of a storage class name manual. It should use RWX as the access mode and have a size of 1Gi. The volume should use the hostPath /opt/volume/nginx

    Next, create a PVC called log-claim requesting a minimum of 200Mi of storage. This PVC should bind to log-volume.

    Mount this in a pod called logger at the location /var/www/nginx. This pod should use the image nginx:alpine.

    我的配置:

    controlplane ~ ➜  vim log-volume-pv.yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: log-volume
    spec:
      capacity:
        storage: 1Gi
      accessModes:
        - ReadWriteMany
      volumeMode: Filesystem
      persistentVolumeReclaimPolicy: Retain
      storageClassName: manual

      hostPath:
        path: /opt/volume/nginx


    controlplane ~ ➜  k create -f log-volume-pv.yaml 
    persistentvolume/log-volume created

    controlplane ~ ➜  k get pv
    NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
    log-volume   1Gi        RWX            Retain           Available           manual         <unset>                          2s


    controlplane ~ ✖ vim log-claim-pvc.yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: log-claim
    spec:
      accessModes:
        - ReadWriteMany
      volumeMode: Filesystem
      resources:
        requests:
          storage: 200Mi
      storageClassName: manual


    controlplane ~ ➜  k create -f  log-claim-pvc.yaml
    persistentvolumeclaim/log-claim created

    controlplane ~ ➜  k get pvc
    NAME        STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
    log-claim   Bound    log-volume   1Gi        RWX            manual         <unset>                 2s

    controlplane ~ ➜  k get pod
    NAME           READY   STATUS    RESTARTS   AGE
    webapp-color   1/1     Running   0          13m

    controlplane ~ ➜  vim logger-pod.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: logger
      namespace: default
    spec:
      restartPolicy: Never
      containers:
      - name: logger
        image: nginx:alpine
        volumeMounts:
        - mountPath: /var/www/nginx
          name: volume
          readOnly: true

      volumes:
      - name: volume
        hostPath:
          path: /opt/volume/nginx


    controlplane ~ ➜  k create -f logger-pod.yaml 
    Error from server (BadRequest): error when creating "logger-pod.yaml": Pod in version "v1" cannot be handled as a Pod: strict decoding error: unknown field "spec.containers[0].volumes"

    controlplane ~ ✖ vim logger-pod.yaml

    controlplane ~ ➜  k create -f logger-pod.yaml 
    pod/logger created

    controlplane ~ ➜  k get pod
    NAME           READY   STATUS    RESTARTS   AGE
    logger         1/1     Running   0          3s
    webapp-color   1/1     Running   0          17m




### 03. Application Design and Build (20%)

1. Deployment + Rolling Update + Rollback 
  Create a new deployment called nginx-deploy, with one single container called nginx, image nginx:1.16 and 4 replicas.
  The deployment should use RollingUpdate strategy with maxSurge=1, and maxUnavailable=2.

    Next upgrade the deployment to version 1.17.

    Finally, once all pods are updated, undo the update and go back to the previous version.

    Step1: upgrade the deployment to version 1.17
      
      我的配置:
      
      controlplane ~ ➜  k create deploy nginx-deploy --image=nginx:1.16 --replicas=4
      deployment.apps/nginx-deploy created

      controlplane ~ ➜  k get deploy
      NAME           READY   UP-TO-DATE   AVAILABLE   AGE
      nginx-deploy   0/4     4            0           5s

      controlplane ~ ➜  k get pod
      NAME                            READY   STATUS    RESTARTS   AGE
      logger                          1/1     Running   0          30m
      nginx-deploy-58df6b7867-jcj7d   1/1     Running   0          8s
      nginx-deploy-58df6b7867-qhrgj   1/1     Running   0          8s
      nginx-deploy-58df6b7867-qsqn8   1/1     Running   0          8s
      nginx-deploy-58df6b7867-xkvsk   1/1     Running   0          8s
      secure-pod                      1/1     Running   0          28m
      webapp-color                    1/1     Running   0          48m

      controlplane ~ ➜  k edit deploy nginx-deploy  (修改maxSurge=1, and maxUnavailable=2)
      deployment.apps/nginx-deploy edited


      controlplane ~ ➜  kubectl set image deployment/nginx-deploy nginx=nginx:1.17
      deployment.apps/nginx-deploy image updated

      controlplane ~ ✖ k rollout history deployment nginx-deploy 
      deployment.apps/nginx-deploy 
      REVISION  CHANGE-CAUSE
      1         <none>
      2         <none>

    
     ---


      Solution:
      controlplane ~ ➜  k set 
      env             (Update environment variables on a pod template)
      image           (Update the image of a pod template)
      resources       (Update resource requests/limits on objects with pod templates)
      selector        (Set the selector on a resource)
      serviceaccount  (Update the service account of a resource)
      subject         (Update the user, group, or service account in a role binding or cluster role binding)

      controlplane ~ ➜  k set image deployments nginx-deploy nginx=nginx:1.16
      deployment.apps/nginx-deploy image updated

      controlplane ~ ➜  k rollout status deployment nginx-deploy 
      deployment "nginx-deploy" successfully rolled out


      controlplane ~ ➜  k rollout history deployment nginx-deploy 
      deployment.apps/nginx-deploy 
      REVISION  CHANGE-CAUSE
      2         <none>
      3         <none>
          (eployment 確實進行了版本變更（從 2 -> 3），但沒有 CHANGE-CAUSE，原因可能是：
          kubectl edit 及 kubectl set image 沒有使用 --record，所以沒有記錄 Change-Cause。
          你可以手動加上 **--record** 來記錄：
          kubectl set image deployment nginx-deploy nginx=nginx:1.17 --record
          這樣 kubectl rollout history 就會顯示變更原因。)


      (為何 kubectl edit deploy 也可以視為 Upgrade？)
      當你使用：
      kubectl edit deploy nginx-deploy
      然後手動修改 image: nginx:1.16 -> nginx:1.17，系統會視為一次 Deployment 更新（Upgrade），因為 Kubernetes 會檢測到 Deployment 規範的變更，然後執行 Rolling Update。

      發生了什麼？
      kubectl edit 會開啟 Deployment 的 YAML，讓你手動編輯。
      當你變更 image: nginx:1.16 -> nginx:1.17 並儲存後，Kubernetes 會自動偵測這次變更，並觸發滾動更新（Rolling Update）。
      kubectl edit 本質上修改了 Deployment 的 .spec.template.spec.containers.image，Kubernetes 會根據 Deployment 的 RollingUpdate 策略（maxSurge=1, maxUnavailable=2）來更新 Pod。
      你可以透過 kubectl rollout status deployment nginx-deploy 來觀察滾動更新的進度。
      所以，即使 kubectl edit 不是專門設計來「執行更新」的指令，它仍然會導致 Deployment 變更，從而觸發升級。

      kubectl edit vs kubectl set image
      這兩種方式本質上都是修改 Deployment 規範，但方式不同：

      |                 指令               |                 修改方式         	 |               影響             |
      |:-----------------------------------|:-----------------------------------|:-------------------------------|
      |kubectl edit deploy nginx-deploy    | 手動編輯 YAML 來修改 image	         |變更後 Kubernetes 會觸發 Rolling Update|
      |kubectl set image deploy/nginx-deploy nginx=nginx:1.17|	直接修改 image 的值	|立即生效，並觸發 Rolling Update|

    Step2: undo the update and go back to the previous version

      我的配置:

      controlplane ~ ➜  k rollout undo deployment nginx-deploy 
      deployment.apps/nginx-deploy rolled back

      controlplane ~ ➜  k rollout history deployment nginx-deploy 
      deployment.apps/nginx-deploy 
      REVISION  CHANGE-CAUSE
      2         <none>
      3         <none>
      
      Solution:

      kubectl rollout undo deployment nginx-deploy (回滾)
      這會讓 Deployment 回到 上一個版本（從 3 回到 2）。

      如果你**想回滾到特定版本**：
      kubectl rollout undo deployment nginx-deploy **--to-revision**=2


      controlplane ~ ➜  k get pod
      NAME                            READY   STATUS    RESTARTS   AGE
      logger                          1/1     Running   0          24m
      nginx-deploy-58df6b7867-25qr9   1/1     Running   0          32s
      nginx-deploy-58df6b7867-f756s   1/1     Running   0          29s
      nginx-deploy-58df6b7867-nwmql   1/1     Running   0          32s
      nginx-deploy-58df6b7867-tz8ck   1/1     Running   0          32s
      secure-pod                      1/1     Running   0          24m
      webapp-color                    1/1     Running   0          44m

      controlplane ~ ➜  k describe deployments.apps nginx-deploy 
      (檢查version變更回nginx: 1.16省略)

 



2. Deployment + Resource + Volume (State Persistence)

    Create a redis deployment with the following parameters:
    Name of the deployment should be redis using the redis:alpine image. It should have exactly 1 replica.
    The container should request for .2 CPU. It should use the label app=redis.
    It should mount exactly 2 volumes.

    a. An Empty directory volume called data at path /redis-master-data.
    b. A configmap volume called redis-config at path /redis-master.
    c. The container should expose the port 6379.
    The configmap has already been created.

    我的配置:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: redis
    spec:
      selector:
        matchLabels:
          app: redis
      replicas: 1
      template:
        metadata:
          labels:
            app: redis
        spec:
          volumes:
            - name: configmap-volume
              configMap:
                name: redis-config
            (**少一個/redis-master-data對應的volumes配置: emptyDir:{}**)
          containers:
          - name: redis
            image: redis:alpine
            ports:
            - containerPort: 6379
            (**未設置container的resource request屬性**)
            volumeMounts:
            - mountPath: /redis-master
              name: configmap-volume
            (**少一個/redis-master-data**)

        (volume mounts跟volumes需要成對，有多少個volumeMounts，就有多少個volumes)
        不一定 "必須" 完全成對，但通常應該一一對應，否則 volumeMounts 可能會因為找不到對應的 volumes 而導致 Pod 創建失敗。
        volumeMounts 和 volumes 的關係
      
      在 Kubernetes 中：

      volumes 是 Pod 內定義的儲存資源。
      volumeMounts 是容器內部掛載 volumes 的路徑。
      基本規則： ✅ 每個 volumeMounts.name 必須對應一個 volumes.name，否則會報錯。
      ✅ 可以有 volumes 但沒有 volumeMounts（例如另一個容器使用該 volume）。
      ❌ 不能有 volumeMounts 但沒有對應的 volumes，否則 Pod 會報錯。    

      volumes 比 volumeMounts 多，是否可以？
      可以。例如，如果一個 Pod 有 多個容器，則 volumes 可能比 volumeMounts 多，因為不是所有容器都會掛載所有 volumes：

      spec:
        volumes:
          - name: shared-volume
            emptyDir: {}
        containers:
          - name: container1
            image: nginx
            volumeMounts:
              - mountPath: /data
                name: shared-volume  # 這個容器掛載 shared-volume
          - name: container2
            image: busybox
            command: ["sleep", "3600"]
            # ❌ 這個容器沒有 volumeMounts，但 `volumes` 仍然存在
      這是合法的，因為 volumes 可以提供給不同的容器選擇性掛載。

      volumeMounts 比 volumes 多，是否可以？
      不可以！
      如果 volumeMounts 指向一個不存在的 volumes，Pod 會無法啟動，並報錯：
      MountVolume.SetUp failed for volume "missing-volume" : volume not found


    Solution:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: redis
      name: redis
    spec:
      selector:
        matchLabels:
          app: redis
      template:
        metadata:
          labels:
            app: redis
        spec:
          volumes:
          - name: data
            emptyDir: {}
          - name: redis-config
            configMap:
              name: redis-config
          containers:
          - image: redis:alpine
            name: redis
            volumeMounts:
            - mountPath: /redis-master-data
              name: data
            - mountPath: /redis-master
              name: redis-config
            ports:
            - containerPort: 6379
            **resources**: 需設置resource request (我的配置中忽略了此設置)
              requests:
                cpu: "0.2"

### 04. Application Observability and Maintenance (15%)



### 05. Services and Networking (20%)
1. Ingress-Network Policy + Service Debugging
  We have deployed a new pod called secure-pod and a service called secure-service. Incoming or Outgoing connections to this pod are not working.
  Troubleshoot why this is happening. 

    Make sure that incoming connection from the pod webapp-color are successful.
    Important: Don't delete any current objects deployed.

    |   方式  |	        設置位置        |      	作用       |	                適用場景                |
    |:--------|:-----------------------|:-----------------|:----------------------------------------|
    |Ingress|	在 目標 Pod (secure-pod)|	限制「誰可以進來」|	當 secure-pod 已經有 NetworkPolicy，想允許 webapp-color 訪問時|
    |Egress|	在 發送流量的 Pod (webapp-color)|	限制「能發去哪」|	當你想防止 webapp-color 訪問其他 Pod 或外部服務時|
    ✔ 在這個問題中，secure-pod 可能已經有一個 Ingress 限制，所以要在 secure-pod 上設置 Ingress Policy，允許 webapp-color 進來，而不是在 webapp-color 上設置 Egress Policy！
    
    我的配置:


    controlplane ~ ➜  k get networkpolicies
    NAME                    POD-SELECTOR     AGE
    default-deny            <none>           38m


    controlplane ~ ✖ cat secure-networkpolicy.yaml 
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: secure-network-policy
      namespace: default
    spec:
      podSelector:
        matchLabels:
          **pod: secure-pod** 未先確認是否真有此label!!!!
      policyTypes:
      - Ingress
      ingress:
      - from:
        - podSelector:
            matchLabels:
              **pod: webapp-color** 未先確認是否真有此label!!!!
        ports:
        - protocol: TCP
          port: 80

 
    controlplane ~ ✖ alias kc="k create -f"

    controlplane ~ ➜  kc secure-networkpolicy.yaml 
    networkpolicy.networking.k8s.io/secure-network-policy created


    Usage: netstat [-ral] [-tuwx] [-enWp]

    Display networking information

            -r      Routing table
            -a      All sockets
            -l      Listening sockets
                    Else: connected sockets
            -t      TCP sockets
            -u      UDP sockets
            -w      Raw sockets
            -x      Unix sockets
                    Else: all socket types
            -e      Other/more information
            -n      Don't resolve names
            -W      Wide display
            -p      Show PID/program name for sockets
      /opt # netstat -w 5 secure-service 80
      Active Internet connections (w/o servers)
      Proto Recv-Q Send-Q Local Address           Foreign Address         State  
          
          
      /opt # netstat -w 5 -h
      netstat: unrecognized option: h
      BusyBox v1.28.4 (2018-07-17 15:21:40 UTC) multi-call binary.

      Usage: netstat [-ral] [-tuwx] [-enWp]

      Display networking information

              -r      Routing table
              -a      All sockets
              -l      Listening sockets
                      Else: connected sockets
              -t      TCP sockets
              -u      UDP sockets
              -w      Raw sockets
              -x      Unix sockets
                      Else: all socket types
              -e      Other/more information
              -n      Don't resolve names
              -W      Wide display
              -p      Show PID/program name for sockets
      
      **IMPORTANT**:
      在配置podSelector之前，先用--show-labels參數檢查設置是否正確!!!!
      kubectl get pods --show-labels
      確保 secure-pod 和 webapp-color 的標籤正確。

      controlplane ~ ➜  k get networkpolicy
      NAME                    POD-SELECTOR     AGE
      default-deny            <none>           4m35s
      secure-network-policy   pod=secure-pod   7s


      controlplane ~ ➜  k exec -it webapp-color -- sh
      /opt # nc -vz -w 5 secure-pod 80 
      ###! nc -vz -w 5 secure-pod 80 會失敗，因為 Pod 名稱不會自動解析為 Pod 的 IP 地址。
      ###! Kubernetes 內部 DNS 解析主要適用於 Service，而不是單獨的 Pod。
      nc: bad address 'secure-pod'
      /opt # nc -vz -w 5 secure-service 80
      nc: secure-service (172.20.244.248:80): Operation timed out  (由於第一個pod selector設置錯誤，導致網路不通)


      /opt # nc -vz -w 5 secure-service 80 (修改pod selector配置後，測試成功)
      secure-service (172.20.244.248:80) open

      
      Solution:

      Step0: 檢查label!!! (小心此步驟容易被忽略!!!!)

      controlplane ~ ➜  k get pod --show-labels 
      NAME           READY   STATUS    RESTARTS   AGE    LABELS
      secure-pod     1/1     Running   0          103s   **run=secure-pod**
      webapp-color   1/1     Running   0          2m4s   **name=webapp-color**

      Step1: 檢查 NetworkPolicy
      檢查目前是否有影響 secure-pod 的 NetworkPolicy，執行：
      
      k get networkpolicy (-n default)

      Step2: 應用修正的 NetworkPolicy
      建立以下 NetworkPolicy，確保 webapp-color Pod 可以訪問 secure-pod：
      vim networkpolicy.yaml
      controlplane ~ ➜  cat secure-networkpolicy.yaml 
      apiVersion: networking.k8s.io/v1
      kind: NetworkPolicy
      metadata:
        name: secure-network-policy
        namespace: default
      spec:
        **podSelector**: # 選擇 被保護的 Pod(應該匹配 secure-pod)
          matchLabels:
            **run: secure-pod** # 原本設置為pod: secure-pod 結果webapp-color 無法訪問到secure-pod
        policyTypes:
        - Ingress
        ingress:
        - from:
          - **podSelector**: # 選擇 允許訪問的 Pod（應該匹配 webapp-color）
              matchLabels:
                **name: webapp-color**
          ports:
          - protocol: TCP
            port: 80


      k create -f networkpolicy.yaml

      controlplane ~ ➜  k get networkpolicies.networking.k8s.io 
      NAME                    POD-SELECTOR     AGE
      default-deny            <none>           4m31s
      secure-network-policy   run=secure-pod   5s

      Step3: 驗證網絡連通性
      Then check the connectivity from the webapp-color pod to the secure-pod:-
      檢查 webapp-color 是否能夠連接到 secure-service:
      k exec -it webapp-color --sh
      root@controlplane:~$ kubectl exec -it webapp-color -- sh
      
      然後在 Pod 內部執行：**nc -v -z -w 5** <secure-service> <80>
      nc: Netcat 命令，用來測試網路連接
      -v: 顯示詳細模式（verbose mode），會輸出連接過程的詳細信息。
      -z: 只檢查連接是否成功，不發送數據，適用於端口掃描。
      -w: 設定超時時間為 5 秒，如果超過 5 秒無回應則放棄連接
      <目標服務名稱（Kubernetes 內部服務）>
      <端口>

      /opt # nc -v -z -w 5 secure-service 80

      如果成功，將顯示:
      secure-service [10.100.1.23] 80 (http) open

      如果失敗，可能會顯示：
      nc: connect to secure-service port 80 (tcp) failed: Connection refused


      **IMPORTANT**:
      可以使用 telnet 指令嗎？
      是的，telnet 也可以用來測試連接，例如：
      telnet secure-service 80

      但與 nc（Netcat）相比，telnet 有幾個問題：
      需要手動輸入 Ctrl+C 才能退出，無法像 nc -z 那樣快速測試端口是否開放。
      某些 Linux 容器鏡像（如 alpine）不一定內建 telnet，但通常會有 nc。
      telnet 主要用於交互式測試，如果只需要檢查連通性，nc 會更方便。
      在 CKAD 考試或 Kubernetes 環境中，一般 推薦使用 nc，因為它簡單高效

      為何使用 netcat (nc)？
      **nc（Netcat）通常內建於許多容器鏡像**，例如：

      busybox（輕量容器，CKAD 考試常見）
      alpine（輕量級 Linux 版本）
      debian、ubuntu（較完整的 Linux 環境）
      很多官方的 Kubernetes 容器映像不會內建 telnet，但大部分會有 nc，因此 nc 是較通用的選擇。

      如果 Pod 內什麼指令都沒有，例如：
        kubectl exec -it mypod -- sh
        sh: nc: not found

      **法 1：進入其他有 nc 的 Pod 測試**
      可以使用一個已有 nc 指令的 Pod，例如：
      kubectl run debug-pod --rm -it --image=busybox -- sh
      然後在這個 Pod 內執行：
      nc -v -z -w 5 secure-service 80
      這樣可以避免安裝新軟體，適用於 CKAD 考試場景。

      **如果 Pod 內沒有 nc，馬上創建一個 busybox 或 alpine Pod 來測試**
      kubectl run debug-pod --rm -it --image=busybox -- sh
      使用 nc -z -v 來快速測試端口開放情況
      nc -z -v secure-service 80


      法 2：手動安裝 nc
      如果 Pod 允許安裝（需 root 權限），可以試著安裝：

      Alpine Linux（常見於 CKAD 考試環境）
      
          apk add --no-cache netcat-openbsd

      Debian / Ubuntu
        
          apt update && apt install -y netcat

      但 CKAD 考試時間有限，通常 方式 1 會更快！

      不要浪費時間安裝 telnet 或 nc，直接用 busybox 測試會更快。

