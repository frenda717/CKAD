Kodekloud Redo List

jsonpath: [*]
kubectl get deploy -n atlanta-page-04 -o=jsonpath="{.items[*].spec.template.spec.containers[*].image}"

jq: []
kubectl get deploy <DEPLOYMENT-NAME> -o json | jq -r '.spec.template.spec.**containers[]**.image'

custom-column: []
k get pod hulk -o custom-columns=Podname:metadata.name,Memorylimit:spec.**containers[]**.resources.limits.memory > pod-metrics



### Mock 01: 
錯題: 01,02,05,11,15,16,19,20
難題: 07,10,13,14,17

7. On the student-node, a Helm chart repository is given under the /opt/ path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. The files have some issues. Fix those issues and deploy them with the following specifications: - (難)

    The release name should be webapp-color-apd.
    All the resources should be deployed on the frontend-apd namespace.
    The service type should be node port.
    Scale the deployment to 3.
    Application version should be 1.20.0.
    NOTE: - Remember to make necessary changes in the values.yaml and Chart.yaml files according to the specifications, and, to fix the issues, inspect the template files.

    You can start new terminal to open student-node command.

    student-node ~ ➜  ls /opt/webapp-color-apd/
    Chart.yaml  templates  values.yaml

    student-node ~ ➜  cd /opt/webapp-color-apd/

    student-node /opt/webapp-color-apd is 📦 v0.1.0 via ⎈ v3.15.1 ✖ cat Chart.yaml 
    apiVersion: v2
    name: webapp-color-apd
    namespace: frontend-apd
    description: A Helm chart for Webapp Color Application
    type: application
    version: 0.1.0
    appVersion: **1.20.0**   # v1           

    student-node /opt/webapp-color-apd is 📦 v0.1.0 via ⎈ v3.15.1 ✖ helm install webapp-color-apd ./
    Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: resource mapping not found for name: "webapp-color-apd" namespace: "" from "": no matches for kind "Deployment" in version "v1"
    ensure CRDs are installed first


    (**沒有進入到templates資料夾裡面排查!**)

    student-node /opt/webapp-color-apd is 📦 v0.1.0 via ⎈ v3.15.1 ➜  ls templates/
    configmap.yaml  deployment.yaml  serviceaccount.yaml  service.yaml

    l udent-node /opt/webapp-color-apd is 📦 v0.1.0 via ⎈ v3.15.1 ➜  cat templates/deployment.yaml
    apiVersion:   apps/v1  #  ~~v1~~
    kind: Deployment
    metadata:
    labels:
        app: webapp-color-apd
    name: {{ .Values.name }}
    spec:
    replicas: {{ .Values.replicaCount }}
    selector:
        matchLabels:
        app: webapp-color-apd
    template:
        metadata:
        labels:
            app: webapp-color-apd
        spec:
        containers:
        - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
            name: webapp-color-apd
            envFrom:
            - configMapRef:
                    name: {{ .Values.configMap.name }}

    student-node /opt/webapp-color-apd is 📦 v0.1.0 via ⎈ v3.15.1 ➜  helm install webapp-color-apd ./
    Error: INSTALLATION FAILED: Unable to continue with install: could not get information about the resource Service "" in namespace "default": resource name may not be empty

    student-node /opt/webapp-color-apd is 📦 v0.1.0 via ⎈ v3.15.1 ➜  cat values.yaml |grep -A5 service:
    service:
    name: webapp-color-svc
    type: NodePort
    port: 8080

    apiVersion: v1
    kind: Service
    metadata:
        name: {{ .values.service.name }} # name: {{ .Values.service.Name }}
    spec:
        ports:
        - port: 8080
            protocol: TCP
            targetPort: 8080
        selector:
            app: webapp-color-apd
        type: {{ .values.service.type }} # type: {{ .Values.service.type }}

    都已修改後:
    student-node /opt/webapp-color-apd is 📦 v0.1.0 via ⎈ v3.15.1 ➜  helm install webapp-color-apd . --set replicaCount=3 -n frontend-apd -n frontend-apd

    W0314 00:31:30.622979    6781 warnings.go:70] unknown field "metadata.namespaece"
    NAME: webapp-color-apd
    LAST DEPLOYED: Fri Mar 14 00:31:30 2025
    NAMESPACE: frontend-apd
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None


    student-node /opt/webapp-color-apd is 📦 v0.1.0 via ⎈ v3.15.1 ➜  helm list -n frontend-apd
    NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
    webapp-color-apd        frontend-apd    1               2025-03-14 00:31:30.112239853 +0000 UTC deployed        webapp-color-apd-0.1.0  1.20.0  
        

