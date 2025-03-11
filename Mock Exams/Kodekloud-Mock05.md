First Attempt Score:  54%
答對: 03,04,06,09,13~16,18~21
錯題: 01,02,05,07,08,10,11,12,17

不熟的Topic: 02 (Persistent Volume回收機制) 08 (Helm uninstall 之後檢查是否有殘留資源)
應該不會考: 建立Ingress Controller (11)

### SECTION: APPLICATION DESIGN AND BUILD
01. (沒有設置ContainerPort) In the ckad-pod-design namespace, start a ckad-httpd-bwutlljzof pod running the httpd:alpine image.
    The pod's container should be exposed at port 8080.
    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    我沒有設置ContainerPort
    student-node ~ ➜  cat ckad-httpd-bwutlljzof.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
        creationTimestamp: null
        labels:
            run: ckad-httpd-bwutlljzof
        name: ckad-httpd-bwutlljzof
        namespace: ckad-pod-design
    spec:
        containers:
            - image: httpd:alpine
            name: ckad-httpd-bwutlljzof
            resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    Solution:
    Create a YAML file with the content as below:

    apiVersion: v1
    kind: Pod
    metadata:
        creationTimestamp: null
        labels:
            run: ckad-httpd-bwutlljzof
        name: ckad-httpd-bwutlljzof
        namespace: ckad-pod-design
    spec:
        containers:
        - image: httpd:alpine
            name: ckad-httpd-bwutlljzof
            ports:
            - **containerPort: 8080**
            resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

02. (**PV狀態為Released時應刪除重建以恢復到Available，使pvc可以綁定**) A persistent volume called papaya-pv-ckad09-str is already created with a storage capacity of 150Mi. It's using the papaya-stc-ckad09-str storage class with the path /opt/papaya-stc-ckad09-str.

    Also, a persistent volume claim named papaya-pvc-ckad09-str has been created on this cluster. This PVC has requested 50Mi of storage from papaya-pv-ckad09-str volume.

    Resize the PVC to 80Mi and make sure the PVC is in Bound state.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    student-node ~ ➜  kc papaya-pv-ckad09-str.yaml 
    persistentvolume/papaya-pv-ckad09-str created

    student-node ~ ➜  k get pvc
    NAME                    STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS            VOLUMEATTRIBUTESCLASS   AGE
    papaya-pvc-ckad09-str   Lost     papaya-pv-ckad09-str   0                         papaya-stc-ckad09-str   <unset>                 117s

    student-node ~ ➜  k get pvc
    NAME                    STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS            VOLUMEATTRIBUTESCLASS   AGE
    papaya-pvc-ckad09-str   Lost     papaya-pv-ckad09-str   0                         papaya-stc-ckad09-str   <unset>                 119s

    student-node ~ ➜  k get pv
    NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY     STATUS     CLAIM                           STORAGECLASS            VOLUMEATTRIBUTESCLASS   REASON   AGE
    papaya-pv-ckad09-str   150Mi      RWO            Retain           **Released**   default/papaya-pvc-ckad09-str   papaya-stc-ckad09-str   <unset>                          6s

    為何直接修改pvc的resource request, 重啟後pvc 為Lost

    你的 PVC（papaya-pvc-ckad09-str）會進入 "Lost" 狀態，是因為其綁定的 PV（papaya-pv-ckad09-str）已進入 "Released" 狀態。這通常表示 PVC 被刪除或解除綁定後，PV 仍然存在，但 Kubernetes 並未自動回收並重新綁定它。因此，直接修改 PVC 的 resource requests 並不會生效，導致 PVC 變成 "Lost"。

    student-node ~ ➜  k get storageclass
    NAME                    PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
    local-path (default)    rancher.io/local-path          Delete          WaitForFirstConsumer   false                  139m
    papaya-stc-ckad09-str   kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   true                   117m

    正確的解決步驟:
    1️⃣ 刪除舊的 PVC
    PVC 可能仍然持有舊的綁定資訊，所以我們應該先刪除它：

    kubectl delete pvc papaya-pvc-ckad09-str
    2️⃣ 刪除並重新創建 PV
    PV 處於 Released 狀態，需要手動刪除並重新創建：
    kubectl delete pv papaya-pv-ckad09-str
    然後重新創建 PV，確認 PV 的狀態是否為 Available

    3️⃣ 重新創建 PVC，請求 80Mi
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
    name: papaya-pvc-ckad09-str
    spec:
        accessModes:
            - ReadWriteOnce
        resources:
            requests:
            storage: 80Mi
        storageClassName: papaya-stc-ckad09-str

    kubectl apply -f papaya-pvc-ckad09-str.yaml
   
    4️⃣ 檢查 PVC 是否成功綁定
    執行：
    kubectl get pvc
    確認 PVC 狀態為 Bound。

    如果 PVC 仍然是 Pending 或 Lost，請再次檢查：
    kubectl describe pvc papaya-pvc-ckad09-str
    
    看是否有錯誤資訊，例如：
    沒有可用的 PV ➝ 確保 storageClassName 和 storage 大小匹配
    PV 仍然是 Released ➝ 確保 PV 處於 Available 狀態


    Solution:

    Edit papaya-pv-ckad09-str PV:
    kubectl get pv papaya-pv-ckad09-str -o yaml > /tmp/papaya-pv-ckad09-str.yaml

    Edit the template:
    vi /tmp/papaya-pv-ckad09-str.yaml

    Delete all entries for uid:, annotations, status:, claimRef: from the template.

    Edit papaya-pvc-ckad09-str PVC:
    kubectl get pvc papaya-pvc-ckad09-str -o yaml > /tmp/papaya-pvc-ckad09-str.yaml

    Edit the template:
    vi /tmp/papaya-pvc-ckad09-str.yaml

    Under resources: -> requests: change storage: 50Mi to storage: 80Mi and save the template.

    Delete the exsiting PVC:
    kubectl delete pvc papaya-pvc-ckad09-str


    Delete the exsiting PV and create using the template:

    kubectl delete pv papaya-pv-ckad09-str
    kubectl apply -f /tmp/papaya-pv-ckad09-str.yaml

    Create the PVC using template:
    kubectl apply -f /tmp/papaya-pvc-ckad09-str.yaml


