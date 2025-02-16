### 01. Application Deployment (20%)

  1. CronJob、Job Limit、BackoffLimit
  
      Create a cronjob called dice that runs every one minute. Use the Pod template located at /root/throw-a-dice. The image throw-dice randomly returns a value between 1 and 6. The result of 6 is considered success and all others are failure.
      The job should be **non-parallel** and **complete the task once**. Use a backoffLimit of 25.
      
      If the task is not completed within 20 seconds the job should fail and pods should be terminated.
      You don't have to wait for the job completion. As long as the cronjob has been created as per the requirements.

      我的配置:

      controlplane ~ ➜  ls
      dice-conjob.yaml  dice-job.yaml  throw-a-dice

      controlplane ~ ➜  cat throw-a-dice/throw-a-dice.yaml (查看當前pod 設置)
        
        apiVersion: v1
        kind: Pod
        metadata:
          name: throw-dice-pod
        spec:
          containers: # 根據doc: "For a non-parallel Job, you can leave both .spec.completions and .spec.parallelism unset. When both are unset, both are defaulted to 1" ，故此處不設置parallelism 跟completions，默認為1。
          -  image: kodekloud/throw-dice (檢查Imgage)
            name: throw-dice
          restartPolicy: Never

      controlplane ~ ➜  cat dice-job.yaml
        
        apiVersion: batch/v1
        kind: Job
        metadata:
          name: dice-job
        spec:
          template:
            spec:
              restartPolicy: Never # required for the feature
              containers:
              - name: throw-dice
                image: kodekloud/throw-dice
          backoffLimit: 25
          activeDeadlineSeconds: 20


      controlplane ~ ➜  k get job
      NAME       STATUS     COMPLETIONS   DURATION   AGE
      dice-job   Complete   1/1           4s         7m38s

      但是!!!目前建立的 dice-job.yaml 是一個 Job，但題目要求 CronJob，所以我們需要修改 apiVersion 和 kind：

      Job 只會執行一次
      CronJob 則可以按照排程執行（如題目要求的「每分鐘一次」）


      根據job的配置，將template spec裡面的內容貼到cronjob配置文件中:

        apiVersion: batch/v1
        kind: CronJob
        metadata:
          name: dice
        spec:
          schedule: "* * * * *"
          jobTemplate:
            spec:
              template:
                spec:
                  restartPolicy: Never # required for the feature
                  containers:
                  - name: throw-dice
                    image: kodekloud/throw-dice
              backoffLimit: 25
              activeDeadlineSeconds: 20 (使用k explain cronjob.spec.jobTemplate.spec.template --recursive查看是否有此配置項)

      controlplane ~ ➜  vim dice-conjob.yaml 
      
      OR
      
      controlplane ~ ➜  kubectl create cronjob dice --image=kodekloud/throw-dice --schedule="*/1 * * * *"
      cronjob.batch/dice created

      controlplane ~ ➜  k edit cronjobs.batch 
        spec:
          concurrencyPolicy: Allow
          failedJobsHistoryLimit: 1
          jobTemplate:
            metadata:
              creationTimestamp: null
              name: dice
            spec:
              activeDeadlineSeconds: 20
              backoffLimit: 25
              template:
                metadata:
                  creationTimestamp: null
                spec:
                  containers:
                  - image: kodekloud/throw-dice
                    imagePullPolicy: Always
                    name: dice
                    resources: {}
                    terminationMessagePath: /dev/termination-log
                    terminationMessagePolicy: File
                  dnsPolicy: ClusterFirst
                  restartPolicy: OnFailure
                  schedulerName: default-scheduler
                  securityContext: {}
                  terminationGracePeriodSeconds: 30
          schedule: '*/1 * * * *'

      cronjob.batch/dice edited

      controlplane ~ ➜  k get job --watch
      NAME            STATUS     COMPLETIONS   DURATION   AGE
      dice-28981055   Complete   1/1           5s         97s
      dice-28981056   Complete   1/1           18s        37s
      dice-28981057   Running    0/1                      0s
      dice-28981057   Running    0/1           0s         0s
      dice-28981057   Running    0/1           3s         3s
      dice-28981057   Complete   1/1           3s         3s

      controlplane ~ ➜  k create -f  dice-conjob.yaml 
      cronjob.batch/dice created

      controlplane ~ ➜  k get cronjobs 
      NAME   SCHEDULE    TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
      dice   * * * * *   <none>     False     0        <none>          4s


      controlplane ~ ✖ kubectl get cronjobs --watch
      NAME   SCHEDULE    TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
      dice   * * * * *   <none>     False     0        48s             90s
      dice   * * * * *   <none>     False     1        0s              102s
      dice   * * * * *   <none>     False     0        4s              106s

      開另一個分頁，觀察pod

      controlplane ~ ➜  **kubectl get pods --watch**
      NAME                  READY   STATUS      RESTARTS   AGE
      dev-pod-dind-878516   3/3     Running     0          25m
      dice-28980667-9k4l2   0/1     Error       0          67s
      dice-28980667-jj27l   0/1     Completed   0          56s
      dice-28980668-tb9gb   0/1     Completed   0          7s
      dice-job-s88jn        0/1     Completed   0          12m
      pod-xyz1123           1/1     Running     0          25m
      webapp-color          1/1     Running     0          25m

      CronJob 會根據 schedule 自動產生 Job，然後 Job 負責執行具體的 Pod。故檢查job是否被建立:

      controlplane ~ ➜  **kubectl get jobs --watch**
      NAME            STATUS     COMPLETIONS   DURATION   AGE
      dice-28980667   Complete   1/1           15s        3m42s
      dice-28980668   Complete   1/1           4s         2m42s
      dice-28980669   Complete   1/1           15s        102s
      dice-28980670   Failed     0/1           42s        42s
      dice-job        Complete   1/1           4s         14m


      controlplane ~ ✖ k get jobs
      NAME            STATUS     COMPLETIONS   DURATION   AGE
      dice-28980667   Complete   1/1           15s        2m29s
      dice-28980668   Complete   1/1           4s         89s
      dice-28980669   Complete   1/1           15s        29s
      dice-job        Complete   1/1           4s         13m



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




