First Attempt: 52% Pass Score: 75%

空題: 1,13
錯題: 2,4,5,8,10,11,12,17

### SECTION: APPLICATION DESIGN AND BUILD
1. (k logs查看initContainer )In the ckad-multi-containers namespace, we created a multi-container pod named readiness-multi-containers.

    For some reasons, it failed to run properly (Init:CrashLoopBackOff), then review the issue in the pod and try to run it again.
    NOTE: delete and recreate readiness-multi-containers pod if needed!

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1
    

    Solution: 

    First query the pod and see if it running or not:

    student-node ~ ➜  kubectl get po -n ckad-multi-containers
    NAME                         READY   STATUS                     RESTARTS     AGE
    readiness-multi-containers   0/1     **Init:CrashLoopBackOff**   1 (6s ago)   9s


    You can see its status is Init:CrashLoopBackOff, which means something went wrong with the pod.

    Check the main container logs:

    kubectl logs readiness-multi-containers -n ckad-multi-containers 
    Error from server (BadRequest): container "main-application" in pod "readiness-multi-containers" is waiting to start: **PodInitializing** # 這步有做到

    The main container can't be started as it waiting for the init container.

    Check the init container logs:

    student-node ~ ✖ **kubectl logs readiness-multi-containers -n ckad-multi-containers init-application**
    sh: slept: not found


    The issue here is slept command is not found (typo issue), you just need to update the command then the pod will start correctly.

    The command of init-container should look like below:

    initContainers:
    - command:
        - sh
        - -c
        - sleep 10

    我的Retry:
    student-node ~ ➜  kubectl config use-context cluster1
    Switched to context "cluster1".

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                         READY   STATUS                  RESTARTS      AGE
    readiness-multi-containers   0/1     Init:CrashLoopBackOff   2 (20s ago)   36s

    student-node ~ ➜  k logs pod -n ckad-multi-containers readiness-multi-containers
    error: error from server (NotFound): pods "pod" not found in namespace "ckad-multi-containers"

    student-node ~ ✖ k describe pod -n ckad-multi-containers readiness-multi-containers 
    Name:             readiness-multi-containers
    Namespace:        ckad-multi-containers
    Priority:         0
    Service Account:  default
    Node:             cluster1-node01/192.168.78.150
    Start Time:       Tue, 11 Mar 2025 23:51:39 +0000
    Labels:           <none>
    Annotations:      <none>
    Status:           Pending
    IP:               10.42.1.3
    IPs:
    IP:  10.42.1.3
    Init Containers:
    init-application:
        Container ID:  containerd://4c1edd4c7663fa591883e1798e841d1cf32c9c86774b93733f8a8003f3a90ce1
        Image:         busybox:1.28
        Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
        Port:          <none>
        Host Port:     <none>
        Command:
        sh
        -c
        slept 10
        State:          Terminated
        **Reason:       Error**
        Exit Code:    127
        Started:      Tue, 11 Mar 2025 23:53:11 +0000
        Finished:     Tue, 11 Mar 2025 23:53:11 +0000
        Last State:     Terminated
        Reason:       Error
        Exit Code:    127
        Started:      Tue, 11 Mar 2025 23:52:24 +0000
        Finished:     Tue, 11 Mar 2025 23:52:24 +0000
        Ready:          False
        Restart Count:  4
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-828zt (ro)
    Containers:
    main-application:
        Container ID:  
        Image:         busybox:1.28
        Image ID:      
        Port:          <none>
        Host Port:     <none>
        Command:
        sh
        -c
        echo The app is running! && sleep 3600
        State:          Waiting
        Reason:       PodInitializing
        Ready:          False
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-828zt (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 False 
    Ready                       False 
    ContainersReady             False 
    PodScheduled                True 
    Volumes:
    kube-api-access-828zt:
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
    Type     Reason     Age                 From               Message
    ----     ------     ----                ----               -------
    Normal   Scheduled  102s                default-scheduler  Successfully assigned ckad-multi-containers/readiness-multi-containers to cluster1-node01
    Normal   Pulling    102s                kubelet            Pulling image "busybox:1.28"
    Normal   Pulled     102s                kubelet            Successfully pulled image "busybox:1.28" in 211ms (211ms including waiting)
    Normal   Pulled     11s (x4 over 102s)  kubelet            Container image "busybox:1.28" already present on machine
    Normal   Created    11s (x5 over 102s)  kubelet            Created container init-application
    Normal   Started    11s (x5 over 102s)  kubelet            Started container init-application
    Warning  BackOff    10s (x8 over 101s)  kubelet            Back-off restarting failed container init-application in pod readiness-multi-containers_ckad-multi-containers(e8dbefc8-b59e-4208-8446-ef9c4ecb1090)

    student-node ~ ➜  k get events -n ckad-multi-containers 
    LAST SEEN   TYPE      REASON      OBJECT                           MESSAGE
    117s        Normal    Scheduled   pod/readiness-multi-containers   Successfully assigned ckad-multi-containers/readiness-multi-containers to cluster1-node01
    116s        Normal    Pulling     pod/readiness-multi-containers   Pulling image "busybox:1.28"
    116s        Normal    Pulled      pod/readiness-multi-containers   Successfully pulled image "busybox:1.28" in 211ms (211ms including waiting)
    25s         Normal    Pulled      pod/readiness-multi-containers   Container image "busybox:1.28" already present on machine
    25s         Normal    Created     pod/readiness-multi-containers   Created container init-application
    25s         Normal    Started     pod/readiness-multi-containers   Started container init-application
    10s         Warning   BackOff     pod/readiness-multi-containers   **Back-off restarting failed container init-application in pod readiness-multi-containers_ckad-multi-containers**(e8dbefc8-b59e-4208-8446-ef9c4ecb1090)

    student-node ~ ➜  k get pod -n ckad-multi-containers readiness-multi-containers -o yaml > readiness-multi-containers.yaml

    student-node ~ ➜  vim readiness-multi-containers.yaml (修改init-container command錯字)

    student-node ~ ➜  k replace -f readiness-multi-containers.yaml --force
    pod "readiness-multi-containers" deleted
    pod/readiness-multi-containers replaced

    student-node ~ ➜  k describe pod -n ckad-multi-containers readiness-multi-containers 
    Name:             readiness-multi-containers
    Namespace:        ckad-multi-containers
    Priority:         0
    Service Account:  default
    Node:             cluster1-node01/192.168.78.150
    Start Time:       Tue, 11 Mar 2025 23:55:25 +0000
    Labels:           <none>
    Annotations:      <none>
    Status:           Running
    IP:               10.42.1.4
    IPs:
    IP:  10.42.1.4
    Init Containers:
    init-application:
        Container ID:  containerd://7b00cefb963fda665ba1d5aae0a92122bf8579ca4c1c61d701f0dcab8ca0fa56
        Image:         busybox:1.28
        Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
        Port:          <none>
        Host Port:     <none>
        Command:
        sh
        -c
        sleep 10
        State:          Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Tue, 11 Mar 2025 23:55:26 +0000
        Finished:     Tue, 11 Mar 2025 23:55:36 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-828zt (ro)
    Containers:
    main-application:
        Container ID:  containerd://4c2532d134d124d806817d575643d0206c9e7da43a671ed6d0ce7c2bf0bdea3c
        Image:         busybox:1.28
        Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
        Port:          <none>
        Host Port:     <none>
        Command:
        sh
        -c
        echo The app is running! && sleep 3600
        State:          Running
        Started:      Tue, 11 Mar 2025 23:55:37 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-828zt (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       True 
    ContainersReady             True 
    PodScheduled                True 
    Volumes:
    kube-api-access-828zt:
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
    Type    Reason   Age   From     Message
    ----    ------   ----  ----     -------
    Normal  Pulled   14s   kubelet  Container image "busybox:1.28" already present on machine
    Normal  Created  14s   kubelet  Created container init-application
    Normal  Started  14s   kubelet  Started container init-application
    Normal  Pulled   3s    kubelet  Container image "busybox:1.28" already present on machine
    Normal  Created  3s    kubelet  Created container main-application
    Normal  Started  3s    kubelet  Started container main-application

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                         READY   STATUS     RESTARTS   AGE
    readiness-multi-containers   0/1     Init:0/1   0          5s

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                         READY   STATUS    RESTARTS   AGE
    readiness-multi-containers   1/1     Running   0          118s

    student-node ~ ➜  k get pod -n ckad-multi-containers --watch
    NAME                         READY   STATUS    RESTARTS        AGE
    readiness-multi-containers   1/1     Running   1 (3m36s ago)   63m


2. (待測試) In the ckad-multi-containers namespace, create a pod named healthy-server, which consists of 2 containers. One main container and one init-container both are running busybox:1.28 image.

    Init container should print this message Initialize application environment! and then sleep for 10 seconds.
    Main container should print this message The app is running! and then sleep for 3600 seconds.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    我的配置:
    student-node ~ ➜  cat healthy-server.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
        creationTimestamp: null
        labels:
            run: healthy-server
        name: healthy-server
        namespace: ckad-multi-containers
    spec:
        containers:
        - image: busybox:1.28
            name: main-container
            command:
            - /bin/sh
            - -c
            - echo "The app is running!" && sleep 3600
        initContainers:
        - image: busybox:1.28
            name: init-container
            command:
            - /bin/sh
            - -c
            - echo "Initialize application environment!" && sleep 10
            resources: {}
        dnsPolicy: ClusterFirst
        restartPolicy: Always
    status: {}

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                         READY   STATUS                  RESTARTS       AGE
    healthy-server               1/1     Running                 2 (17m ago)    138m


    Solution:
    Use below YAML to create required pod:

    apiVersion: v1
    kind: Pod
        metadata:
        name: healthy-server
        namespace: ckad-multi-containers
        labels:
            app.kubernetes.io/name: healthy-server
    spec:
        containers:
            - name: server-container
            image: busybox:1.28
            command:
                - sh
                - -c
                - echo The app is running! && sleep 3600
        initContainers:
            - name: init-myservice
            image: busybox:1.28
            command:
                - sh
                - -c
                - echo Initialize application environment! && sleep 10

    我的Retry:

    Details

    O Is the healthy-server running?
    O Does main container run busybox:1.28 image?
    X Does main container run the desired command?
    O Does init container run busybox:1.28 image?
    X Does init container run the desired command?

    student-node ~ ✖ k  run healthy-server -n ckad-multi-containers --image=busybox:1.28 --dry-run=client -o yaml > healthy-server.yaml

    student-node ~ ➜  cat healthy-server.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: healthy-server
    name: healthy-server
    namespace: ckad-multi-containers
    spec:
    containers:
    - image: busybox:1.28
        name: main-container
        command: ["/bin/sh","-c", "echo 'The app is running!' && sleep 3600"]
        resources: {}
    initContainers:
    - image: busybox:1.28
        name: init-container
        command: ["/bin/sh","-c", "echo 'Initialize application environment!' && sleep 10"]
        resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    student-node ~ ➜  k create -f  healthy-server.yaml 
    pod/healthy-server created

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                         READY   STATUS     RESTARTS   AGE
    readiness-multi-containers   1/1     Running    0          5m28s
    healthy-server               0/1     Init:0/1   0          5s

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                         READY   STATUS    RESTARTS   AGE
    readiness-multi-containers   1/1     Running   0          5m41s
    healthy-server               1/1     Running   0          18s

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                         READY   STATUS    RESTARTS   AGE
    readiness-multi-containers   1/1     Running   0          6m22s
    healthy-server               1/1     Running   0          59s

    從你的 kubectl logs 結果來看，init-container 和 main-container 都成功執行了 echo 指令並輸出訊息，但 KodeKloud 仍然判斷**"Does main container run the desired command?"** 和 "Does init container run the desired command?" 為失敗。這可能與 exact string match（嚴格字串匹配）或 格式問題 有關。

    1️⃣ 你的 command 可能沒有完全匹配 KodeKloud 預期的格式
    你的 YAML 使用：
    command: ["/bin/sh","-c", "echo 'The app is running!' && sleep 3600"]
    但 echo 的引號可能影響了輸出格式（KodeKloud 可能用 grep 或 diff 來驗證輸出，可能會因為引號不同而判定不符合要求）。

    解決方法
    試試移除 echo 的單引號：
    command: ["/bin/sh","-c", "echo The app is running! && sleep 3600"]

    2️⃣ restartPolicy: Always 可能影響驗證
    你的 YAML 有：
    restartPolicy: Always
    但 initContainer 執行完成後應該不會再重啟，而且 main-container 只是一個長時間運行的容器。
    KodeKloud 可能預期 restartPolicy 為 Never 或 OnFailure，避免 initContainer 影響驗證。

    Third Try:

    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: healthy-server
    name: healthy-server
    namespace: ckad-multi-containers
    spec:
    containers:
        - image: busybox:1.28
        name: main-container
        command: ["/bin/sh","-c", "echo The app is running! && sleep 3600"]
        resources: {}
    initContainers:
        - image: busybox:1.28
        name: init-container
        command: ["/bin/sh","-c", "echo Initialize application environment! && sleep 10"]
        resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}
    ~          

    (本次被驗證成功)
    結論: 拿掉''單引號，最後initContainer不設置restartPolicy: Always，建議為Never


3. (待測試) In the ckad-job namespace, create a job named pi that simply computes a π (pi) to 2000 places and prints it out.

    This job should be configured to retry maximum 3 times before marking this job failed, and the duration of this job should not exceed 100 seconds.

    Use perl:5.34.0 image for your container.

    我的配置:

    student-node ~ ➜  k get job -n ckad-job 
    NAME   COMPLETIONS   DURATION   AGE
    ppi    1/1           10s        130m

    student-node ~ ➜  vim p
    pod-metrics  ppi.yaml     

    student-node ~ ➜  cat ppi.yaml 
    apiVersion: batch/v1
    kind: Job
        metadata:
        creationTimestamp: null
        name: ppi
        namespace: ckad-job
    spec:
        backoffLimit: 3
        activeDeadlineSeconds: 100
        template:
            metadata:
            creationTimestamp: null
            spec:
                containers:
                - image: perl:5.34.0
                    name: ppi
                    command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)","sleep 7200"]
                    resources: {}
            restartPolicy: Never
    status: {}

    所以如果Job最終為Succeded，則視為正確 (不應該出現CrashLoopBackOff)
    如果Job為CrashLoopBackOff，則檢查RESTART次數如果達到BackOfLimit，則手動刪除重新創建該Job,讓pod重新運行。

    Solution:
    Use below YAML to create job:

    apiVersion: batch/v1
    kind: Job
    metadata:
        name: pi
        namespace: ckad-job
    spec:
        backoffLimit: 3
        activeDeadlineSeconds: 100
        template:
            spec:
            containers:
            - name: pi
                image: perl:5.34.0
                command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
            restartPolicy: Never

    我的Retry:
    student-node ~ ➜  kubectl config use-context cluster2
    Switched to context "cluster2".

    student-node ~ ➜  k create job pi -n ckad-job --image=perl:5.34.0 --dry-run=client -o yaml > pi.yaml

    student-node ~ ➜  vim pi.yaml 

    student-node ~ ➜  k create -f  pi.yaml 
    Error from server (BadRequest): error when creating "pi.yaml": Job in version "v1" cannot be handled as a Job: strict decoding error: unknown field "spec.template.spec.containers[0].cmmand"

    student-node ~ ✖ vim pi.yaml 

    student-node ~ ➜  k create -f  pi.yaml 
    job.batch/pi created

    student-node ~ ➜  k get job -n ckad-job 
    NAME   COMPLETIONS   DURATION   AGE
    pi     0/1           10s        10s

    student-node ~ ➜  k describe job -n ckad-job 
    Name:                     pi
    Namespace:                ckad-job
    Selector:                 batch.kubernetes.io/controller-uid=7508bc59-e928-40fd-b4bb-66469f46f584
    Labels:                   batch.kubernetes.io/controller-uid=7508bc59-e928-40fd-b4bb-66469f46f584
                            batch.kubernetes.io/job-name=pi
                            controller-uid=7508bc59-e928-40fd-b4bb-66469f46f584
                            job-name=pi
    Annotations:              <none>
    Parallelism:              1
    Completions:              1
    Completion Mode:          NonIndexed
    Suspend:                  false
    Backoff Limit:            3
    Start Time:               Wed, 12 Mar 2025 00:08:41 +0000
    Active Deadline Seconds:  100s
    Pods Statuses:            1 Active (0 Ready) / 0 Succeeded / 0 Failed
    Pod Template:
    Labels:  batch.kubernetes.io/controller-uid=7508bc59-e928-40fd-b4bb-66469f46f584
            batch.kubernetes.io/job-name=pi
            controller-uid=7508bc59-e928-40fd-b4bb-66469f46f584
            job-name=pi
    Containers:
    pi:
        Image:      perl:5.34.0
        Port:       <none>
        Host Port:  <none>
        Command:
        perl
        -Mbignum=bpi
        -wle
        print bpi(2000)
        Environment:   <none>
        Mounts:        <none>
    Volumes:         <none>
    Node-Selectors:  <none>
    Tolerations:     <none>
    Events:
    Type    Reason            Age   From            Message
    ----    ------            ----  ----            -------
    Normal  SuccessfulCreate  20s   job-controller  Created pod: pi-j5gvm

    student-node ~ ➜  k get pod -n ckad-job 
    NAME       READY   STATUS        RESTARTS   AGE
    pi-j5gvm   0/1     **Completed**   0          31s