### SECTION: APPLICATION DESIGN AND BUILD
04. (答對) Create a persistent volume called cloudstack-pv with the below properties:
    - Its capacity should be 128Mi.
    - The volume type should be hostpath and path should be /opt/cloudstack-pv.
    Next, create a persistent volume claim called cloudstack-pvc as per below properties:
    - Request 50Mi of storage from cloudstack-pv PV.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    student-node ~ ➜  kubectl config use-context cluster2
    Switched to context "cluster2".

    student-node ~ ➜  vim cloudstack-pv.yaml

    student-node ~ ➜  k get storageclas
    error: the server doesn't have a resource type "storageclas"

    student-node ~ ✖ k get storageclass
    NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
    local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  32m

    student-node ~ ➜  vim cloudstack-pv.yaml

    student-node ~ ➜  kc cloudstack-pv.yaml
    Error from server (BadRequest): error when creating "cloudstack-pv.yaml": PersistentVolume in version "v1" cannot be handled as a PersistentVolume: json: cannot unmarshal array into Go struct field PersistentVolumeSpec.spec.hostPath of type v1.HostPathVolumeSource

    student-node ~ ✖ vim cloudstack-pv.yaml

    student-node ~ ➜  kc cloudstack-pv.yaml
    persistentvolume/cloudstack-pv created

    student-node ~ ➜  vim cloustack-pvc.yaml

    student-node ~ ➜  kc cloustack-pvc.yaml
    persistentvolumeclaim/cloudstack-pvc created

    student-node ~ ➜  k get pvc 
    NAME             STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
    cloudstack-pvc   Pending                                      local-path     <unset>                 3s
    (pvc yaml 文件沒有指定storageclass，這邊卻被自動綁定)

    student-node ~ ➜  k get pv
    NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
    cloudstack-pv   128Mi      RWO            Retain           Available                          <unset>                          51s

    student-node ~ ➜  vim cloustack-pvc.yaml
    student-node ~ ➜  cat cloustack-pvc.yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
        name: cloudstack-pvc
    spec:
        **storageClassName: ""** # 強制清空
        accessModes:
            - ReadWriteOnce
        resources:
            requests:
            storage: 50Mi

    student-node ~ ➜  vim cloustack-pvc.yaml

    student-node ~ ➜  kr cloustack-pvc.yaml --force
    persistentvolumeclaim "cloudstack-pvc" deleted
    persistentvolumeclaim/cloudstack-pvc replaced

    student-node ~ ➜  k get pvc
    NAME             STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
    cloudstack-pvc   Bound    cloudstack-pv   128Mi      RWO                           <unset>                 3s


05. (Cronjob schedule錯誤) In the ckad-job namespace, schedule a job called learning-every-hour that prints this message in the shell **every hour at 0 minutes**: I will pass CKAD certification.

    In case the container in pod failed for any reason, it should be restarted automatically.

    Use alpine image for the cronjob!

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    我的配置:
    apiVersion: batch/v1
    kind: CronJob
    metadata:
    creationTimestamp: null
    name: learning-every-hour
    namespace: ckad-job
    spec:
    jobTemplate:
        metadata:
        creationTimestamp: null
        name: learning-every-hour
        spec:
        template:
            metadata:
            creationTimestamp: null
            spec:
            containers:
            - image: alpine
                name: learning-every-hour
                command: ["/bin/sh","-c",echo "I will pass CKAD certification"]
                resources: {}
            restartPolicy: OnFailure
    schedule: ~~'* */1 * * *'~~ # **'* */1 * * *' 這個表達式的第一個 * 代表「分鐘」，而 */1 代表「每 1 小時執行一次」。這樣的配置會導致 CronJob 在每分鐘的 0 秒開始執行一次，而不只是每小時執行一次，這不是你想要的結果。!!**
    status: {}

    Solution
    Create a YAML file with the content as below:

    apiVersion: batch/v1
    kind: CronJob
    metadata:
    namespace: ckad-job
    name: learning-every-hour
    spec:
        schedule: **"0 * * * *"**
        jobTemplate:
            spec:
                template:
                    spec:
                        containers:
                        - name: learning-every-hour
                            image: alpine
                            imagePullPolicy: IfNotPresent
                            command:
                            - /bin/sh
                            - -c
                            - echo I will pass CKAD certification
                restartPolicy: OnFailure



    Then use kubectl apply -f file_name.yaml to create the required object.