10.   We have an external webserver running on student-node which is exposed at port 9999. (難)

    We have also created a service called external-webserver-ckad01-svcn that can connect to our local webserver from within the cluster3 but, at the moment, it is not working as expected.

    Fix the issue so that other pods within cluster3 can use external-webserver-ckad01-svcn service to access the webserver.

    我的配置:

    以為service label沒有對應到pod22, 所以主動添加label -> 錯!!

    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  k describe svc external-webserver-ckad01-svcn 
    Name:              external-webserver-ckad01-svcn
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          run=pod22-ckad-svcn
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                172.20.125.208
    IPs:               172.20.125.208
    Port:              <unset>  80/TCP
    TargetPort:        9999/TCP
    Endpoints:         172.17.1.6:9999
    Session Affinity:  None
    Events:            <none>

    Solution
    Let's check if the webserver is working or not:


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
    Endpoints:         <none> # there are no endpoints for the service
    ...

    
    As we can see there is no endpoints specified for the service, hence we won't be able to get any output. Since we can not destroy any k8s object, let's create the endpoint manually for this service as shown below:

    student-node ~ ➜  export IP_ADDR=$(ifconfig eth0 | grep inet | awk '{print $2}')

    student-node ~ ➜ kubectl --context cluster3 apply -f - <<EOF
    apiVersion: v1
    kind: Endpoints
    metadata:
    /# the name here should match the name of the Service
    name: external-webserver-ckad01-svcn
    subsets:
    - addresses:
        - ip: $IP_ADDR
        ports:
        - port: 9999
    EOF


    Finally check if the curl test works now: (透過 kubectl run 創建一個測試 Pod，並使用 curl 來檢查是否能連接到 Service)
    student-node ~ ➜  kubectl **--context cluster3** run --rm  -i test-curl-pod --image=curlimages/curl --restart=Never -- curl -m 2 external-webserver-ckad01-svcn
    ...
    <title>Welcome to nginx!</title>
    ...


    **Web 伺服器是外部的 (student-node:9999)**

    這個 Web 伺服器**並不是 Kubernetes 內部的 Pod**，而是運行在 student-node 節點上的一個外部服務。
    Kubernetes Service 預設只會將請求路由到符合 Selector 的 Pod，但這裡 Web 伺服器 並不是一個 Pod，所以它不會自動出現在 Service 的 Endpoints。
    Service 需要一個 Endpoints

    kubectl describe svc external-webserver-ckad01-svcn 顯示 Endpoints: <none>，代表這個 Service 沒有對應的 Endpoint，所以流量無法轉發。

    故手動建立Endpoint

    **此題為Kubernetes Service 無法直接偵測外部 Web 伺服器**
    **Service 預設會透過 Label Selector 綁定 Kubernetes 內的 Pod**，但**這次的 Web 伺服器是在 Kubernetes 叢集之外**，所以 **Label Selector 無法自動找到它，必須手動配置 Endpoints**。

    (剛剛我自己設置的selecter也需要刪掉!!)

    是否需要手動添加Endpiont,需要判斷服務是叢集內or叢集外!

    **如何判斷服務屬於叢集內or叢集外**?
    叢集內 (Pod)
    kubectl get pod --show-labels 確認 Pod 是否存在，並且有對應的 Label

    **如何判斷服務屬於叢集外**？
    該服務並不是運行在 Kubernetes 叢集內部，而是運行在某個 外部的 VM、**外部Node**，或雲端的 API。 (Cluster> Node，同cluster底下的node可以自由通訊)
    kubectl get pod --show-labels 找不到對應的 Pod，因為該服務根本不是一個 Pod。
    該如何處理？
    ✅ 手動創建 Endpoints
    ✅ 確保 Service Selector 為空:
        不應該設定 Selector，否則 Kubernetes 會自動嘗試尋找符合的 Pod，結果會導致 Service 無法匹配 Endpoints：



    Cluster3 是 Kubernetes 叢集，其中包含：

    cluster3-controlplane（控制平面 / Master）
    cluster3-node01（工作節點）
    其他可能的工作節點
    student-node 是 Kubernetes 叢集之外的機器（可能是 VM 或物理機）。

    它運行了一個 Web 伺服器 (student-node:9999)。
    Service external-webserver-ckad01-svcn 在 Cluster3 內部

    但 因為 Web 伺服器不在 Cluster3 內，所以 Service 找不到 Pod 對應。
    這時候 kubectl get endpoints external-webserver-ckad01-svcn 可能會顯示 <none>，導致流量無法轉發。
    解決方案

    手動建立 Endpoints，指向 student-node:9999，讓 Service 知道 Web 伺服器的位置。
    這樣，Cluster3 內的 Pod 就可以透過 external-webserver-ckad01-svcn 訪問 student-node:9999。


    查看student-node的ip地址:
    student-node ~ ✖ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host 
        valid_lft forever preferred_lft forever
    3: eth0@if7169: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
        link/ether 72:4a:43:aa:f4:fc brd ff:ff:ff:ff:ff:ff link-netnsid 0
        inet **192.168.36.212**/32 scope global eth0
        valid_lft forever preferred_lft forever
        inet6 fe80::704a:43ff:feaa:f4fc/64 scope link 
        valid_lft forever preferred_lft forever

    將student-node地址添加進Endpoint設置:
    student-node ~ ➜  vim external-webserver-ckad01-ep.yaml
    apiVersion: v1
    kind: Endpoints
    metadata:
    name: **external-webserver-ckad01-svcn** # 需要跟service同名!!!
    subsets:
    - addresses:
        - ip: 192.168.36.212

    student-node ~ ✖ k create -f  external-webserver-ckad01-ep.yaml
    endpoints/external-webserver-ckad01-svcn created

    student-node ~ ➜  k get ep 
    NAME                             ENDPOINTS              AGE
    external-webserver-ckad01-svcn   192.168.36.212         5s
    kubernetes                       192.168.129.221:6443   103m

    student-node ~ ➜  k describe svc external-webserver-ckad01-svcn 
    Name:              external-webserver-ckad01-svcn
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          <none>
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                172.20.134.246
    IPs:               172.20.134.246
    Port:              <unset>  80/TCP
    TargetPort:        9999/TCP
    Endpoints:         192.168.36.212
    Session Affinity:  None
    Events:            <none>

    驗證:

    student-node ~ ➜  k run --context cluster3 tmp --image=nginx:alpine --rm --restart=Never -it -- curl student-node:9999
    curl: (6) Could not resolve host: student-node
    pod "tmp" deleted
    pod default/tmp terminated (Error)

    student-node ~ ✖ k run --context cluster3 tmp --image=nginx:alpine --rm --restart=Never -it -- curl external-webserver-ckad01-svcn.default:80
    If you don't see a command prompt, try pressing enter.
    warning: couldn't attach to pod/tmp, falling back to streaming logs: Internal error occurred: error attaching to container: failed to load task: no running task found: task e5a3a78ff441b6f5ad3773a61306b071661592f5da0e81635f5e2c5c0f80afb0 not found: not found
    curl: (7) Failed to connect to external-webserver-ckad01-svcn.default port 80 after 109 ms: Could not connect to server
    pod "tmp" deleted
    pod default/tmp terminated (Error)

    student-node ~ ✖ k run --context cluster3 tmp --image=nginx:alpine --rm --restart=Never -it -- curl external-webserver-ckad01-svcn
    curl: (7) Failed to connect to external-webserver-ckad01-svcn port 80 after 676 ms: Could not connect to server
    pod "tmp" deleted
    pod default/tmp terminated (Error)

    由於沒有在Endpoints設置port! curl 當然不成功!!

    重來:
    student-node ~ ✖ vim external-webserver-ckad01-ep.yaml 
    apiVersion: v1
    kind: Endpoints
    metadata:
    name: external-webserver-ckad01-svcn
    subsets:
    - addresses:
        - ip: 192.168.36.212
        ports:
        - port: 9999

    student-node ~ ➜  k replace -f external-webserver-ckad01-ep.yaml --force
    endpoints/external-webserver-ckad01-svcn replaced

    student-node ~ ➜  k get ep
    NAME                             ENDPOINTS              AGE
    external-webserver-ckad01-svcn   192.168.36.212:9999    7s
    kubernetes                       192.168.129.221:6443   112m

    student-node ~ ➜  k describe svc external-webserver-ckad01-svcn 
    Name:              external-webserver-ckad01-svcn
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          <none>
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                172.20.134.246
    IPs:               172.20.134.246
    Port:              <unset>  80/TCP
    TargetPort:        9999/TCP
    Endpoints:         192.168.36.212:9999
    Session Affinity:  None
    Events:            <none>

    (成功)
    student-node ~ ➜  kubectl --context cluster3 run --rm  -i tmp --image=nginx:alpine --restart=N
    ever -- curl external-webserver-ckad01-svcn
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100   612  100   612    0     0   2185      0 --:--:-- --:--:-- --:--:--  2185
    ...
    pod "tmp" deleted

    student-node ~ ➜  kubectl --context cluster3 run --rm  -i tmp --image=nginx:alpine --restart=Never -- curl external-webserver-ckad01-svcn.default:80
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100   612  100   612    0     0   1043      0 --:--:-- --:--:-- --:--:--  1042
    <!DOCTYPE html>
    ...
    pod "tmp" deleted