### SECTION: APPLICATION DEPLOYMENT
5. (使題考察Helm，但我會錯意) The team Garuda has deployed one application in the testing-apd namespace. 
    
    The testing was done, and the team wants you to delete that release. Find out the release name and delete it.    
    NOTE: - Do not worry about the status of the deployment.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    我的配置: (忘記這是在考helm)

    student-node ~ ➜  k get all -n testing-apd 
    NAME                        READY   STATUS    RESTARTS   AGE
    pod/image-scanner-trivy-0   1/1     Running   0          22s

    NAME                          TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
    service/image-scanner-trivy   ClusterIP   10.43.96.25   <none>        4954/TCP   22s

    NAME                                   READY   AGE
    statefulset.apps/image-scanner-trivy   1/1     22s

    (以為是scale statefulset為0)

    student-node ~ ➜  helm ls -n testing-apd
    NAME            NAMESPACE       REVISION        UPDATED                                 STATUS       CHART           APP VERSION
    image-scanner   testing-apd     1               2025-03-11 19:27:36.198060092 +0000 UTC deployed     trivy-0.12.0    0.60.0    

    Solution
    Run the following command to change the context: -
    kubectl config use-context cluster2

    On the cluster2, testing application is running on the testing-apd namespace, which we must delete because it is consuming resources.

    helm ls -n testing-apd
    helm uninstall -n testing-apd image-scanner 