06. (答對) An application called results-apd is running on cluster2. In the weekly meeting, the team decides to upgrade the version of the existing image to 1.23.3 and wants to store the new version of the image in a file /root/records/new-image-records.txt on the cluster2-controlplane instance.

    After upgrading the image version, to increase the availability and performance of the application, scale the deployment to 4, and ensure that the targeted pod is running.

    You can SSH into the cluster2 using ssh cluster2-controlplane command.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2


    cluster2-controlplane ~ ✖ k get pod -A
    NAMESPACE       NAME                                      READY   STATUS      RESTARTS   AGE
    kube-system     local-path-provisioner-84db5d44d9-wsl2k   1/1     Running     0          41m
    kube-system     coredns-6799fbcd5-flnsz                   1/1     Running     0          41m
    kube-system     helm-install-traefik-crd-59fds            0/1     Completed   0          41m
    kube-system     metrics-server-67c658944b-ptvjs           1/1     Running     0          41m
    kube-system     svclb-traefik-61dc1183-dr2dt              2/2     Running     0          40m
    kube-system     svclb-traefik-61dc1183-mbqc6              2/2     Running     0          40m
    kube-system     helm-install-traefik-b8hhv                0/1     Completed   2          41m
    kube-system     traefik-f4564c4f4-nckdh                   1/1     Running     0          40m
    dashboard-apd   results-apd-5d7744765d-kbwdw              1/1     Running     0          94s
    ckad-job        learning-every-hour-29011950-q2jvt        0/1     Completed   0          77s
    ckad-job        learning-every-hour-29011951-7j7bj        0/1     Completed   0          17s

    cluster2-controlplane ~ ➜  k get all -A| grep results-apd
    dashboard-apd   pod/results-apd-5d7744765d-kbwdw              1/1     Running     0          2m8s
    dashboard-apd   deployment.apps/results-apd              1/1     1            1           2m8s
    dashboard-apd   replicaset.apps/results-apd-5d7744765d              1         1         1       2m8s

    cluster2-controlplane ~ ➜  k get pod -n dashboard-apd 
    NAME                           READY   STATUS    RESTARTS   AGE
    results-apd-5d7744765d-kbwdw   1/1     Running   0          2m28s

    cluster2-controlplane ~ ➜  k get pod -n dashboard-apd -o yaml > dashboard-apd.yaml

    cluster2-controlplane ~ ➜  vim dashboard-apd.yaml 

    cluster2-controlplane ~ ✖ k replace -f dashboard-apd.yaml --force
    pod "results-apd-5d7744765d-kbwdw" deleted
    pod/results-apd-5d7744765d-kbwdw replaced

    cluster2-controlplane ~ ➜  k get pod -n dashboard-apd 
    NAME                           READY   STATUS    RESTARTS   AGE
    results-apd-5d7744765d-48zfv   1/1     Running   0          31s



07. (helm upgrade --set replicaCount) One co-worker deployed an nginx helm chart on the cluster3 server called lvm-crystal-apd. A new update is pushed to the helm chart, and the team wants you to update the helm repository to fetch the new changes.
    
    After updating the helm chart, upgrade the helm chart version to 18.1.15 and increase the replica count to 2.

    NOTE: - We have to perform this task on the cluster3-controlplane node.

    You can SSH into the cluster3 using ssh cluster3-controlplane command.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  helm list
    NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION

    student-node ~ ➜  helm list -A
    NAME            NAMESPACE       REVISION        UPDATED                                 STATUS      CHART            APP VERSION
    lvm-crystal-apd crystal-apd-ns  1               2025-02-28 04:37:23.46649879 +0000 UTC  deployed    nginx-18.1.0     1.27.0     

    
    Solution:
    Run the following command to change the context: -
    kubectl config use-context cluster3

    In this task, we will use the kubectl and helm commands. Here are the steps: -
    Log in to the cluster3-controlplane node first and use the helm ls command to list all the releases installed using Helm in the Kubernetes cluster.

    helm ls -A
    Here -A or --all-namespaces option lists all the releases of all the namespaces.

    Identify the namespace where the resources get deployed.
    Use the helm repo ls command to list the helm repositories.
    **helm repo ls** 

    Now, update the helm repository with the following command: -
    **helm repo update lvm-crystal-apd -n crystal-apd-ns**

    The above command updates the local cache of available charts from the configured chart repositories.

    The helm search command searches for all the available charts in a specific Helm chart repository. In our case, it's the nginx helm chart.
    **helm search repo lvm-crystal-apd/nginx -n crystal-apd-ns -l | head -n30**
    The -l or --versions option is used to display information about all available chart versions.

    Upgrade the helm chart to 18.1.15 and also, increase the replica count of the deployment to 2 from the command line. Use the helm upgrade command as follows: -
    **helm upgrade lvm-crystal-apd lvm-crystal-apd/nginx -n crystal-apd-ns --version=18.1.15 --set replicaCount=2**

    After upgrading the chart version, you can verify it with the following command: -
    helm ls -n crystal-apd-ns

    Look under the CHART column for the chart version.
    Use the kubectl get command to check the replicas of the deployment: -
    kubectl get deploy -n crystal-apd-ns

    The available count 2 is under the AVAILABLE column.



### SECTION: APPLICATION DEPLOYMENT