17. Create a ResourceQuota called ckad16-rqc in the namespace ckad16-rqc-ns and enforce **a limit of one ResourceQuota** for the namespace.
    
    student-node ~ ➜  vim ckad16-rqc.yaml

    student-node ~ ➜  kr ckad16-rqc.yaml --force
    resourcequota/ckad16-rqc replaced

    student-node ~ ➜  k get resourcequotas -n ckad16-rqc-ns 
    NAME         AGE   REQUEST               LIMIT
    ckad16-rqc   11s   resourcequotas: 1/1   

    student-node ~ ➜  cat ckad16-rqc.yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
    name: ckad16-rqc
    namespace: ckad16-rqc-ns
    spec:
    hard:
        resourcequotas: "1"


### Mock 02:
錯題: 02,04,07,10 ,14,16,17,18,20,21
難題: 01, 14(CRD), 19 (提取欄位)

1. (這題sidecar container的配置跟killer1.32 第16題的配置不同，故此題先忽略)

    student-node ~ ➜  k replace  -f  ckad-sidecar-pod.yaml --force
    pod/ckad-sidecar-pod replaced

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME               READY   STATUS            RESTARTS   AGE
    ckad-sidecar-pod   1/2     PodInitializing   0          2s

    student-node ~ ➜  k logs -n ckad-multi-containers ckad-sidecar-pod -c sidecar-container

    Every 2s: cat /var/log/index.html                                        2025-03-14 01:24:58

    12: Fri Mar 14 01:24:58 UTC 2025 Hi I am from Sidecar container


    ^C
    / # 
    / # exit
    command terminated with exit code 130

    student-node ~ ✖ k get pod -n ckad-multi-containers 
    NAME               READY   STATUS    RESTARTS   AGE
    ckad-sidecar-pod   2/2     Running   0          96s

    student-node ~ ➜  cat ckad-sidecar-pod.yaml 
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
        name: ckad-sidecar-pod
        volumeMounts:
        - name: my-vol
            mountPath: /usr/share/nginx/html
    initContainers:
    - image: busybox:1.28
        name: sidecar-container
        command: ["/bin/sh","-c",'i=0; while true; do echo "$i: $(date) Hi I am from Sidecar container" > /var/log/index.html ; i=$((i+1)); sleep 5; done']
        volumeMounts:
        - name: my-vol
            mountPath: /var/log
        restartPolicy: Always
    volumes:
    - name: my-vol
        emptyDir: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    student-node ~ ➜  k exec -it -n ckad-multi-containers ckad-sidecar-pod -c sidecar-container -- sh
    / # cat /var/log/index.html
    26: Fri Mar 14 01:26:08 UTC 2025 Hi I am from Sidecar container