6. (答對 驗證流量的方式只需要看Service是否設置了跟Deployment相同的label即可) We have deployed two applications called circle-apd and square-apd on the default namespace using the kodekloud/webapp-color:v1 and kodekloud/webapp-color:v2.

    We have done all the tests and do not want circle-apd deployment to receive traffic from the foundary-svc service anymore. So, route all the traffic to another existing deployment.

    Do change the service specifications to route traffic to the square-apd deployment.


    You can test the application from the terminal by running the curl command with the following syntax: -

    curl http://cluster3-controlplane:NODE-PORT
    <!doctype html>
    <title>Hello from Flask</title>
    ...
    <h2>
        Application Version: v2
    </h2>

    As shown above, we will get the Application Version: v2 in the output.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3


8. (Helm) One application, webpage-server-01, is deployed on the Kubernetes cluster by the Helm tool. Now, the team wants to deploy a new version of the application by replacing the existing one. A new version of the helm chart is given in the /root/new-version directory on the student-node. Validate the chart before installing it on the Kubernetes cluster. 

    Use the helm command to validate and install the chart. After successfully installing the newer version, uninstall the older version. 

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    Solution
    Run the following command to change the context: -
    kubectl config use-context cluster1

    In this task, we will use the helm commands. Here are the steps: -
    Use the helm ls command to list the release deployed on the default namespace using helm.
    helm ls -n default

    First, validate the helm chart by using the helm lint command: -
    cd /root/

    helm lint ./new-version # 此步驟我有做


    Now, install the new version of the application by using the helm install command as follows: -
    helm install --generate-name ./new-version


    We haven't got any release name in the task, so we can generate the random name from the --generate-name option.

    Finally, uninstall the old version of the application by using the helm uninstall command: -
    helm uninstall webpage-server-01 -n default # 我刪錯