08. (helm **Uninstall之後可能有殘留的資源!! 善用k delete all -l <label> or helm uninstall --no-hooks**) One application, webpage-server-01, is deployed on the Kubernetes cluster by the Helm tool. Now, the team wants to deploy a new version of the application by replacing the existing one. A new version of the helm chart is given in the /root/new-version directory on the student-node. Validate the chart before installing it on the Kubernetes cluster. 

    Use the helm command to validate and install the chart. After successfully installing the newer version, uninstall the older version. 

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    student-node ~ ➜  helm lint /root/new-version/
    Chart.yaml   templates/   values.yaml  

    student-node ~ ➜  helm lint /root/new-version/Chart.yaml 
    ==> Linting /root/new-version/Chart.yaml
    Error unable to check Chart.yaml file in chart: stat /root/new-version/Chart.yaml/Chart.yaml: not a directory

    Error: 1 chart(s) linted, 1 chart(s) failed

    student-node ~ ✖ helm lint /root/new-version/Chart.yaml ^C^C

    student-node ~ ✖ cp /root/new-version/Chart.yaml /root/new-version/Chart.yaml.bk

    student-node ~ ➜  vim /root/new-version/Chart.yaml

    student-node ~ ➜  helm lint /root/new-version/
    ==> Linting /root/new-version/
    [INFO] Chart.yaml: icon is recommended

    1 chart(s) linted, 0 chart(s) failed

    student-node ~ ✖ helm install webpage-server-01 /root/new-version/Chart.yaml
    Error: INSTALLATION FAILED: file '/root/new-version/Chart.yaml' seems to be a YAML file, but expected a gzipped archive

    student-node ~ ✖ helm install webpage-server-01 /root/new-version/
    Error: INSTALLATION FAILED: cannot re-use a name that is still in use

    **可以使用--generate-name 讓helm 自動幫取一個新的release name:**
    **helm install --generate-name ./new-version**

    student-node ~ ✖ helm install webpage-server-01-new /root/new-version/
    NAME: webpage-server-01-new
    LAST DEPLOYED: Fri Feb 28 06:00:08 2025
    NAMESPACE: default
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None

    student-node ~ ➜  helm repo list
    Error: no repositories to show

    student-node ~ ✖ helm list
    NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS       CHART                   APP VERSION
    webpage-server-01       default         1               2025-02-28 04:38:37.962769763 +0000 UTC deployed     webpage-server-01-0.1.0 v1         
    webpage-server-01-new   default         1               2025-02-28 06:00:08.240527456 +0000 UTC deployed     webpage-server-02-0.1.1 v2         

    student-node ~ ➜  helm uninstall webpage-server-01
    release "webpage-server-01" uninstalled # 被判斷為沒有成功刪除
    
    **即使 helm uninstall 成功，Kubernetes 可能還殘留相關的 Pod、Service、ConfigMap 等資源。你可以手動確認**：
    kubectl get all -n default | grep webpage-server-01
    
    如果仍有資源，你可能需要手動刪除：
    **kubectl delete all -l app=webpage-server-01** -n default

    **也有可能helm uninstall 操作實際上沒有執行**
    
    你可以檢查 Helm 的歷史記錄：
    **helm history webpage-server-01 -n default**
    
    如果仍然有紀錄，說明 helm uninstall 沒有正確執行。你可以**強制刪除**：
    helm uninstall webpage-server-01 -n default **--no-hooks**


    
    Solution:

    Run the following command to change the context: -
    kubectl config use-context cluster1

    In this task, we will use the helm commands. Here are the steps: -
    Use the helm ls command to list the release deployed on the default namespace using helm.
    helm ls -n default

    First, validate the helm chart by using the helm lint command: -
    cd /root/
    **helm lint ./new-version** # 這步驟有做

    Now, install the new version of the application by using the helm install command as follows: -
    **helm install --generate-name ./new-version**

    We haven't got any release name in the task, so we can **generate the random name from the --generate-name option**.

    Finally, uninstall the old version of the application by using the helm uninstall command: -
    helm uninstall webpage-server-01 -n default # 這步驟有做

    Details
    O Is the new version app deployed? 
    X Is the old version app uninstalled?