7. (Helm) (這題還沒做)

14. Define a Kubernetes custom resource definition (CRD) for a new resource kind called Foo (plural form - foos) in the samplecontroller.k8s.io group. (這題解答添加了: annotations: "api-approved.kubernetes.io": "unapproved, experimental-only"，此題型CKAD考試不會考，故忽略)

    This CRD should have a version of v1alpha1 with a schema that includes two properties as given below:
    deploymentName (a string type) and replicas (an integer type with minimum value of 1 and maximum value of 5).

    It should also include a status subresource which enables retrieving and updating the status of Foo object, including the availableReplicas property, which is an integer type.
    The Foo resource should be namespace scoped.

    Note: We have provided a template /root/foo-crd-aecs.yaml for your ease. There are few issues with it so please make sure to incorporate the above requirements before deploying on cluster.


    student-node ~ ➜  k create -f  Foo.yaml
    Error from server (BadRequest): error when creating "Foo.yaml": CustomResourceDefinition in version "v1" cannot be handled as a CustomResourceDefinition: strict decoding error: unknown field "metadata.annotaions"

    student-node ~ ✖ cat Foo.yaml
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
        name: foos.samplecontroller.k8s.io
        annotaions:
            **api-approved.kubernetes.io: "unapproved"**
    spec:
        group: samplecontroller.k8s.io
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
                status:
                    type: object
                    properties:
                    availableReplicas:
                        type: integer
            subresources:
                # status enables the status subresource.
            status: {}