3. Secret Volume + Node Scheduling
    
    Create a pod called my-busybox in the dev2406 namespace using the busybox image. The container should be called secret and should sleep for 3600 seconds.
    The container should mount a read-only secret volume called secret-volume at the path /etc/secret-volume. The secret being mounted has already been created for you and is called dotfile-secret.
    Make sure that the pod is scheduled on controlplane and no other node in the cluster.

    我的配置:
    
    k run my-busybox -n dev2406 --iamge=busybox -o yaml > my-busybox-pod.yaml

    vim my-busybox-pod.yaml
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: **beta.kubernetes.io/arch** # 此處配置錯誤!!! 
                operator: **In** # 錯誤
                values: 
                - **amd64** # 錯誤
        containers:
          - image: busybox
          imagePullPolicy: Always
          name: secret
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          command: ["/bin/sh","-c","sleep 3600"]
          volumeMounts:
          - mountPath: /etc/secret-volume
            name: secret-vol
            readOnly: true
        volumes:
          - name: secret-vol
            secret:
              secretName: dotfile-secret
        tolerations:
          - key: **"beta.kubernetes.io/arch"** # 配置錯誤!
            operator: "Exists"
            effect: "NoSchedule"


    controlplane ~ ➜  k describe node controlplane 
    Name:               controlplane
    Roles:              control-plane
    **Labels**:         beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/os=linux
                        kubernetes.io/arch=amd64
                        kubernetes.io/hostname=controlplane
                        kubernetes.io/os=linux
                        node-role.kubernetes.io/control-plane=
                        node.kubernetes.io/exclude-from-external-load-balancers=
    
    controlplane ~ ➜  vim my-buxybox.yaml 

    controlplane ~ ➜  k delete pod my-busybox -n dev2406 --force
    Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
    pod "my-busybox" force deleted

    controlplane ~ ➜  k create -f  my-buxybox.yaml 
    pod/my-busybox created

    controlplane ~ ➜  k get pod -n dev2406
    NAME          READY   STATUS    RESTARTS   AGE
    my-busybox    1/1     Running   0          3s
    nginx2406     1/1     Running   0          35m
    pod-var2016   1/1     Running   0          35m

    controlplane ~ ✖ k describe pod -n dev2406 my-busybox |grep -i "toleration"
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s

    controlplane ~ ➜  k describe nodes controlplane |grep -i "taints"
    Taints:             <none>

    controlplane ~ ➜  k taint node controlplane beta.kubernetes.io/arch=amd64:NoSchedule

    controlplane ~ ✖ k describe node controlplane |grep -i "taint"
    Taints:             beta.kubernetes.io/arch=amd64:NoSchedule

    
    驗證my-busybox是否運行在controlplane節點:
    controlplane ~ ➜  k get pod my-busybox -n dev2406 -o wide
    NAME         READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
    my-busybox   1/1     Running   0          70s   172.17.1.15   **node01**   <none>           <none>
    (失敗)


    修正後的配置:

    (重新檢查label)
    controlplane ~ ➜  kubectl get nodes --show-labels
    NAME           STATUS   ROLES           AGE   VERSION   LABELS
    controlplane   Ready    control-plane   32m   v1.31.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,**kubernetes.io/hostname=controlplane**,kubernetes.io/os=linux,**node-role.kubernetes.io/control-plane=**,node.kubernetes.io/exclude-from-external-load-balancers=
    node01         Ready    <none>          31m   v1.31.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux


    controlplane ~ ➜  vim my-buxybox.yaml 

    controlplane ~ ➜  k create -f  my-buxybox.yaml 
    pod/my-busybox created

    controlplane ~ ➜  k get pod my-busybox -n dev2406 -o wide
    NAME         READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
    my-busybox   0/1     Pending   0          12s   <none>   <none>   <none>           <none>


    controlplane ~ ➜  cat my-buxybox.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-busybox
      namespace: dev2406
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: **"kubernetes.io/hostname"**
                operator: In
                values:
                - **controlplane**
      containers:
      - image: busybox
        imagePullPolicy: Always
        name: secret
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        command: ["/bin/sh","-c","sleep 3600"]
        volumeMounts:
        - mountPath: /etc/secret-volume
          name: secret-vol
          readOnly: true
      volumes:
        - name: secret-vol
          secret:
            secretName: dotfile-secret
      tolerations:
        - key: **"kubernetes.io/hostname"**
          operator: Exists
          effect: NoSchedule

    controlplane ~ ✖ kubectl taint node controlplane beta.kubernetes.io/arch=amd64:NoSchedule-
    node/controlplane untainted 刪除taint

    controlplane ~ ✖ kubectl taint node controlplane kubernetes.io/hostname=controlplane:NoSchedule
    node/controlplane tainted

    controlplane ~ ➜  k describe node controlplane |grep -i "taint"
    Taints:             kubernetes.io/hostname=controlplane:NoSchedule


    重新驗證:
    controlplane ~ ➜  k get pod my-busybox -n dev2406 -o wide
    NAME         READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
    my-busybox   1/1     Running   0          6m46s   172.17.0.6   controlplane   <none>           <none>


    卻發現仍有大量的pod 運行在controlplane上!!!!
    controlplane ~ ➜  kubectl get pods -A -o wide | grep controlplane
    default         nginx                                       0/1     ContainerCreating   0          34m   <none>            controlplane   <none>           <none>
    dev2406         my-busybox                                  1/1     Running             0          11m   172.17.0.6        controlplane   <none>           <none>
    ingress-nginx   ingress-nginx-admission-patch-l7nfk         0/1     Completed           1          38m   172.17.0.5        controlplane   <none>           <none>
    kube-system     calico-kube-controllers-5d7d9cdfd8-sbfcz    1/1     Running             0          43m   172.17.0.2        controlplane   <none>           <none>
    kube-system     canal-5nkrt                                 2/2     Running             0          43m   192.168.231.151   controlplane   <none>           <none>
    kube-system     coredns-77d6fd4654-fjvc7                    1/1     Running             0          43m   172.17.0.3        controlplane   <none>           <none>
    kube-system     coredns-77d6fd4654-sj7w5                    1/1     Running             0          43m   172.17.0.4        controlplane   <none>           <none>
    kube-system     etcd-controlplane                           1/1     Running             0          43m   192.168.231.151   controlplane   <none>           <none>
    kube-system     kube-apiserver-controlplane                 1/1     Running             0          43m   192.168.231.151   controlplane   <none>           <none>
    kube-system     kube-controller-manager-controlplane        1/1     Running             0          43m   192.168.231.151   controlplane   <none>           <none>
    kube-system     kube-proxy-dk842                            1/1     Running             0          43m   192.168.231.151   controlplane   <none>           <none>
    kube-system     kube-scheduler-controlplane                 1/1     Running             0          43m   192.168.231.151   controlplane   <none>           <none>

    ❌ 你的 Toleration 設置為什麼無法讓其他 Pod 轉移？
    🔹 Toleration 的作用
    Toleration 只會影響 Pod 的調度過程，它不會影響已經運行的 Pod，也不會強制 Pod 轉移。

    🔹 Taint 的作用
    如果你想讓 controlplane 只允許 my-busybox 運行，而其他 Pod 轉移到其他節點，你需要：

    設定一個更嚴格的 Taint，讓 controlplane 只允許 my-busybox 調度。
    讓其他 Pod 不能容忍這個 Taint，這樣 Kubernetes 會「驅逐」它們到其他節點。    


    controlplane ~ ➜  kubectl taint nodes controlplane kubernetes.io/hostname:NoSchedule-
    node/controlplane untainted

    controlplane ~ ➜  kubectl taint nodes controlplane only-my-busybox=true:NoSchedule
    node/controlplane tainted

    controlplane ~ ➜  vim my-buxybox.yaml 

    controlplane ~ ➜  k delete pod my-busybox --force
    Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
    Error from server (NotFound): pods "my-busybox" not found

    controlplane ~ ✖ k delete pod my-busybox -n dev2406 --force
    Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
    pod "my-busybox" force deleted

    controlplane ~ ➜  k create -f  my-buxybox.yaml 
    pod/my-busybox created

    controlplane ~ ➜  k get pod  -n dev2406 -o wide
    NAME          READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
    my-busybox    1/1     Running   0          5s    172.17.0.7   controlplane   <none>           <none>
    pod-var2016   1/1     Running   0          44m   172.17.1.9   node01         <none>           <none>

    controlplane ~ ➜  kubectl get pods -A -o wide | grep controlplane
    default         nginx                                       0/1     ContainerCreating   0          39m   <none>            controlplane   <none>           <none>
    dev2406         my-busybox                                  1/1     Running             0          14s   172.17.0.7        controlplane   <none>           <none>
    ingress-nginx   ingress-nginx-admission-patch-l7nfk         0/1     Completed           1          44m   172.17.0.5        controlplane   <none>           <none>
    kube-system     calico-kube-controllers-5d7d9cdfd8-sbfcz    1/1     Running             0          48m   172.17.0.2        controlplane   <none>           <none>
    kube-system     canal-5nkrt                                 2/2     Running             0          48m   192.168.231.151   controlplane   <none>           <none>
    kube-system     coredns-77d6fd4654-fjvc7                    1/1     Running             0          48m   172.17.0.3        controlplane   <none>           <none>
    kube-system     coredns-77d6fd4654-sj7w5                    1/1     Running             0          48m   172.17.0.4        controlplane   <none>           <none>
    kube-system     etcd-controlplane                           1/1     Running             0          48m   192.168.231.151   controlplane   <none>           <none>
    kube-system     kube-apiserver-controlplane                 1/1     Running             0          48m   192.168.231.151   controlplane   <none>           <none>
    kube-system     kube-controller-manager-controlplane        1/1     Running             0          48m   192.168.231.151   controlplane   <none>           <none>
    kube-system     kube-proxy-dk842                            1/1     Running             0          48m   192.168.231.151   controlplane   <none>           <none>
    kube-system     kube-scheduler-controlplane                 1/1     Running             0          48m   192.168.231.151   controlplane   <none>           <none>

    (原本的pod卻沒有轉移)

    如何強制將現有的 Pod 轉移？
    如果你希望 controlplane 上的 kube-system 內建 Pod 和其他 Pod 轉移到其他節點，你需要 使用 NoExecute Taint，這樣：

    Kubernetes 會立刻驅逐 (evict) 不符合條件的 Pod。
    只有具備 Toleration 的 Pod（如 my-busybox）能夠存活在 controlplane。
    🔹 1️⃣ 移除舊的 NoSchedule Taint
    先刪除原本的 NoSchedule Taint：

    kubectl taint nodes controlplane only-my-busybox=true:NoSchedule-

    🔹 2️⃣ 設置 NoExecute Taint（會強制驅逐不符合的 Pod）
    kubectl taint nodes controlplane only-my-busybox=true:NoExecute

    NoExecute 會強制驅逐所有沒有 Toleration 的 Pod。
    🔹 3️⃣ 確保 my-busybox 允許 NoExecute
    確保 my-busybox.yaml 允許 NoExecute，否則它也會被驅逐：


    tolerations:
      - key: "only-my-busybox"
        operator: "Equal"
        value: "true"
        effect: "NoExecute"


    🔹 4️⃣ 刪除並重新建立 my-busybox
    刪除舊的 Pod，並用新的 Toleration 重新創建：

    kubectl delete pod my-busybox -n dev2406 --force
    kubectl apply -f my-busybox.yaml

    驗證是否成功
    1️⃣ 檢查 controlplane 是否還有其他 Pod

    kubectl get pods -A -o wide | grep controlplane

    2️⃣ 檢查 Taint 是否生效

    kubectl describe node controlplane | grep -i "Taints"



    Solution:

    "Make sure that the pod is scheduled on controlplane and no other node in the cluster": 
    這個 Pod 必須運行在 controlplane 節點上，不能被 Kubernetes 排程到其他 Worker 節點。
    通常可以透過 Node Selector、Node Affinity 或 Taints & Tolerations 來強制 Pod 只能運行在特定節點上
    (解答使用node selector)

      apiVersion: v1
      kind: Pod
      metadata:
        creationTimestamp: null
        labels:
          run: my-busybox
        name: my-busybox
        namespace: dev2406
      spec:
        volumes:
        - name: secret-volume
          secret:
            secretName: dotfile-secret
        nodeSelector:
          kubernetes.io/hostname: controlplane
        containers:
        - command:
          - sleep
          args:
          - "3600"
          image: busybox
          name: secret
          volumeMounts:
          - name: secret-volume
            readOnly: true
            mountPath: "/etc/secret-volume"


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
1. A pod called dev-pod-dind-878516 has been deployed in the default namespace. Inspect the logs for the container called log-x and redirect the warnings to /opt/dind-878516_logs.txt on the controlplane node

    我的配置:

    step1 檢查pod日誌:
    controlplane ~ ✖ k logs dev-pod-dind-878516
    Defaulted container "engine-x" out of: engine-x, agent-x, log-x
    /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
    /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
    10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
    10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
    /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
    /docker-entrypoint.sh: Configuration complete; ready for start up
    2025/02/07 14:55:51 [notice] 1#1: using the "epoll" event method
    2025/02/07 14:55:51 [notice] 1#1: nginx/1.27.4
    2025/02/07 14:55:51 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14) 
    2025/02/07 14:55:51 [notice] 1#1: OS: Linux 5.15.0-1075-gcp
    2025/02/07 14:55:51 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
    2025/02/07 14:55:51 [notice] 1#1: start worker processes
    2025/02/07 14:55:51 [notice] 1#1: start worker process 80
    ...

    
    倘若pod日誌無法查看，則需要進入到容器內:
    k exec -it pod dev-pod-dind-878516 -c log-x   
    k exec -it dev-pod-dind-878516 -c log-x -- /bin/sh(進入log-x容器內)
    cat /path/to/logs |grep -i "warn"

    step2 警告訊息的格式包含關鍵字 "WARNING" 或 "WARN":
    kubectl logs dev-pod-dind-878516 -c log-x -n default | grep -i "warn"

    controlplane ~ ➜  kubectl logs dev-pod-dind-878516 -c log-x -n default | grep -i "warn"
    [2025-02-07 14:56:28,693] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:56:31,697] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
    [2025-02-07 14:56:33,699] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:56:38,707] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:56:39,708] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
    [2025-02-07 14:56:43,713] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:56:47,719] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
    [2025-02-07 14:56:48,720] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:56:53,726] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:56:55,728] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
    [2025-02-07 14:56:58,731] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:57:03,736] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:57:03,736] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
    [2025-02-07 14:57:08,740] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:57:11,744] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
    [2025-02-07 14:57:13,747] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED 


    step3: 將警告訊息存入 /opt/dind-878516_logs.txt
    ~~kubectl logs dev-pod-dind-878516 -c log-x -n default | grep -i "warn" | kubectl exec -it controlplane -- bash -c "cat > /opt/dind-878516_logs.txt"~~

    (以上顯示錯誤，)

    改先把log拷貝到本地文件，再將log文件內容寫入container:
    controlplane ~ ➜  kubectl exec dev-pod-dind-878516 -c log-x -- mkdir /opt;touch /opt/dind-878516_logs.txt  

    controlplane ~ ✖ k cp log-x.txt dev-pod-dind-878516:/opt/dind-878516_logs.txt -c log-x

    controlplane ~ ✖ k exec -it dev-pod-dind-878516 -c log-x -- sh
    / # ls
    bin                 lib                 proc                sys
    dev                 log                 root                tmp
    etc                 media               run                 usr
    event-simulator.py  mnt                 sbin                var
    home                opt                 srv
    / # cat opt/dind-878516_logs.txt 
    [2025-02-07 14:56:28,693] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:56:31,697] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
    [2025-02-07 14:56:33,699] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    ...



    Solution:
    kubectl logs dev-pod-dind-878516 -c log-x | grep WARNING > /opt/dind-878516_logs.txt

    controlplane ~ ➜  kubectl logs dev-pod-dind-878516 -c log-x | grep WARNING > /opt/dind-878516_logs.txt

    controlplane ~ ➜  cat /opt/dind-878516_logs.txt 
    [2025-02-07 14:56:28,693] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:56:31,697] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
    [2025-02-07 14:56:33,699] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:56:38,707] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
    [2025-02-07 14:56:39,708] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.