10. (Helm **show出values.yaml內參數**) One co-worker deployed an **nginx** helm chart on the cluster1 server called bitnami. A new update is pushed to the helm chart, and the team wants you to update the helm repository to fetch the new changes.

    After updating the helm chart, upgrade the helm chart version to 18.3.0 and increase the replica count to 2.
    NOTE: - We have to perform this task on the cluster1-controlplane node.

    You can SSH into the cluster1 using ssh cluster1-controlplane command.

    我的配置:

    student-node ~ ✖ ssh cluster1-controlplane

    cluster1-controlplane ~ ➜  helm repo list
    NAME    URL                               
    bitnami https://charts.bitnami.com/bitnami

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    我的配置:
    cluster1-controlplane ~ ➜  helm repo update bitnami
    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "bitnami" chart repository
    Update Complete. ⎈Happy Helming!⎈

    cluster1-controlplane ~ ➜  helm upgrade bitnami 
    Error: "helm upgrade" requires 2 arguments

    Usage:  helm upgrade [RELEASE] [CHART] [flags]

    cluster1-controlplane ~ ✖ helm search repo bitnami
    NAME                                            CHART VERSION   APP VERSION     DESCRIPTION                                       
    bitnami/airflow                                 22.7.0          2.10.5          Apache Airflow is a tool to express and execute...
    bitnami/apache                                  11.3.4          2.4.63          Apache HTTP Server is an open-source HTTP serve...
    bitnami/apisix                                  4.2.0           3.11.0          Apache APISIX is high-performance, real-time AP...
    bitnami/appsmith                                5.2.1           1.62.0          Appsmith is an open source platform for buildin...
    bitnami/argo-cd                                 7.2.3           2.14.5          Argo CD is a continuous delivery tool for Kuber...
    bitnami/argo-workflows                          11.1.10         3.6.5           Argo Workflows is meant to orchestrate Kubernet...
    bitnami/aspnet-core                  

    cluster1-controlplane ~ ➜  helm upgrade  **bitnami/nginx** -n ckad10-finance-ns --version=18.3.0
    Error: "helm upgrade" requires 2 arguments

    Usage:  helm upgrade [RELEASE] [CHART] [flags]


    cluster1-controlplane ~ ➜  helm show chart bitnami/nginx -n ckad10-finance-ns|grep version
    Pulled: us-central1-docker.pkg.dev/kk-lab-prod/helm-charts/bitnami/nginx:19.0.0
    Digest: sha256:e01e0935b7d0e4739c67897bc9205f5c1eb3b857d22629b6b3abaa3f4429d42f
    version: 2.x.x
    version: 19.0.0


    cluster1-controlplane ~ ✖ helm upgrade  bitnami bitnami/nginx -n ckad10-finance-ns --version=18.3.
    0
    Pulled: us-central1-docker.pkg.dev/kk-lab-prod/helm-charts/bitnami/nginx:18.3.0
    Digest: sha256:01518303d92f8a7d07c21774f019818f3a3b8717d55da313908711e02737b2df
    Release "bitnami" has been upgraded. Happy Helming!
    NAME: bitnami
    LAST DEPLOYED: Tue Mar 11 20:22:02 2025
    NAMESPACE: ckad10-finance-ns
    STATUS: deployed
    REVISION: 2
    TEST SUITE: None
    NOTES:
    CHART NAME: nginx
    **CHART VERSION: 18.3.0**
    APP VERSION: 1.27.3
    ...

    REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
    1               Tue Mar 11 19:57:16 2025        superseded      nginx-17.0.0    1.26.0          Install complete
    2               Tue Mar 11 20:22:02 2025        superseded      nginx-18.3.0    1.27.3          Upgrade complete
    3               Tue Mar 11 20:24:20 2025        deployed        nginx-18.3.0    1.27.3          Upgrade complete

    欲得知--set 參數:
    法一: 至values.yaml查找
    法二: 列出當前 Release 的 values    
    ~~helm get values bitnami -n ckad10-finance-ns~~
    
    **helm show values** bitnami/nginx

    cluster1-controlplane ~ ➜  helm show values bitnami/nginx| grep Replica
    Pulled: us-central1-docker.pkg.dev/kk-lab-prod/helm-charts/bitnami/nginx:19.0.0
    Digest: sha256:e01e0935b7d0e4739c67897bc9205f5c1eb3b857d22629b6b3abaa3f4429d42f
    /## @param autoscaling.minReplicas Minimum number of replicas to scale back
    /## @param autoscaling.maxReplicas Maximum number of replicas to scale out
    minReplicas: ""
    maxReplicas: ""

    cluster1-controlplane ~ ➜  helm show values bitnami/nginx| grep ReplicaCount
    Pulled: us-central1-docker.pkg.dev/kk-lab-prod/helm-charts/bitnami/nginx:19.0.0
    Digest: sha256:e01e0935b7d0e4739c67897bc9205f5c1eb3b857d22629b6b3abaa3f4429d42f

    cluster1-controlplane ~ ✖ **helm show values bitnami/nginx| grep replicaCount**
    Pulled: us-central1-docker.pkg.dev/kk-lab-prod/helm-charts/bitnami/nginx:19.0.0
    Digest: sha256:e01e0935b7d0e4739c67897bc9205f5c1eb3b857d22629b6b3abaa3f4429d42f
    /## @param replicaCount Number of NGINX replicas to deploy
    replicaCount: 1


    Solution
    Run the following command to change the context: -
    kubectl config use-context cluster1

    In this task, we will use the kubectl and helm commands. Here are the steps: -
    Log in to the cluster1-controlplane node first and use the helm ls command to list all the releases installed using Helm in the Kubernetes cluster.

    helm ls -A

    Here -A or --all-namespaces option lists all the releases of all the namespaces.
    Identify the namespace where the resources get deployed.

    Use the helm repo ls command to list the helm repositories.
    helm repo ls 

    Now, update the helm repository with the following command: -
    helm repo update bitnami -n ckad10-finance-ns

    The above command updates the local cache of available charts from the configured chart repositories.

    The helm search command searches for all the available charts in a specific Helm chart repository. In our case, it's the nginx helm chart.
    helm search repo bitnami/nginx -n ckad10-finance-ns -l | head -n30

    The -l or --versions option is used to display information about all available chart versions.

    Upgrade the helm chart to 18.3.0 and also, increase the replica count of the deployment to 2 from the command line. Use the helm upgrade command as follows: -

    helm upgrade bitnami bitnami/nginx -n ckad10-finance-ns **--version=18.3.0 --set replicaCount=2**
    **--set 參數的來源 Helm Chart 的 --set 參數對應的是 values.yaml 中的 key-value 設定**

    After upgrading the chart version, you can verify it with the following command: -
    helm ls -n ckad10-finance-ns

    Look under the CHART column for the chart version.

    Use the kubectl get command to check the replicas of the deployment: -
    kubectl get deploy -n ckad10-finance-ns

    The available count 2 is under the AVAILABLE column.