19. Identify the kube api-resources that use api_version=storage.k8s.io/v1 using kubectl command line interface and store them in /root/api-version.txt on student-node. (簡單題)
    student-node ~ ➜  k api-resources --sort-by=name |grep storage.k8s.io/v > api-version.txt

    student-node ~ ➜  cat api-version.txt 
    csidrivers                                     storage.k8s.io/v1                   false        CSIDriver
    csinodes                                       storage.k8s.io/v1                   false        CSINode
    csistoragecapacities                           storage.k8s.io/v1                   true         CSIStorageCapacity
    storageclasses                    sc           storage.k8s.io/v1                   false        StorageClass
    volumeattachments                              storage.k8s.io/v1                   false        VolumeAttachment



### Mock 03:
錯題: 02,04,05,07,09,10,14,16,20,21
難題: 12 (Service Account), 17(Endpiont)

12. We have a Kubernetes namespace called ckad12-ctm-sa-aecs, which contains a service account and a pod. Your task is to modify the pod so that it uses the service account defined in the same namespace. (小心下方有同樣的配置，導致被覆蓋!!!) (簡單題)

    Additionally, you need to ensure that the pod has access to the API credentials associated with the service account by enabling the automounting feature for the credentials.

    student-node ~ ➜  k get sa -n ckad12-ctm-sa-aecs 
    NAME                       SECRETS   AGE
    default                    0         15s
    ckad12-my-custom-sa-aecs   0         15s

    student-node ~ ➜  k get pod -n ckad12-ctm-sa-aecs ckad12-ctm-nginx-aecs -o yaml > ckad12-ctm-nginx-aecs.yaml

    student-node ~ ➜  k get pod -n ckad12-ctm-sa-aecs 
    NAME                    READY   STATUS    RESTARTS   AGE
    ckad12-ctm-nginx-aecs   1/1     Running   0          7s

    student-node ~ ➜  cat ckad12-ctm-nginx-aecs.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"ckad12-ctm-nginx-aecs","namespace":"ckad12-ctm-sa-aecs"},"spec":{"automountServiceAccountToken":false,"containers":[{"image":"nginx","name":"nginx"}]}}
    creationTimestamp: "2025-03-14T01:56:35Z"
    name: ckad12-ctm-nginx-aecs
    namespace: ckad12-ctm-sa-aecs
    resourceVersion: "2410"
    uid: d3560176-62a4-4f6e-99ce-221387a83e5e
    spec:
    serviceAccountName: ckad12-my-custom-sa-aecs 
    (把:automountServiceAccountToken: false 刪除)
    containers:
    - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File

    student-node ~ ➜  vim ckad12-ctm-nginx-aecs.yaml 

    student-node ~ ➜  k replace -f  ckad12-ctm-nginx-aecs.yaml --force
    pod "ckad12-ctm-nginx-aecs" deleted
    pod/ckad12-ctm-nginx-aecs replaced

    student-node ~ ➜  k describe pod -n ckad12-ctm-sa-aecs ckad12-ctm-nginx-aecs |grep Service (重啟後發現Service Account沒有套用!)
    Service Account:  default

    spec:
    **serviceAccountName: ckad12-my-custom-sa-aecs**
    containers:
    - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: cluster2-node01
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    **serviceAccountName: default** # 被覆蓋了!!!