2. Troubleshooting + Liveness Probe
  
    We have deployed a few pods in this cluster in various namespaces. Inspect them and identify the pod which is not in a Ready state. Troubleshoot and fix the issue.
    Next, add a check to restart the container on the same pod if the command ls /var/www/html/file_check fails. This check should **start after a delay of 10 seconds** and **run every 60 seconds**.
    You may delete and recreate the object. Ignore the warnings from the pro

    Hint: 找出
    **READY 欄位不是 1/1**
    **STATUS 欄位是 CrashLoopBackOff、Pending、Error**、~~Completed~~ 等

    controlplane ~ ✖ k get pod -A
    NAMESPACE       NAME                                        READY   STATUS      RESTARTS   AGE
    default         pod-xyz1123                                 1/1     Running     0          29m
    default         webapp-color                                1/1     Running     0          29m
    dev0403         nginx0403                                   1/1     Running     0          29m
    dev0403         pod-dar85                                   1/1     Running     0          29m
    **dev1401       nginx1401                                   0/1     Running     0          29m**
    dev1401         pod-kab87                                   1/1     Running     0          29m
    dev2406         nginx2406                                   1/1     Running     0          29m
    dev2406         pod-var2016                                 1/1     Running     0          29m
    e-commerce      e-com-1123                                  1/1     Running     0          29m
    **ingress-nginx ingress-nginx-admission-create-9cdqh        0/1     Completed   0          28m**
    **ingress-ngin  ingress-nginx-admission-patch-nq7xg         0/1     Completed   1          28m**
    ingress-nginx   ingress-nginx-controller-7fb4f97b75-gcqj5   1/1     Running     0          28m
    kube-system     calico-kube-controllers-5d7d9cdfd8-n6cns    1/1     Running     0          34m
    kube-system     canal-6srw7                                 2/2     Running     0          34m
    kube-system     canal-dwc2k                                 2/2     Running     0          34m
    kube-system     coredns-77d6fd4654-7lvks                    1/1     Running     0          34m
    kube-system     coredns-77d6fd4654-qhgbf                    1/1     Running     0          34m
    kube-system     etcd-controlplane                           1/1     Running     0          34m
    kube-system     kube-apiserver-controlplane                 1/1     Running     0          34m
    kube-system     kube-controller-manager-controlplane        1/1     Running     0          34m
    kube-system     kube-proxy-dt25z                            1/1     Running     0          34m
    kube-system     kube-proxy-nqdps                            1/1     Running     0          34m
    kube-system     kube-scheduler-controlplane                 1/1     Running     0          34m
    marketing       redis-896d5c767-66547                       1/1     Running     0          29m


    在CKAD考試中不可能一個個去修改每個pod的配置文件，故先找到deployment, 直接修改deployment的配置，如果沒有deployment則修改pod
    

    我的配置: (解題思路錯誤!!!)

    controlplane ~ ➜  k get deploy -n dev1401
    No resources found in dev1401 namespace.

    (此處判斷錯誤，由於nginx-admission-create 跟 nginx-admission-patch 兩個pod的狀態均為completed，不符合"is not in a Ready state")
    ~~controlplane ~ ➜  k get deploy -n ingress-nginx~~
    ~~NAME                       READY   UP-TO-DATE   AVAILABLE   AGE~~
    ~~ingress-nginx-controller   1/1     1            1           33m~~

    以下修改的是ingress-nginx-controller deploy. 實際考試的時候由於pod狀態是completed,故此deployment不需要修改

        spec:
          progressDeadlineSeconds: 600
          replicas: 1
          revisionHistoryLimit: 10
          selector:
            matchLabels:
              app.kubernetes.io/component: controller
              app.kubernetes.io/instance: ingress-nginx
              app.kubernetes.io/name: ingress-nginx
          strategy:
            rollingUpdate:
              maxSurge: 25%
              maxUnavailable: 25%
            type: RollingUpdate
          template:
            metadata:
              creationTimestamp: null
              labels:
                app.kubernetes.io/component: controller
                app.kubernetes.io/instance: ingress-nginx
                app.kubernetes.io/name: ingress-nginx
            spec:
              containers:
              ... (中間忽略)
            livenessProbe:
              failureThreshold: 5
              httpGet:
                path: /healthz
                port: 10254
                scheme: HTTP
              initialDelaySeconds: 10
              periodSeconds: 10
              successThreshold: 1
              timeoutSeconds: 1


        修改為:

            livenessProbe:
              exec: (添加此block)
                command:
                  - ["ls", "/var/www/html/file_check"] 
              failureThreshold: 5
              httpGet:
                path: /healthz
                port: 10254
                scheme: HTTP
              initialDelaySeconds: 10 (Checked)
              periodSeconds: 60 (Modified)
              successThreshold: 1
              timeoutSeconds: 1


              livenessProbe:
              exec:
                command: ["ls", "/var/www/html/file_check"]
              failureThreshold: 5
              httpGet:
                path: /healthz
                port: 10254
                scheme: HTTP
              initialDelaySeconds: 10
              periodSeconds: 60
              successThreshold: 1
              timeoutSeconds: 1

    
          ERROR:
          # deployments.apps "ingress-nginx-controller" was not valid:
          # * spec.template.spec.containers[0].livenessProbe.httpGet: Forbidden: may not specify more than 1 handler type
          #

            exec 跟httpGet 這兩個handler不可同時配置


            再修改為:
            livenessProbe:
              exec:
                command: ["ls", "/var/www/html/file_check"]
              failureThreshold: 5
              initialDelaySeconds: 10
              periodSeconds: 60
              successThreshold: 1
              timeoutSeconds: 1

      ~~controlplane ~ ✖ k edit deploy -n ingress-nginx ingress-nginx-controller~~
      ~~deployment.apps/ingress-nginx-controller edited~~

      ingress-nginx-admission-create 和 ingress-nginx-admission-patch 不需要修改，因為它們是一次性運行的 Pod，Completed 狀態是預期行為。
      nginx1401 需要修正，**因為它是一個持續運行的服務**，但 READY 0/1，表示 Readiness Probe 失敗，導致無法對外提供服務。
      這就是為什麼解答只修改 nginx1401，而不處理 ingress-nginx-admission-* 這兩個 Pod。 


      Solution: 

      From the doc: "If the probe succeeds, the Pod will be marked as ready and will receive traffic from services. If the **readiness probe fails**, **the pod will be marked unready and will not receive traffic from any services**."

      故應優先檢查readinessProbe設置!!!!

      檢查 nginx1401 pod配置:

      k describe pod nginx1401 -n dev1401

      ...
      Events:
      Type     Reason     Age                   From               Message
      ----     ------     ----                  ----               -------
      Normal   Scheduled  4m35s                 default-scheduler  Successfully assigned dev1401/nginx1401 to node01
      Normal   Pulling    4m33s                 kubelet            Pulling image "kodekloud/nginx"
      Normal   Pulled     4m29s                 kubelet            Successfully pulled image "kodekloud/nginx" in 3.969s (3.969s including waiting). Image size: 50986074 bytes.
      Normal   Created    4m29s                 kubelet            Created container nginx
      Normal   Started    4m29s                 kubelet            Started container nginx
      Warning  Unhealthy  97s (x21 over 4m29s)  kubelet            **Readiness probe failed: Get "http://172.17.1.2:8080/": dial tcp 172.17.1.2:8080: connect: connection refused**

      spec:
        containers:
        - image: kodekloud/nginx
          imagePullPolicy: IfNotPresent
          name: nginx
          ports:
          - **containerPort: 9080**
            protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /
              **port: 8080**
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          resources: {}

      **readinessProbe監聽的端口跟containerPort(實際使用)的端口不一致，導致pod 無法正常將status修正為ready**:

      先實際確認端口是否存在:

      (倘若curl, netstat等指令不存在，直接安裝)
      controlplane ~ ✖ kubectl exec -it nginx1401 -n dev1401 -- curl -I http://localhost:8080
      error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "ae2eaad09b6b6740aa669804cd0de1e940e4f8169c06bc8c640661015b902736": OCI runtime exec failed: exec failed: unable to start container process: exec: "curl": executable file not found in $PATH: unknown


      controlplane ~ ✖ kubectl exec -it nginx1401 -n dev1401 -- sh
      # **apt update && apt install -y net-tools iproute2 curl**
      Get:1 http://security.debian.org/debian-security buster/updates InRelease [34.8 kB]
      Get:2 http://deb.debian.org/debian buster InRelease [122 kB]
      Get:3 http://deb.debian.org/debian buster-updates InRelease [56.6 kB]
      Get:4 http://security.debian.org/debian-security buster/updates/main amd64 Packages [610 kB]
      Get:5 http://deb.debian.org/debian buster/main amd64 Packages [7909 kB]
      Get:6 http://deb.debian.org/debian buster-updates/main amd64 Packages [8788 B]
      Fetched 8741 kB in 2s (4354 kB/s)                         

      # curl -I http://localhost:8080 
      curl: (7) Failed to connect to localhost port 8080: Connection refused
      # curl -I http://localhost:8090         
      curl: (7) Failed to connect to localhost port 8090: Connection refused
      # curl -I http://localhost:9080 
      HTTP/1.1 200 OK
      Server: nginx/1.17.8
      Date: Thu, 06 Feb 2025 17:15:55 GMT
      Content-Type: text/html
      Content-Length: 612
      Last-Modified: Tue, 21 Jan 2020 13:36:08 GMT
      Connection: keep-alive
      ETag: "5e26fe48-264"
      Accept-Ranges: bytes

      spec:
        containers:
        - image: kodekloud/nginx
          imagePullPolicy: IfNotPresent
          name: nginx
          ports:
          - containerPort: 9080
            protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /
              **port: 9080** 修正為9080
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1

      controlplane ~ ➜  kc nginx1401.yaml 
      pod/nginx1401 created

      controlplane ~ ➜  k get pod -A
      NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
      default       dev-pod-dind-878516                        3/3     Running   0          21m
      default       pod-xyz1123                                1/1     Running   0          21m
      default       webapp-color                               1/1     Running   0          18m
      dev0403       nginx0403                                  1/1     Running   0          21m
      dev0403       pod-dar85                                  1/1     Running   0          21m
      dev1401       nginx1401                                  1/1     Running   0          3s
      dev1401       pod-kab87                                  1/1     Running   0          21m
      dev2406       nginx2406                                  1/1     Running   0          21m
      dev2406       pod-var2016                                1/1     Running   0          21m
      e-commerce    e-com-1123                                 1/1     Running   0          21m
      kube-system   calico-kube-controllers-5d7d9cdfd8-ql5jv   1/1     Running   0          24m
      kube-system   canal-q9rzw                                2/2     Running   0          24m
      kube-system   canal-rmclb                                2/2     Running   0          23m
      kube-system   coredns-77d6fd4654-99vzv                   1/1     Running   0          24m
      kube-system   coredns-77d6fd4654-jd5sh                   1/1     Running   0          24m
      kube-system   etcd-controlplane                          1/1     Running   0          24m
      kube-system   kube-apiserver-controlplane                1/1     Running   0          24m
      kube-system   kube-controller-manager-controlplane       1/1     Running   0          24m
      kube-system   kube-proxy-b5m4v                           1/1     Running   0          23m
      kube-system   kube-proxy-tdl64                           1/1     Running   0          24m
      kube-system   kube-scheduler-controlplane                1/1     Running   0          24m
      marketing     redis-896d5c767-hbc2j                      1/1     Running   0          21m


      (建議添加: initialDelaySeconds: 10  # ✅ 等待 10 秒後再檢查 有時候 Readiness Probe 需要一些時間來通過，建議增加 initialDelaySeconds，減少 Readiness Probe 過早失敗的情況。)

      繼續修改文件，添加livenessProbe區塊

      controlplane ~ ➜  vim nginx1401.yaml 

      spec:
      containers:
      - image: kodekloud/nginx
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 9080
          protocol: TCP
        **livenessProbe:** 再添加此block
          exec:
            command:
              - sh
              - -c
              - ls /var/www/html/file_check  # ✅  **如果這個命令失敗，容器就會重啟**
          initialDelaySeconds: 10  # ✅  啟動後 10 秒才開始檢查
          periodSeconds: 60        # ✅  之後每 60 秒執行一次
          failureThreshold: 1      # ✅  一次失敗就重啟
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 9080
            scheme: HTTP
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources: {}

      controlplane ~ ➜  k delete pod nginx1401 -n dev1401 --force
      Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
      pod "nginx1401" force deleted

      controlplane ~ ➜  kc nginx1401.yaml 
      pod/nginx1401 created

      controlplane ~ ➜  k get pod -A
      NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
      default       dev-pod-dind-878516                        3/3     Running   0          32m
      default       pod-xyz1123                                1/1     Running   0          32m
      default       webapp-color                               1/1     Running   0          29m
      dev0403       nginx0403                                  1/1     Running   0          32m
      dev0403       pod-dar85                                  1/1     Running   0          32m
      dev1401       nginx1401                                  1/1     Running   0          2s
      dev1401       pod-kab87                                  1/1     Running   0          32m
      dev2406       nginx2406                                  1/1     Running   0          32m
      dev2406       pod-var2016                                1/1     Running   0          32m
      e-commerce    e-com-1123                                 1/1     Running   0          32m
      kube-system   calico-kube-controllers-5d7d9cdfd8-ql5jv   1/1     Running   0          35m
      kube-system   canal-q9rzw                                2/2     Running   0          35m
      kube-system   canal-rmclb                                2/2     Running   0          34m
      kube-system   coredns-77d6fd4654-99vzv                   1/1     Running   0          35m
      kube-system   coredns-77d6fd4654-jd5sh                   1/1     Running   0          35m
      kube-system   etcd-controlplane                          1/1     Running   0          35m
      kube-system   kube-apiserver-controlplane                1/1     Running   0          35m
      kube-system   kube-controller-manager-controlplane       1/1     Running   0          35m
      kube-system   kube-proxy-b5m4v                           1/1     Running   0          34m
      kube-system   kube-proxy-tdl64                           1/1     Running   0          35m
      kube-system   kube-scheduler-controlplane                1/1     Running   0          35m
      marketing     redis-896d5c767-hbc2j                      1/1     Running   0          32m




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




2. Ingress + Virtual Host Routing
    Create a single ingress resource called ingress-vh-routing. The resource should route HTTP traffic to multiple hostnames as specified below:
    The service video-service should be accessible on http://watch.ecom-store.com:30093/video
    The service apparels-service should be accessible on http://apparels.ecom-store.com:30093/wear
    To ensure that the path is correctly rewritten for the backend service, add the following annotation to the resource:
    nginx.ingress.kubernetes.io/rewrite-target: /
    Here 30093 is the port used by the Ingress Controller

    controlplane ~ ➜  k get deployment -o wide
    NAME              READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS      IMAGES                         SELECTOR
    webapp-apparels   1/1     1            1           109s   simple-webapp   kodekloud/ecommerce:apparels   app=webapp-apparels
    webapp-video      1/1     1            1           109s   simple-webapp   kodekloud/ecommerce:video      app=webapp-video

    controlplane ~ ✖ vim ingress-vh-routing.yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: ingress-vh-routing
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - host: watch.ecom-store.com
        http:
          paths:
          - path: /video
            pathType: Prefix
            backend:
              service:
                name: video-service
                port:
                  ~~number: 30093~~
      - host: apparels.ecom-store.com
        http:
          paths:
          - path: /wear
            pathType: Prefix
            backend:
              service:
                name: apparels-service
                port:
                  number: ~~30093~~

    controlplane ~ ➜  k create -f  ingress-vh-routing.yaml
    ingress.networking.k8s.io/ingress-vh-routing created


    ingress.networking.k8s.io/ingress-vh-routing edited

    controlplane ~ ➜  k get ingress
    NAME                 CLASS    HOSTS                                          ADDRESS          PORTS   AGE
    ingress-vh-routing   <none>   watch.ecom-store.com,apparels.ecom-store.com   172.20.103.190   80      4m40s



    Solution:

    controlplane ~ ➜  k get svc 
    NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    apparels-service   ClusterIP   172.20.14.91   <none>        8080/TCP   14m
    kubernetes         ClusterIP   172.20.0.1     <none>        443/TCP    62m
    video-service      ClusterIP   172.20.61.90   <none>        8080/TCP   14m

    controlplane ~ ✖ vim ingress-vh-routing.yaml

    ---
    kind: Ingress
    apiVersion: networking.k8s.io/v1
    metadata:
      name: ingress-vh-routing
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - host: watch.ecom-store.com
        http:
          paths:
          - pathType: Prefix
            path: "/video"
            backend:
              service:
                name: video-service
                port:
                  number: **8080**
      - host: apparels.ecom-store.com
        http:
          paths:
          - pathType: Prefix
            path: "/wear"
            backend:
              service:
                name: apparels-service
                port:
                  number: **8080**



    controlplane ~ ➜  k get ingress
    NAME                 CLASS    HOSTS                                          ADDRESS          PORTS   AGE
    ingress-vh-routing   <none>   watch.ecom-store.com,apparels.ecom-store.com   172.20.103.190   80      11m

    controlplane ~ ➜  curl -I http://172.20.103.190
    HTTP/1.1 404 Not Found
    Date: Fri, 07 Feb 2025 11:40:09 GMT
    Content-Type: text/html
    Content-Length: 146
    Connection: keep-alive


    controlplane ~ ➜  curl -H "Host: watch.ecom-store.com" http://172.20.103.190/video

    <!doctype html>
    <title>Hello from Flask</title>
    <body style="background: #30336b;">

    <div style="color: #e4e4e4;
        text-align:  center;
        height: 90px;
        vertical-align:  middle;">
        <img src="https://res.cloudinary.com/cloudusthad/image/upload/v1547052431/video.jpg">

    </div>

    </body>
    controlplane ~ ➜  

    controlplane ~ ➜  curl -H "Host: apparels.ecom-store.com" http://172.20.103.190/wear
    <!doctype html>
    <title>Hello from Flask</title>
    <body style="background: #2980b9;">

    <div style="color: #e4e4e4;
        text-align:  center;
        height: 90px;
        vertical-align:  middle;">
        <img src="https://res.cloudinary.com/cloudusthad/image/upload/v1547052428/apparels.jpg">

    </div>

    </body>


    Ingress 依賴 Host 來匹配流量
    Ingress 設定了 基於 Host 的路由（watch.ecom-store.com 和 apparels.ecom-store.com）。
    當你直接訪問 http://172.20.103.190/video，瀏覽器的請求 不會帶有 Host 標頭，Kubernetes Ingress Controller 無法識別該請求應該發送到哪個 Service，導致 404 Not Found。
    2️⃣ Ingress Controller 預設拒絕未匹配的請求
    當請求的 Host 與 Ingress rules 不匹配時，Ingress Controller 不會自動將流量轉發，而是返回 404 Not Found。
    解決方法：你需要告訴瀏覽器使用 watch.ecom-store.com 來發送請求（見下方解決方案）。
    3️⃣ DNS 解析問題
    watch.ecom-store.com 並不是一個公共域名，瀏覽器無法解析它到 172.20.103.190。
    解決方法：手動配置 /etc/hosts 來讓本機知道這個域名應該解析到 Ingress Controller 的 IP。



        