### SECTION: SERVICES AND NETWORKING
11. We have already deployed an ingress resource in the app-space namespace.

    But for better SEO practices, you are requested to change the URLs at which the applications are made available.

    Change the path of the video application to make it available at /stream.

    Note: Check the backend services configured for the paths in the ingress resource.
    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3
    
    我的配置:
    
    O Does path /wear select wear-service?
    O Does path /stream select video application?
    X Is backend port for /wear service correct?
    X Is backend port for /stream service service?

    student-node ~ ➜  k get all -n app-space 
    NAME                                   READY   STATUS    RESTARTS   AGE
    pod/default-backend-79755fc44c-8fht4   1/1     Running   0          75m
    pod/webapp-video-74bf8df7f5-w6z7n      1/1     Running   0          75m
    pod/webapp-wear-7b784f68d8-nxhcw       1/1     Running   0          75m

    NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    service/default-backend-service   ClusterIP   172.20.242.147   <none>        80/TCP     75m
    service/video-service             ClusterIP   172.20.240.88    <none>        8080/TCP   75m
    service/wear-service              ClusterIP   172.20.63.234    <none>        8080/TCP   75m

    NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/default-backend   1/1     1            1           75m
    deployment.apps/webapp-video      1/1     1            1           75m
    deployment.apps/webapp-wear       1/1     1            1           75m

    NAME                                         DESIRED   CURRENT   READY   AGE
    replicaset.apps/default-backend-79755fc44c   1         1         1       75m
    replicaset.apps/webapp-video-74bf8df7f5      1         1         1       75m
    replicaset.apps/webapp-wear-7b784f68d8       1         1         1       75m

    student-node ~ ➜  k get pod -n app-space -o wide
    NAME                               READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
    default-backend-79755fc44c-8fht4   1/1     Running   0          75m   172.17.1.33   cluster3-node01   <none>           <none>
    webapp-video-74bf8df7f5-w6z7n      1/1     Running   0          75m   172.17.1.32   cluster3-node01   <none>           <none>
    webapp-wear-7b784f68d8-nxhcw       1/1     Running   0          75m   172.17.1.31   cluster3-node01   <none>           <none>

    spec:
    ingressClassName: nginx
    rules:
    - http:
        paths:
        - backend:
            service:
                name: webapp-wear # 原本為wear-service
                port:
                number: 8080
            path: /wear
            pathType: Prefix
        - backend:
            service:
                name: webapp-video # 原本為video-service
                port:
                number: 8080
            path: /stream
            pathType: Prefix


    Solution:
    To change the path of the service, you need to edit the ingress

    Edit the ingress using kubectl edit -n app-space ingress ingress-resource-svcn.
    Edit the /watch to make it as /stream.
    spec:
    ingressClassName: nginx
    rules:
    - http:
        paths:
        - backend:
            service:
                name: wear-service
                port:
                number: 8080
            path: /wear
            pathType: Prefix
        - backend:
            service:
                name: video-service
                port:
                number: 8080
            path: /watch  #change path to /stream
            pathType: Prefix
    status:
    loadBalancer:
        ingress:
        - ip: 10.108.22.212