17. A pod named ckad-nginx-pod-aom is deployed and exposed with a service ckad-nginx-service-aom, but it seems the service is not configured properly and is not selecting the correct pod. (簡單題)

    Make the required changes to service and ensure the endpoint is configured for service.

    student-node ~ ➜  k get pod --show-labels 
    NAME                              READY   STATUS    RESTARTS   AGE     LABELS
    nginx-app-ckad-5c95f547fb-p4qnq   1/1     Running   0          7m36s   app=nginx-app-ckad,pod-template-hash=5c95f547fb,scenario=multiport
    nginx-app-ckad-5c95f547fb-wq4mv   1/1     Running   0          7m36s   app=nginx-app-ckad,pod-template-hash=5c95f547fb,scenario=multiport
    app-ckad-svcn                     1/1     Running   0          7m36s   app=app-ckad,scenario=multiport
    nginx-app-ckad-5c95f547fb-x6t5k   1/1     Running   0          7m36s   app=nginx-app-ckad,pod-template-hash=5c95f547fb,scenario=multiport
    ckad-nginx-pod-aom                1/1     Running   0          2m43s   app=nginx

    student-node ~ ➜  k get svc ckad-nginx-service-aom 
    NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
    ckad-nginx-service-aom   ClusterIP   10.43.96.103   <none>        80/TCP    2m49s

    student-node ~ ➜  k describe svc ckad-nginx-service-aom 
    Name:              ckad-nginx-service-aom
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          app=ngnix
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                10.43.96.103
    IPs:               10.43.96.103
    Port:              80-80  80/TCP
    TargetPort:        80/TCP
    Endpoints:         <none>
    Session Affinity:  None
    Events:            <none>

    student-node ~ ➜  k edit svc ckad-nginx-service-aom 
    service/ckad-nginx-service-aom edited

    student-node ~ ➜  k describe svc ckad-nginx-service-aom 
    Name:              ckad-nginx-service-aom
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          app=nginx
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                10.43.96.103
    IPs:               10.43.96.103
    Port:              80-80  80/TCP
    TargetPort:        80/TCP
    Endpoints:         10.42.1.5:80
    Session Affinity:  None
    Events:            <none>


### Mock 04:
錯題: 02,03,04,09,10,11,13,15
難題: 06(helm), ~~17(CRD)~~ 同Mock01-14, 20 (k top pod )