09. (沒有在指定的節點上操作!!!) In the dev-apd namespace, one of the developers has performed a rolling update and upgraded the application to a newer version. But somehow, application pods are not being created.
    To regain the working state, rollback the application to the previous version.

    After rolling the deployment back, ****on the controlplane node****, **save the image currently in use to the /root/records/**rolling-back-record.txt file and increase the replica count to 3.

    You can SSH into the cluster2 using ssh cluster2-controlplane command.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2


    Details: 
    O Is rolling back successful?
    X Is the Image saved to a file?
    O Is the deployment scaled?



    student-node ~ ✖ k get deploy -A
    NAMESPACE       NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
    kube-system     local-path-provisioner   1/1     1            1           53m
    kube-system     coredns                  1/1     1            1           53m
    kube-system     metrics-server           1/1     1            1           53m
    kube-system     traefik                  1/1     1            1           52m
    dashboard-apd   results-apd              4/4     4            4           13m
    app-lox12387    deluxe-apd               1/1     1            1           3m16s
    newspace        deluxe-apd               1/1     1            1           17s

    student-node ~ ➜  k delete deploy -n app-lox12387 deluxe-apd --force
    Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
    deployment.apps "deluxe-apd" force deleted

    student-node ~ ➜  kubectl config use-context cluster2
    Switched to context "cluster2".

    student-node ~ ➜  k get deploy -n dev-apd 
    NAME         READY   UP-TO-DATE   AVAILABLE   AGE
    webapp-apd   2/2     1            2           27s

    student-node ~ ➜  k rollout history -n dev-apd webapp-apd
    error: the server doesn't have a resource type "webapp-apd"

    student-node ~ ✖ k rollout history deploy -n dev-apd webapp-apd
    deployment.apps/webapp-apd 
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>


    student-node ~ ➜  k rollout undo deployment -n dev-apd webapp-apd 
    deployment.apps/webapp-apd rolled back

    student-node ~ ➜  k rollout history deploy -n dev-apd webapp-apd
    deployment.apps/webapp-apd 
    REVISION  CHANGE-CAUSE
    2         <none>
    3         <none>


    student-node ~ ➜  k get deploy -n dev-apd webapp-apd |grep Image

    student-node ~ ✖ k get deploy -n dev-apd webapp-apd
    NAME         READY   UP-TO-DATE   AVAILABLE   AGE
    webapp-apd   2/2     2            2           107s

    student-node ~ ➜  k describe deploy -n dev-apd webapp-apd |grep Image
        Image:         kodekloud/webapp-color

    student-node ~ ➜  k describe deploy -n dev-apd webapp-apd |grep Image > /root/records/rolling-back-record.txt
    -su: /root/records/rolling-back-record.txt: No such file or directory

    student-node ~ ✖ mkdir /root/records

    student-node ~ ➜  touch /root/records/rolling-back-record.txt

    student-node ~ ➜  k describe deploy -n dev-apd webapp-apd |grep Image > /root/records/rolling-back-record.txt

    student-node ~ ➜  cat /root/records/rolling-back-record.txt
        Image:         kodekloud/webapp-color

    題目要求：

    ssh cluster2-controlplane
    echo "kodekloud/webapp-color" > /root/records/rolling-back-record.txt
    但你的解法中，kubectl describe 是直接在 學生節點 (student-node) 上執行的，而非在 cluster2-controlplane 節點上執行。

    解決方案
    如果還沒執行這一步，請進入 cluster2-controlplane 並重新執行：
    ssh cluster2-controlplane
    echo "kodekloud/webapp-color" > /root/records/rolling-back-record.txt
    然後再檢查：
    ssh cluster2-controlplane
    cat /root/records/rolling-back-record.txt
    確認內容是否正確。




    student-node ~ ➜  k edit deploy -n dev-apd webapp-apd 
    deployment.apps/webapp-apd edited

    student-node ~ ➜  k get deploy -n dev-apd webapp-apd
    NAME          READY   UP-TO-DATE   AVAILABLE   AGE
    webapp-apd   3/3     3            3           4m34s

    
    Solution:

    Run the following command to change the context: -
    kubectl config use-context cluster2


    In this task, we will use the kubectl describe, kubectl get, kubectl rollout and kubectl scale commands. Here are the steps: -

    First check the status of the pods: -
    kubectl get pods -n dev-apd

    One of the pods is in an error state. By using the kubectl describe command. We can see that there is an issue with the image.
    We can check the revision history of a deployment by using the kubectl history command as follows: -
    kubectl rollout history -n dev-apd deploy webapp-apd  


    Inspect the revision in detail as follows: -
    kubectl rollout history -n dev-apd deploy webapp-apd --revision=2

    We found that the image issue happened because of the wrong image tag, and the previous image was correct. As a quick fix, we need to roll back to the previous revision. Use the kubectl rollout command: -
    kubectl rollout undo -n dev-apd deploy webapp-apd

    After successful rolling back, inspect the updated image: -
    kubectl describe deploy -n dev-apd webapp-apd | grep -i image

    On the Controlplane node, save the image name to the given path /root/records/rolling-back-record.txt: -
    ssh cluster2-controlplane
    echo "kodekloud/webapp-color" > /root/records/rolling-back-record.txt


    If the records directory is absent, use the mkdir command to create this directory.
    NOTE: - To exit from any node, type exit on the terminal or press CTRL + D.

    And increase the replica count to the 3 with help of kubectl scale command: -
    kubectl scale deploy -n dev-apd webapp-apd --replicas=3

    Verify it by running the command: kubectl get deploy -n dev-apd


### SECTION: SERVICES AND NETWORKING

11. (配置文件debug得太慢!) For this scenario, create an ingress controller.
    We have already deployed some of the required resources (Namespaces, Service accounts, Roles and Rolebindings).

    Your task is to create the Ingress controller Deployment using the manifest given at /root/nginx-controller.yaml. There are some issues in the configuration. Please find the issues and fix them.

    Note: All the resources are deployed in ingress-nginx namespace.

    Is the deployme

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    student-node ~ ✖ cp /root/nginx-controller.yaml nginx-controller.yaml.bk


    student-node ~ ➜  ka nginx-controller.yaml
    error: resource mapping not found for name: "ingress-nginx-controller" namespace: "ingress-nginx" from "nginx-controller.yaml": no matches for kind "deployment" in version "apps/betav1"
    ensure CRDs are installed first

    student-node ~ ✖ k api-resources |grep ingress
    ingressclasses                                 networking.k8s.io/v1              false        IngressClass
    ingresses                         ing          networking.k8s.io/v1              true         Ingress
    ingressroutes                                  traefik.containo.us/v1alpha1      true         IngressRoute
    ingressroutetcps                               traefik.containo.us/v1alpha1      true         IngressRouteTCP
    ingressrouteudps                               traefik.containo.us/v1alpha1      true         IngressRouteUDP
    ingressroutes                                  traefik.io/v1alpha1               true         IngressRoute
    ingressroutetcps                               traefik.io/v1alpha1               true         IngressRouteTCP
    ingressrouteudps                               traefik.io/v1alpha1               true         IngressRouteUDP

    student-node ~ ➜  k api-resources |grep deploy
    deployments                       deploy       apps/v1                           true         Deployment

    student-node ~ ➜  vim nginx-controller.yaml

    student-node ~ ➜  ka nginx-controller.yaml
    Error from server (BadRequest): error when creating "nginx-controller.yaml": deployment in version "v1" cannot be handled as a Deployment: no kind "deployment" is registered for version "apps/v1" in scheme "k8s.io/apimachinery@v1.29.0-k3s1/pkg/runtime/scheme.go:100"

    student-node ~ ✖ vim nginx-controller.yaml

    student-node ~ ➜  vim nginx-controller.yaml

    student-node ~ ➜  ka nginx-controller.yaml
    Error from server (BadRequest): error when creating "nginx-controller.yaml": deployment in version "v1" cannot be handled as a Deployment: no kind "deployment" is registered for version "apps/v1" in scheme "k8s.io/apimachinery@v1.29.0-k3s1/pkg/runtime/scheme.go:100"

    student-node ~ ➜  vim nginx-controller.yaml

    student-node ~ ➜  ka nginx-controller.yaml
    Error from server (BadRequest): error when creating "nginx-controller.yaml": deployment in version "v1" cannot be handled as a Deployment: **no kind "deployment"** is registered for version "apps/v1" in scheme "k8s.io/apimachinery@v1.29.0-k3s1/pkg/runtime/scheme.go:100"

    **應為大寫: Deployment**

    Solution:

    Use cat command to view the contents of the file. cat /root/nginx-controller.yaml

    Please check the apiVersion, resource kind, namespace and the container port sections
    apiVersion: apps/betav1 #wrong api version
    kind: deployment #change this
    metadata:
    labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
        app.kubernetes.io/version: 1.6.4
    name: ingress-nginx-controller
    namespace: ingressnginx  #issue1 ingress-nginx
    spec:
        minReadySeconds: 0
        revisionHistoryLimit: 10
        selector:
            matchLabels:
            app.kubernetes.io/component: controller
            app.kubernetes.io/instance: ingress-nginx
            app.kubernetes.io/name: ingress-nginx
        template:
            metadata:
            labels:
                app.kubernetes.io/component: controller
                app.kubernetes.io/instance: ingress-nginx
                app.kubernetes.io/name: ingress-nginx
            spec:
            containers:
            - args:
                - /nginx-ingress-controller
                - --publish-service=/ingress-nginx-controller
                - --election-id=ingress-nginx-leader
                - --controller-class=k8s.io/ingress-nginx
                - --ingress-class=nginx
                - --configmap=/ingress-nginx-controller
                - --validating-webhook=:8443
                - --validating-webhook-certificate=/usr/local/certificates/cert
                - --validating-webhook-key=/usr/local/certificates/key
                env:
                - name: POD_NAME
                valueFrom:
                    fieldRef:
                    fieldPath: metadata.name
                - name: POD_NAMESPACE
                valueFrom:
                    fieldRef:
                    fieldPath: metadata.namespace
                - name: LD_PRELOAD
                value: /usr/local/lib/libmimalloc.so
                image: registry.k8s.io/ingress-nginx/controller:v1.6.4@sha256:15be4666c53052484dd2992efacf2f50ea77a78ae8aa21ccd91af6baaa7ea22f
                imagePullPolicy: IfNotPresent
                lifecycle:
                preStop:
                    exec:
                    command:
                    - /wait-shutdown
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
                name: controller
                ports:
                -    containerPort: 80 #wrong indentation 
                name: http
                protocol: TCP
                - containerPort: 443
                name: https
                protocol: TCP
                - containerPort: 8443
                name: webhook
                protocol: TCP
                readinessProbe:
                failureThreshold: 3
                httpGet:
                    path: /healthz
                    port: 10254
                    scheme: HTTP
                initialDelaySeconds: 10
                periodSeconds: 10
                successThreshold: 1
                timeoutSeconds: 1
                resources:
                requests:
                    cpu: 100m
                    memory: 90Mi
                securityContext:
                allowPrivilegeEscalation: true
                capabilities:
                    add:
                    - NET_BIND_SERVICE
                    drop:
                    - ALL
                runAsUser: 101
                volumeMounts:
                - mountPath: /usr/local/certificates/
                name: webhook-cert
                readOnly: true
            dnsPolicy: ClusterFirst
            nodeSelector:
                kubernetes.io/os: linux
            serviceAccountName: ingress-nginx
            terminationGracePeriodSeconds: 300
            volumes:
            - name: webhook-cert
                secret:
                secretName: ingress-nginx-admission




12. (沒有建立與Pod Label 相同的Service Selector!) Create a Kubernetes Service named ckad14-svcn that routes traffic to the external domain my.application.test.com. This service should be configured as type ExternalName.

    Create the service in the default namespace.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3


    student-node ~ ➜  k get svc
    NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
    app-ckad-svcn   ClusterIP   172.20.37.29   <none>        80/TCP,53/TCP   2s
    kubernetes      ClusterIP   172.20.0.1     <none>        443/TCP         68m
    

    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  k create svc externalname ckad14-svcn --external-name=my.application.test.com
    service/ckad14-svcn created

    student-node ~ ➜  k get svc app-ckad-svcn -o yaml
    apiVersion: v1
    kind: Service
    metadata:
        creationTimestamp: "2025-02-28T04:58:25Z"
        labels:
            app: app-ckad
            scenario: multiport # (不是只有這裡需要Labels!!)
        name: app-ckad-svcn
        namespace: default
        resourceVersion: "6468"
        uid: 1e1134b0-4af0-4f61-a903-0ad2874424b7
    spec:
        # **沒有設置Selector!!!!  需要依照pod 的Labels來建立Service 的Labels!!**
        clusterIP: 172.20.37.29
        clusterIPs:
        - 172.20.37.29
        internalTrafficPolicy: Cluster
        ipFamilies:
        - IPv4
        ipFamilyPolicy: SingleStack
        ports:
        - name: http
            port: 80
            protocol: TCP
            targetPort: 80
        - name: dns
            port: 53
            protocol: TCP
            targetPort: 53
        selector:
            app: app-ckad
            scenario: multiport
        sessionAffinity: None
        type: ClusterIP
        status:
        loadBalancer: {}

    student-node ~ ➜  k get svc
    NAME            TYPE           CLUSTER-IP     EXTERNAL-IP               PORT(S)         AGE
    app-ckad-svcn   ClusterIP      172.20.37.29   <none>                    80/TCP,53/TCP   61s
    ckad14-svcn     ExternalName   <none>         my.application.test.com   <none>          3s
    kubernetes      ClusterIP      172.20.0.1     <none>                    443/TCP         69m

    
    Solution
    The pod app-ckad-svcn is deployed in default namespace.

    To view the pod along with labels, use the following command.
    student-node ~ ➜  kubectl get pods app-ckad-svcn --show-labels 
    NAME            READY   STATUS    RESTARTS   AGE     LABELS
    app-ckad-svcn   1/1     Running   0          2m58s   app=app-ckad,scenario=multiport

    We will use those labels to create the service.

    Create a service using the following manifest. It will create a service with multiple ports expose with different protocols.
    apiVersion: v1
    kind: Service
    metadata:
    name: multi-port-svcn
    labels:
        app: app-ckad
        scenario: multiport
    spec:
        **selector:**
            app: app-ckad
            **scenario: multiport**
        ports:
        - port: 80
            targetPort: 80
            protocol: TCP
            name: http
        - port: 53
            targetPort: 53
            protocol: UDP
            name: dns



13. (答對) A new payment service has been introduced. Since it is a sensitive application, it is deployed in its own namespace sensitive-space. Inspect the resources and service created.

    You are requested to make the new application available at /pay. Create an ingress resource named ingress-ckad14-pay for the payment application to make it available at /pay

    Identify and implement the best approach to making this application available on the ingress controller and test to make sure its working. Look into annotations: rewrite-target as well.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    student-node ~ ➜  kubectl config use-context cluster2
    Switched to context "cluster2".

    student-node ~ ➜  k get svc -n sensitive-space 
    NAME               TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
    **pay-service**   ClusterIP   10.43.196.5   <none>       **8282/TCP**   80s

    student-node ~ ➜  k get ingress -A
    No resources found

    student-node ~ ➜  k get deploy -n sensitive-space 
    NAME          READY   UP-TO-DATE   AVAILABLE   AGE
    payment-pod   1/1     1            1           105s

    student-node ~ ➜  k get deploy -n ingress-nginx 
    NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
    ingress-nginx-controller   1/1     1            1           118s

    student-node ~ ➜  vim ingress-ckad14-pay.yaml
    student-node ~ ✖ cat ingress-ckad14-pay.yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: ingress-ckad14-pay
    namespace: sensitive-space
    annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
    ingressClassName: nginx-example
    rules:
    - http:
        paths:
        - path: /pay
            pathType: Prefix
            backend:
            service:
                name: pay-service
                port:
                number: 8282

    student-node ~ ➜  kc ingress-ckad14-pay.yaml
    ingress.networking.k8s.io/ingress-ckad14-pay created

    student-node ~ ➜  k get ingress -n sensitive-space 
    NAME                 CLASS           HOSTS   ADDRESS   PORTS   AGE
    ingress-ckad14-pay   nginx-example   *                 80      14s

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY
15. (答對) We have already deployed the required pods and services in the namespace ckad01-appstk-sec-aecs.

    Create a new secret named ckad01-db-scrt-aecs with the data given below.
    Secret Name: ckad01-db-scrt-aecs
    Secret 1: DB_Host=sql01
    Secret 2: DB_User=root
    Secret 3: DB_Password=password12

    Configure ckad01-webapp-pod-aecs to load environment variables from the newly created secret, where the keys from the secret should become the environment variable name in the Pod.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    student-node ~ ✖ vim ckad01-db-scrt-aecs.yaml
    student-node ~ ➜  cat ckad01-db-scrt-aecs.yaml
    apiVersion: v1
    kind: Secret
    metadata:
    name: ckad01-db-scrt-aecs
    namespace: ckad01-appstk-sec-aecs
    annotations:
        kubernetes.io/service-account.name: "sa-name"
    type: Opaque
    stringData:
    DB_Host: sql01
    DB_User: root
    DB_Password: password123

    student-node ~ ➜  kc ckad01-db-scrt-aecs.yaml
    secret/ckad01-db-scrt-aecs created

    student-node ~ ➜  k get secrets -n ckad01-appstk-sec-aecs
    NAME                  TYPE     DATA   AGE
    ckad01-db-scrt-aecs   Opaque   3      21s

    student-node ~ ➜  k get pod -n ckad01-appstk-sec-aecs 
    NAME                     READY   STATUS    RESTARTS   AGE
    ckad01-webapp-pod-aecs   1/1     Running   0          4m10s
    ckad01-db-pod-aecs       1/1     Running   0          4m9s

    student-node ~ ➜  k get svc -n ckad01-appstk-sec-aecs 
    NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    ckad01-webapp-service-aecs   NodePort    10.43.59.60     <none>        8080:30080/TCP   4m22s
    ckad01-db-svc-aecs           ClusterIP   10.43.140.129   <none>        3306/TCP         4m21s

    student-node ~ ➜  vim ckad01-webapp-pod-aecs.yaml 
    student-node ~ ➜  cat ckad01-webapp-pod-aecs.yaml 
    ...
    spec:
    containers:
    - image: kodekloud/simple-webapp-mysql
        imagePullPolicy: Always
        name: webapp
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        env:
        - name: DB_Host # Notice that the case is different here
            valueFrom:
            secretKeyRef:
                name: ckad01-db-scrt-aecs    # The ConfigMap this value comes from.
                key: DB_Host # The key to fetch.
        - name: DB_User # Notice that the case is different here
            valueFrom:
            secretKeyRef:
                name: ckad01-db-scrt-aecs    # The ConfigMap this value comes from.
                key: DB_User
        - name: DB_Password # Notice that the case is different here
            valueFrom:
            secretKeyRef:
                name: ckad01-db-scrt-aecs    # The ConfigMap this value comes from.
                key: DB_Password
    ...

    student-node ~ ➜  kr ckad01-webapp-pod-aecs.yaml --force
    pod "ckad01-webapp-pod-aecs" deleted
    pod/ckad01-webapp-pod-aecs replaced

    student-node ~ ➜  k get pod -n ckad01-appstk-sec-aecs 
    NAME                     READY   STATUS    RESTARTS   AGE
    ckad01-db-pod-aecs       1/1     Running   0          12m
    ckad01-webapp-pod-aecs   1/1     Running   0          7s


17. (RoleBinding參數有遺漏) Create a role named pod-reader in the ckad17-auth-ns namespace, and grant only the list, watch and get permissions on pods resources.

    Create a role binding named read-pods in the same namespace, and assign the pod-reader role to a user named jane.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    Details: 
    O Is correct resource for role "pod-reader" specified
    O Are correct "verbs" specified for the role?
    O Is correct role used for rolebinding?
    X Is correct user specified for rolebinding?

    student-node ~ ✖ kubectl auth can-i create pod --as jane --namespace ckad17-auth-ns
    no > 這應該得yes
    
    我的name沒有設置到:
    student-node ~ ➜  k describe rolebinding -n ckad17-auth-ns read-pods 
    Name:         read-pods
    Labels:       <none>
    Annotations:  <none>
    Role:
    Kind:  Role
    Name:  pod-reader
    Subjects:
    Kind  Name  Namespace
    /----  ----  ---------


    應為: 
    Name:         read-pods-2
    Labels:       <none>
    Annotations:  <none>
    Role:
    Kind:  Role
    Name:  pod-reader
    Subjects:
    Kind  Name  Namespace
   / ----  ----  ---------
    User  **jane**  

    (設置rolebinding建議使用文件以避免遺漏參數)

### SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE
20. (答對) A pod named backend-pod is deployed and exposed with a service service-backend, but it seems the service is not configured properly and is not selecting the correct pod.

    Make the required changes to service and ensure the endpoint is configured for service.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    student-node ~ ✖ k get pod 
    NAME                                 READY   STATUS       RESTARTS   AGE
    my-simple-job-b7l9b                  0/1     StartError   0          68m
    my-simple-job-zjks5                  0/1     StartError   0          67m
    my-simple-job-qxkdw                  0/1     StartError   0          67m
    my-simple-job-b687p                  0/1     StartError   0          66m
    my-simple-job-nhvqf                  0/1     StartError   0          65m
    my-simple-job-9rgv7                  0/1     StartError   0          62m
    my-simple-job-spwdh                  0/1     StartError   0          57m
    webpage-server-01-5d57d97f8c-l7x7p   1/1     Running      0          48m
    webpage-server-01-5d57d97f8c-srwsg   1/1     Running      0          48m
    probe-ckad-aom                       1/1     Running      0          106s
    resou-limit-aom                      1/1     Running      0          23s
    backend-pod                          1/1     Running      0          13s

    student-node ~ ➜  k get svc
    NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    kubernetes              ClusterIP   10.43.0.1       <none>        443/TCP    97m
    webpage-server-01-svc   ClusterIP   10.43.237.113   <none>        8080/TCP   48m
    service-backend         ClusterIP   10.43.186.104   <none>        80/TCP     19s


    student-node ~ ➜  k describe pod backend-pod |grep IP:
    IP:               10.42.1.12
    IP:  10.42.1.12


    student-node ~ ✖ vim backend-endpoint.yaml
    student-node ~ ✖ cat backend-endpoint.yaml
    apiVersion: v1
    kind: Endpoints
    metadata:
    name: endpoint-backend
    subsets:
    - addresses:
        - ip: 10.42.1.12


    student-node ~ ➜  kc backend-endpoint.yaml
    endpoints/endpoint-backend created

    student-node ~ ✖ k describe svc service-backend
    Name:              service-backend
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          app=ngnix
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                10.43.186.104
    IPs:               10.43.186.104
    Port:              80-80  80/TCP
    TargetPort:        80/TCP
    Endpoints:         <none>
    Session Affinity:  None
    Events:            <none>


    student-node ~ ➜  k describe pod backend-pod |grep Labels
    Labels:           app=nginx

    student-node ~ ➜  k describe svc service-backend |grep Selector:
    Selector:          app=ngnix

    student-node ~ ➜  k edit svc service-backend 
    service/service-backend edited

    student-node ~ ➜  k describe svc service-backend
    Name:              service-backend
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          app=nginx
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                10.43.186.104
    IPs:               10.43.186.104
    Port:              80-80  80/TCP
    TargetPort:        80/TCP
    Endpoints:         10.42.1.12:80
    Session Affinity:  None
    Events:            <none>