12. (未弄清楚egress出站規則影響到的資源為何) We have deployed some pods in the namespaces ckad-alpha and ckad-beta.

    You need to create a NetworkPolicy named ns-netpol-ckad that will restrict all Pods in Namespace ckad-alpha to only have outgoing traffic to Pods in Namespace ckad-beta . Ingress traffic should not be affected.

    However, the NetworkPolicy you create should allow egress traffic on port 53 TCP and UDP.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3
    
    我的配置:
    student-node ~ ➜  k get all -n ckad-alpha 
    NAME             READY   STATUS    RESTARTS   AGE
    pod/ckad-pod-1   1/1     Running   0          11s

    NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    service/ckad-pod-1   ClusterIP   172.20.19.125   <none>        80/TCP    11s

    student-node ~ ➜  k get all -n ckad-beta 
    NAME             READY   STATUS    RESTARTS   AGE
    pod/ckad-pod-2   1/1     Running   0          23s

    NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    service/ckad-pod-2   ClusterIP   172.20.129.94   <none>        80/TCP    23s

    student-node ~ ➜  vim ns-netpol-ckad.yaml

    student-node ~ ➜  k get pod -n ckad-alph -o wide
    No resources found in ckad-alph namespace.

    student-node ~ ➜  k get pod -n ckad-alpa -o wide
    No resources found in ckad-alpa namespace.

    student-node ~ ➜  k get pod ckad-pod-1 -n ckad-alpa -o wide
    Error from server (NotFound): namespaces "ckad-alpa" not found

    student-node ~ ✖ k get pod -n ckad-alpha ckad-pod-1 --show-labels 
    NAME         READY   STATUS    RESTARTS   AGE     LABELS
    ckad-pod-1   1/1     Running   0          3m45s   run=ckad-pod-1

    student-node ~ ➜  k get pod -n ckad-beta ckad-pod-2 --show-labels 
    NAME         READY   STATUS    RESTARTS   AGE     LABELS
    ckad-pod-2   1/1     Running   0          3m54s   run=ckad-pod-2

    student-node ~ ➜  vim ns-netpol-ckad.yaml

    student-node ~ ➜  k get ns --show-labels 
    NAME              STATUS   AGE     LABELS
    app-space         Active   10m     kubernetes.io/metadata.name=app-space
    blue-apd          Active   60m     kubernetes.io/metadata.name=blue-apd
    ckad-alpha        Active   4m51s   kubernetes.io/metadata.name=ckad-alpha
    ckad-beta         Active   4m51s   kubernetes.io/metadata.name=ckad-beta
    ckad9-uat         Active   51m     kubernetes.io/metadata.name=ckad9-uat
    default           Active   106m    kubernetes.io/metadata.name=default
    ingress-nginx     Active   10m     app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx,kubernetes.io/metadata.name=ingress-nginx
    kube-node-lease   Active   106m    kubernetes.io/metadata.name=kube-node-lease
    kube-public       Active   106m    kubernetes.io/metadata.name=kube-public
    kube-system       Active   106m    kubernetes.io/metadata.name=kube-system

    student-node ~ ➜  vim ns-netpol-ckad.yaml

    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
        name: ns-netpol-ckad
        namespace: ckad-alpha
    spec:
        podSelector:
            matchLabels:
                ~~run: ckad-pod-1~~ # 不應設置label! 因為這只會影響有run=ckad-pod-1這個標籤的pod所有 Pod
                應設置為: **podSelector: {}**  # **影響 ckad-alpha 命名空間內的所有資源**
        policyTypes:
        - Egress
        egress:
        - to:
        - namespaceSelector:
            matchLabels:
                kubernetes.io/metadata.name: ckad-beta
        ~~- podSelector:~~ # 不用設置
            matchLabels:
                run: ckad-pod-2 
        ports:
        - protocol: TCP
        port: 53
        - protocol: UDP
        port: 53

    student-node ~ ➜  k get netpol -n ckad-alpha 
    NAME             POD-SELECTOR     AGE
    ns-netpol-ckad   run=ckad-pod-1   8s

    
    Solution:
    The following manifest will restrict all the pods in ckad-alpha namespace to only have egress traffic to pods in namespace ckad-beta. But it will allow egress on port 53.

    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
    name: ns-netpol-ckad
    namespace: ckad-alpha
    spec:
    podSelector: {}
    policyTypes:
    - Egress
    egress:
    - ports:
        - port: 53
        protocol: TCP
        - port: 53
        protocol: UDP   
    - to:
        - namespaceSelector:
            matchLabels:
            kubernetes.io/metadata.name: ckad-beta