6. (Helm) On the cluster1, the team has installed multiple helm charts on a different namespace. By mistake, those deployed resources include one of the vulnerable images called kodekloud/click-counter:latest. Find out the release name and uninstall it.

    student-node ~ ✖ helm list -A 
    NAME                    NAMESPACE               REVISION        UPDATED                                 STATUS     CHART                           APP VERSION
    atlanta-page-apd        atlanta-page-04         1               2025-03-14 02:16:34.852804953 +0000 UTC deployed   atlanta-page-apd-0.1.0          1.16.0     
    digi-locker-apd         digi-locker-02          1               2025-03-14 02:16:33.754077315 +0000 UTC deployed   digi-locker-apd-0.1.0           1.16.0     
    security-alpha-apd      security-alpha-01       1               2025-03-14 02:16:33.215772795 +0000 UTC deployed   security-alpha-apd-0.1.0        1.16.0     
    traefik                 kube-system             1               2025-03-14 01:55:19.428033098 +0000 UTC deployed   traefik-25.0.2+up25.0.0         v2.10.5    
    traefik-crd             kube-system             1               2025-03-14 01:55:01.726297003 +0000 UTC deployed   traefik-crd-25.0.2+up25.0.0     v2.10.5    
    web-dashboard-apd       web-dashboard-03        1               2025-03-14 02:16:34.343924524 +0000 UTC deployed   web-dashboard-apd-0.1.0         1.16.0     


    
    student-node ~ ➜  kubectl get pods -o=jsonpath="{.items[*]['metadata.name', 'status.capacity']}"

    student-node ~ ➜  kubectl get deploy -n atlanta-page-04 -o=jsonpath="{.items[*]['metadata.name']}"
    atlanta-page-apd
    student-node ~ ➜  kubectl get deploy -n atlanta-page-04 -o=jsonpath="{.items[*]['spec.template.spec.containers']}"
    [{"image":"kodekloud/webapp-color:v1","imagePullPolicy":"IfNotPresent","name":"atlanta-page-apd","resources":{},"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File"}]
    student-node ~ ➜  kubectl get deploy -n atlanta-page-04 -o=jsonpath="{.items[*]['spec.template.spec.containers.iamge']}"
    

    注意! **containers[*]**
    student-node ~ ✖ kubectl get deploy -n atlanta-page-04 -o=jsonpath="{.items[*].spec.template.spec.containers[*].image}"
    kodekloud/webapp-color:v1

    student-node ~ ➜  kubectl get deploy -n digi-locker-02 -o=jsonpath="{.items[*].spec.template.spec.containers[*].image}"
    kodekloud/webapp-color:v2
    student-node ~ ➜  kubectl get deploy -n security-alpha-01 -o=jsonpath="{.items[*].spec.template.spec.containers[*].image
    }"
    kodekloud/click-counter:latest
    student-node ~ ➜  kubectl get deploy -n kube-system -o=jsonpath="{.items[*].spec.template.spec.containers[*].image}"
    rancher/local-path-provisioner:v0.0.24 rancher/mirrored-coredns-coredns:1.10.1 rancher/mirrored-metrics-server:v0.6.3 rancher/mirrored-library-traefik:2.10.5
    student-node ~ ➜  kubectl get deploy -n web-dashboard-03 -o=jsonpath="{.items[*].spec.template.spec.containers[*].image}
    "
    kodekloud/webapp-color:v3

    student-node ~ ✖ helm uninstall security-alpha-apd -n security-alpha-01
    release "security-alpha-apd" uninstalled


20. Three pods hulk,thor and ironman were created on cluster1. Of the three pods, identify the following and copy them to below file, (簡單題)

    Pod with high memory usage
    Memory limit configured to the identified pod.

    copy them as Podname,Memorylimit to /root/pod-metrics file on student-node.

    student-node ~ ✖ k top pod 
    NAME                             CPU(cores)   MEMORY(bytes)   
    foo-controller-7b6956d98-q5d6x   2m           6Mi             
    hulk                             136m         250Mi           
    ironman                          20m          30Mi            
    kodekloud-logs-aom               1m           0Mi             
    thor                             1m           16Mi            

    student-node ~ ➜  k describe pod hulk 
    Name:             hulk
    Namespace:        default
    Priority:         0
    Service Account:  default
    Node:             cluster1-node02/192.168.81.134
    Start Time:       Fri, 14 Mar 2025 02:34:05 +0000
    Labels:           <none>
    Annotations:      <none>
    Status:           Running
    IP:               10.42.2.9
    IPs:
    IP:  10.42.2.9
    Containers:
    mem-stress:
        Container ID:  containerd://b29c185b4e29816b5f93479ac2e041fefec41b589e3ff854b7008850b7306eba
        Image:         polinux/stress
        Image ID:      docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa
        Port:          <none>
        Host Port:     <none>
        Command:
        stress
        Args:
        --vm
        1
        --vm-bytes
        250M
        --vm-hang
        1
        State:          Running
        Started:      Fri, 14 Mar 2025 02:34:07 +0000
        Ready:          True
        Restart Count:  0
        Limits:
        memory:  400Mi
        Requests:
        memory:     400Mi
        Environment:  <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pp44m (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       True 
    ContainersReady             True 
    PodScheduled                True 
    Volumes:
    kube-api-access-pp44m:
        Type:                    Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds:  3607
        ConfigMapName:           kube-root-ca.crt
        ConfigMapOptional:       <nil>
        DownwardAPI:             true
    QoS Class:                   Burstable
    Node-Selectors:              <none>
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
    Type    Reason     Age   From               Message
    ----    ------     ----  ----               -------
    Normal  Scheduled  68s   default-scheduler  Successfully assigned default/hulk to cluster1-node02
    Normal  Pulling    67s   kubelet            Pulling image "polinux/stress"
    Normal  Pulled     66s   kubelet            Successfully pulled image "polinux/stress" in 687ms (687ms including waiting)
    Normal  Created    66s   kubelet            Created container mem-stress
    Normal  Started    66s   kubelet            Started container mem-stress

    student-node ~ ➜  k get pod hulk -o yaml > hulk.yaml

    student-node ~ ➜  vim hulk.yaml 

    student-node ~ ➜  k get pod hulk -o custom-columns=Podname:metadata.name,Memorylimit:spec.containers.resources.limits.memory
    Podname   Memorylimit
    hulk      <none>

    student-node ~ ➜  k get pod hulk -o custom-columns=Podname:metadata.name,Memorylimit:spec.**containers[]**.resources.limits.memory
    Podname   Memorylimit
    hulk      400Mi

    student-node ~ ➜  k get pod hulk -o custom-columns=Podname:metadata.name,Memorylimit:spec.containers[].resources.limits.memory > pod-metrics

    student-node ~ ➜  cat pod-metrics 
    Podname   Memorylimit
    hulk      400Mi



### Mock 05:
錯題: 01,05,07,10,11,12,17
難題: 02 (Persistent Volume回收機制), 08 (Helm uninstall 之後檢查是否有殘留資源)
應該不會考: ~~11~~ (建立Ingress Controller)

7. (helm upgrade --set replicaCount) One co-worker deployed an nginx helm chart on the cluster3 server called lvm-crystal-apd. A new update is pushed to the helm chart, and the team wants you to update the helm repository to fetch the new changes.
    
    After updating the helm chart, upgrade the helm chart version to 18.1.15 and increase the replica count to 2.
    NOTE: - We have to perform this task on the cluster3-controlplane node.
    You can SSH into the cluster3 using ssh cluster3-controlplane command.



### Mock 06:
錯題: 02,05,08,11,12,13,14,17,19,20
難題: 01 (storageClass 為maual), 07 (jq tool)

1. 

7. 

### Mock 07:
錯題: 2,4,5,8,10,11,12,17
難題: 1,13

1. 

5. The team Garuda has deployed one application in the testing-apd namespace. 
    
    The testing was done, and the team wants you to delete that release. Find out the release name and delete it.    
    NOTE: - Do not worry about the status of the deployment.

13. 


### Mock 08:


### killer.sh
錯題: ~~06~~,10,20
難題/空題: 9,11,15,16,17,18,19,22