13. (空題Solving Ingress Issue ) We have deployed an application in the green-space namespace. we also deployed the ingress controller and the ingress resource.

    However, currently, the ingress controller is not working as expected. Inspect the ingress definitions and troubleshoot the issue so that the services are accessible as per the ingress resource definition.

    Note: You are allowed to edit or delete the resources related to ingress but do not change the pods.  

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    Solution
    Check the status of the ingress, pods, and application related services.

    cluster2-controlplane ~ ➜  **k get pods -n ingress-nginx** 
    NAME                                        READY   STATUS      RESTARTS      AGE
    ingress-nginx-admission-create-l6fgw        0/1     Completed   0             11m
    ingress-nginx-admission-patch-sfgc4         0/1     Completed   0             11m
    ingress-nginx-controller-5f8964959d-278rc   0/1     Error       2 (26s ago)   29s

    You would see an Error or CrashLoopBackOff in the ingress-nginx-controller. Inspect the logs of the controller pod.

    cluster2-controlplane ~ ✖ k logs -n ingress-nginx ingress-nginx-controller-5f8964959d-278rc 
    /-------------------------------------------------------------------------------
    /--------
    F0316 08:03:28.111614      57 main.go:83] No service with name default-backend-service found in namespace default:


    You see an error msg saying "No service with name default-backend-service found in namespace default".

    We don't have the service with that name in the default namespace, so we need to edit the ingress controller deployment to use the service that we have .i.e. default-backend-service in the green-space namespace.
    To create the controller deployment with correct backend service, first save the deployment in a file, delete the controller deployment, edit the file and create the deployment.
    Save the deployment in file

    k get -n ingress-nginx deployments.apps ingress-nginx-controller -o yaml >> ing-control.yaml

    Delete the deployment.
    k delete -n ingress-nginx deploy ingress-nginx-controller
    Edit the file to match the correct service.

     spec:
        containers:
        - args:
          - /nginx-ingress-controller
          - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
          - --election-id=ingress-controller-leader
          - --watch-ingress-without-class=true
          - --default-backend-service=green-space/default-backend-service   #Changed to correct namespace
          - --controller-class=k8s.io/ingress-nginx
          - --ingress-class=nginx
          - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
          - --validating-webhook=:8443
          - --validating-webhook-certificate=/usr/local/certificates/cert
          - --validating-webhook-key=/usr/local/certificates/key
    Apply the manifest, it should be up and running.

    You can verify the application is running via going to host cluster2-controlplane: ssh cluster2-controlplane and curl request to the application endpoint

    /# App Video Service
    curl -H "Host: app-video.localhost" http://localhost:30080

    /# App Wear Service 
    curl -H "Host: app-wear.localhost" http://localhost:30080


15. (答對) Create a Kubernetes LimitRange object named ckad15-memlt-aecs. This object limits the amount of memory each container in the namespace ckad15-memlt-ns-aecs can use.

    The default memory limit should be 512Mi, and the default memory request should be 256Mi. Ensure that the manifest applies to container resources only.

    Note: You may need to create or delete resources to complete this task.

    student-node ~ ✖ vim ckad15-memlt-aecs.yaml

    student-node ~ ➜  kc ckad15-memlt-aecs.yaml
    limitrange/ckad15-memlt-aecs created

    student-node ~ ➜  k get limitranges -n ckad15-memlt-ns-aecs 
    NAME                CREATED AT
    ckad15-memlt-aecs   2025-03-11T20:52:03Z

    student-node ~ ➜  cat ckad15-memlt-aecs.yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
    name: ckad15-memlt-aecs
    namespace: ckad15-memlt-ns-aecs
    spec:
    limits:
    - default: # this section defines default limits
        memory: 512Mi
        defaultRequest: # this section defines default requests
        memory: 256Mi
        type: Container

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY

17.  (**k get all 不會列出 Secret 資源!!**) 
    
    Update secret/ckad07-sec-cr-aecs in the default namespace with one additional credential stating the Message of the day as given below:

    Name: ckad07-sec-cr-aecs

    Creds:
    key: motd, value: We make DevOps shine!

    Note: Make only the necessary changes. Do not modify other fields of the secret.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    student-node ~ ➜  k get all
    NAME                                     READY   STATUS    RESTARTS   AGE
    pod/webpage-server-02-55566ffb45-ln62v   1/1     Running   0          23m
    pod/webpage-server-02-55566ffb45-ddprm   1/1     Running   0          23m
    pod/webpage-server-02-55566ffb45-8g49x   1/1     Running   0          23m

    NAME                            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    service/kubernetes              ClusterIP   10.43.0.1      <none>        443/TCP          115m
    service/webpage-server-02-svc   NodePort    10.43.93.251   <none>        8080:30702/TCP   23m

    NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/webpage-server-02   3/3     3            3           23m

    NAME                                           DESIRED   CURRENT   READY   AGE
    replicaset.apps/webpage-server-02-55566ffb45   3         3         3       23m

    student-node ~ ➜  k get all -A|grep ckad07

    student-node ~ ➜  **k get secret/ckad07-sec-cr-aecs**
    NAME                 TYPE     DATA   AGE
    ckad07-sec-cr-aecs   Opaque   2      43s


    OR **直接 k get secret**
    student-node ~ ✖ k get secret
    NAME                                           TYPE                 DATA   AGE
    sh.helm.release.v1.new-version-1741738845.v1   helm.sh/release.v1   1      26m
    ckad07-sec-cr-aecs                             Opaque               2      3m28s


    Solution:
    student-node ~ ➜  kubectl config use-context cluster1
    Switched to context "cluster1".

    student-node ~ ➜  k get secret/ckad07-sec-cr-aecs -o yaml > secret-motd.yaml

    student-node ~ ➜  echo 'We make DevOps shine!' | base64 
    V2UgbWFrZSBEZXZPcHMgc2hpbmUhCg==

    student-node ~ ➜  vim secret-motd.yaml

    student-node ~ ➜  cat secret-motd.yaml 
    apiVersion: v1
        data:
            eligibility: RGV2T3BzR3V5cw==
                supporters: S29kZUtsb3VkIFRlYW0=
                motd: V2UgbWFrZSBEZXZPcHMgc2hpbmUhCg==  #added
    kind: Secret
    metadata:
        name: ckad07-sec-cr-aecs
        namespace: default
    type: Opaque

    student-node ~ ➜  k apply -f secret-motd.yaml 
    secret/ckad07-sec-cr-aecs configured

    student-node ~ ➜  kubectl get secret/ckad07-sec-cr-aecs -o go-template='{{.data.motd | base64decode}}'
    We make DevOps shine!





### SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE 全對