First Attempt
Socre: 40% Pass Score: 75%
對題: 01,05,07,08,12,14,16,18,19,21
錯題: 02,03,04,13
空題: 06,09~11,20
難題: 17 (CRD 設置annotation)
1.32 新Concept: Privileged Container (01)
不熟的topic: helm(06), CRD(17), k top pod (20)


### SECTION: APPLICATION DESIGN AND BUILD
01. In the ckad-pod-design namespace, create a pod named privileged-pod that runs the nginx:1.17 image, and the container should be run in privileged mode.
    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1




02. (Job 層級的配置誤寫到Pod層級內導致設置沒有生效) In the ckad-job namespace, create a job named very-long-pi that simply computes a π (pi) to 1024 places and prints it out.

    This job should be configured to retry maximum 5 times before marking this job failed, and the duration of this job should not exceed 100 seconds.

    Use perl:5.34.0 image for your container.
    
    我的配置:
    student-node ~ ➜  kr very-long-pi.yaml --force
    job.batch "very-long-pi" deleted
    job.batch/very-long-pi replaced

    student-node ~ ➜  k get job -n ckad-job 
    NAME           COMPLETIONS   DURATION   AGE
    very-long-pi   1/1           5s         6s

    student-node ~ ➜  cat very-long-pi.yaml 
    apiVersion: batch/v1
    kind: Job
    metadata:
    creationTimestamp: null
    name: very-long-pi
    namespace: ckad-job
    spec:
        backoffLimit: 5
        # activeDeadlineSeconds: 100 --------> 應該要設置在這裡
        template:
            metadata:
            creationTimestamp: null
            spec:
                containers:
                - image: perl:5.34.0
                    name: very-long-pi
                    command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(1024)"]
                    resources: {}
                restartPolicy: Never
                ~~activeDeadlineSeconds: 100~~ # 這個不能設置在這裡，這是Job的property
    status: {}

    student-node ~ ➜  k get pod -n ckad-job 
    NAME                 READY   STATUS      RESTARTS   AGE
    very-long-pi-kpc6p   0/1     Completed   0          122m

    student-node ~ ➜  k logs -n ckad-job very-long-pi-kpc6p 
    3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141273724587006606315588174881520920962829254091715364367892590360011330530548820466521384146951941511609433057270365759591953092186117381932611793105118548074462379962749567351885752724891227938183011949129833673362440656643086021394946395224737190702179860943702770539217176293176752384674818467669405132000568127145263560827785771342757789609173637178721468440901224953430146549585371050792279689258923542019956112129021960864034418159813629774771309960518707211349999998372978049951059731732816096318595024459455346908302642522308253344685035261931188171010003137838752886587533208381420617177669147303598253490428755468731159562863882353787593751957781857780532171226806613001927876611195909216420198938095257201065485863279

    為何我的配置沒有報錯？
    Kubernetes 允許未知字段：

    Kubernetes API 解析 YAML 時，對於無法識別的字段（例如 containers 內的 activeDeadlineSeconds），它並不會直接報錯，而是默默忽略它。
    activeDeadlineSeconds 只適用於 Job 層級：

    activeDeadlineSeconds 是 Job 的 spec 屬性，應該直接放在 spec 下，而不是 template.spec.containers 內。放在錯誤的層級時，Kubernetes 不會生效。


    Solution:

    Use below YAML to create job:

    apiVersion: batch/v1
    kind: Job
    metadata:
    name: very-long-pi
    namespace: ckad-job
    spec:
        backoffLimit: 5
        **activeDeadlineSeconds: 100**
        template:
            spec:
                containers:
                - name: pi
                    image: perl:5.34.0
                    command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(1024)"]
            restartPolicy: Never



    You can verify the output by running below command against the job's pod (noted that the pod name will be different):

    Identify pod name by this command:
    kubectl get pod -n ckad-job -l job-name=very-long-pi



03. (**Tricky!!! 題目沒有指定hostPath -> 那就自定義!!!**) Create a persistent volume called red-pv-ckad03-str of type: hostPath and capacity: 100Mi.
    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    student-node ~ ➜  k get storageclasses.storage.k8s.io 
    NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
    local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  29m

    student-node ~ ➜  k describe storageclasses.storage.k8s.io  local-path 
    Name:                  local-path
    IsDefaultClass:        Yes
    Annotations:           objectset.rio.cattle.io/applied=H4sIAAAAAAAA/4yRT+vUMBCGv4rMua1bu1tKwIOu7EUEQdDzNJlux6aZkkwry7LfXbIqrIffn2PyZN7hfXIFXPg7xcQSwEBSiXimaupSxfJ2q6GAiYMDA9/+oKPHlKCAmRQdKoK5AoYgisoSUj5K/5OsJtIqslQWVT3lNM4xUDzJ5VegWJ63CQxMTXogW128+czBvf/gnIQXIwLOBAa8WPTl30qvGkoL2jw5rT2V6ZKUZij+SbG5eZVRDKR0F8SpdDTg6rW8YzCgcSW4FeCxJ/+sjxHTCAbqrhmag20Pw9DbZtfu210z7JuhPnQ719m2w3cOe7fPof81W1DHfLlE2Th/IEUwEDHYkWJe8PCsgJgL8PxVPNsLGPhEnjRr2cSvM33k4Dicv4jLC34g60niiWPSo4S0zhTh9jsAAP//ytgh5S0CAAA,objectset.rio.cattle.io/id=,objectset.rio.cattle.io/owner-gvk=k3s.cattle.io/v1, Kind=Addon,objectset.rio.cattle.io/owner-name=local-storage,objectset.rio.cattle.io/owner-namespace=kube-system,storageclass.kubernetes.io/is-default-class=true
    Provisioner:           rancher.io/local-path
    Parameters:            <none>
    AllowVolumeExpansion:  <unset>
    MountOptions:          <none>
    ReclaimPolicy:         Delete
    VolumeBindingMode:     WaitForFirstConsumer
    Events:                <none>

    student-node ~ ➜  k get StorageClass
    NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
    local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  31m

    student-node ~ ➜  cat red-pv-ckad3-str.yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
    name: red-pv-ckad03-str
    spec:
    capacity:
        storage: 100Mi
    accessModes:
        - ReadWriteOnce
    storageClassName: local-path
    #hostPath:
        #path:  # 如果題目沒有指定 hostPath 的 path 值，**那麼你可以自行決定這個目錄的位置**!!!

    
    
    所以:

    student-node ~ ➜  ls -l /opt
    total 4
    -rw-r--r-- 1 root root 90 Feb 26 15:11 ocean-revision-count.txt

    student-node ~ ➜  mkdir  /opt//red-pv-ckad03-str

    student-node ~ ➜  ls -l /opt
    total 8
    -rw-r--r-- 1 root root   90 Feb 26 15:11 ocean-revision-count.txt
    drwxr-xr-x 2 root root 4096 Feb 26 16:30 red-pv-ckad03-str

    student-node ~ ➜  **chmod 777 /opt/red-pv-ckad03-str** # 記得添加write權限!!!

    student-node ~ ➜  ls -l /opt
    total 8
    -rw-r--r-- 1 root root   90 Feb 26 15:11 ocean-revision-count.txt
    drwxrwxrwx 2 root root 4096 Feb 26 16:30 red-pv-ckad03-str

    #hostPath:
    #path:  # 就可以寫為: /opt/red-pv-ckad03-str


    Solution:
    Use below YAML to create required volume:

    apiVersion: v1
    kind: PersistentVolume
    metadata:
    name: red-pv-ckad03-str
    spec:
    capacity:
        storage: 100Mi
    accessModes:
        - ReadWriteOnce
    hostPath:
       **path: /opt/red-pv-ckad03-str** # 解答給的是在/opt 底下的red-pv-ckad03-str位置


04. (沒有輸出內容到正確的volume路徑內) In the ckad-multi-containers namespaces, create a ckad-web-pod pod that matches the following requirements.
     Pod has an emptyDir volume named my-vol.

    The first container named web-app, runs nginx:1.16 image. This container mounts the my-vol volume at /usr/share/nginx/html path.

    The second container named log-container, runs alpine image. This container mounts the my-vol volume at /var/log path.

    Every 5 seconds, this container should **write the current date along with greeting message** Hi I am from Sidecar container to index.html **in the my-vol volume**.
    (此問題類同Mock02-第01題)

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    我的配置:
        student-node ~ ✖ cat ckad-web-pod.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: ckad-web-pod
    name: ckad-web-pod
    namespace: ckad-multi-containers
    spec:
    containers:
        - name: web-app
        image: nginx:1.16
        volumeMounts:
            - name: my-vol
            mountPath: /usr/share/nginx/html
        - name: log-container
        image: alpine
        volumeMounts:
            - name: my-vol
            mountPath: /var/log
        args: [/bin/sh, -c,   # 我沒有配置date!!!!  -> **write the current date along with greeting message** 
                'i=0; while true; do echo "Hi I am from Sidecar container" > ~~index.html~~; i=$((i+1)); sleep 5; done']     # 應為: /var/log/index.html!!!  -> **in the my-vol volume**
    volumes:
        - name: my-vol
        emptyDir: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    Solution:

    Use below YAML to create the desired pod:
    apiVersion: v1
    kind: Pod
    metadata:
    namespace: ckad-multi-containers
    name: ckad-web-pod
    spec:
    containers:
        - image: nginx:1.16
        name: web-app
        resources: {}
        ports:
            - containerPort: 80
        volumeMounts:
            - name: my-vol
            mountPath: /usr/share/nginx/html
        - image: alpine
        command:
            - /bin/sh
            - -c
            - while true; do echo $(date -u) Hi I am from Sidecar container >> /var/log/index.html; sleep 5;done   
        name: log-container
        resources: {}
        volumeMounts:
            - name: my-vol
            mountPath: /var/log
    dnsPolicy: Default
    volumes:
        - name: my-vol
        emptyDir: {}




### SECTION: APPLICATION DEPLOYMENT
05. (這題答對 RollingUpdate改Recreate) A web application running on cluster2 called robox-west-apd on the fusion-apd-x1df5 namespace. The Ops team has created a new service account with a set of permissions for this web application. Update the newly created SA for this deployment.
    student-node ~ ➜  k get serviceaccounts -n fusion-apd-x1df5 
    NAME              SECRETS   AGE
    default           0         17m
    galaxy-apd-xb12   0         17m
    
    student-node ~ ➜  k get deploy -n fusion-apd-x1df5 robox-west-apd
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    robox-west-apd   3/3     3            3           10s


    Also, change the strategy type to Recreate, so it will delete all the pods immediately and update the newly created SA to all the pods.

    student-node ~ ✖ k get deploy -n fusion-apd-x1df5 robox-west-apd -o yaml > fusion-apd-x1df5.yaml

    student-node ~ ➜  vim fusion-apd-x1df5.yaml 

    student-node ~ ➜  kr fusion-apd-x1df5.yaml --force
    deployment.apps "robox-west-apd" deleted
    The Deployment "robox-west-apd" is invalid: spec.strategy.rollingUpdate: Forbidden: may not be specified when strategy `type` is 'Recreate'

    (當 strategy.type 設置為 Recreate 時，不允許指定 rollingUpdate 設定。
    錯誤原因：

    rollingUpdate 這些設定（如 maxSurge 和 maxUnavailable）僅適用於 RollingUpdate 策略。
    你在 Recreate 策略中嘗試使用 rollingUpdate 設定，這是不允許的。)

    spec:
    progressDeadlineSeconds: 600
    replicas: 3
    revisionHistoryLimit: 10
    selector:
        matchLabels:
        global-kgh: robox-west-apd
    strategy:
        **type: Recreate**
    #strategy:
        #rollingUpdate:
        #maxSurge: 25%
        #maxUnavailable: 25%
    template:
        metadata:
        creationTimestamp: null
        labels:
            global-kgh: robox-west-apd
        spec:
        containers:
        - image: nginx
            imagePullPolicy: Always
            name: robox-container
            resources: {}
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        serviceAccount: galaxy-apd-xb12
        serviceAccountName: galaxy-apd-xb12
        terminationGracePeriodSeconds: 30   

    student-node ~ ➜  kr fusion-apd-x1df5.yaml --force
    deployment.apps/robox-west-apd replaced

    student-node ~ ➜  k get deploy -n fusion-apd-x1df5 
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    robox-west-apd   3/3     3            3           7s

    student-node ~ ➜  k describe deploy -n fusion-apd-x1df5 robox-west-apd |grep StrategyType:
    StrategyType:       Recreate




06.  (helm uninstall) On the cluster1, the team has installed multiple helm charts on a different namespace. By mistake, those deployed resources include one of the vulnerable images called kodekloud/click-counter:latest. Find out the release name and uninstall it.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    student-node ~ ➜  helm repo list
    Error: no repositories to show

    student-node ~ ✖ helm list (**要添加: - A 參數列出所有namespace**)
    NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION


    Solution:
    Run the following command to change the context: -

    kubectl config use-context cluster1
    In this task, we will use the helm commands and jq tool. Here are the steps: -

    Run the helm ls command with -A option to list the releases deployed on all the namespaces using helm.

    **helm ls -A** # Or: helm list -A

    We will use the **jq tool to extract the image name from the deployments**. # OR 直接 k describe deploy <> |grep Image: 

    kubectl get deploy -n <NAMESPACE> <DEPLOYMENT-NAME> -o json | jq -r '.spec.template.spec.containers[].image'

    Replace <NAMESPACE> with the namespace and <DEPLOYMENT-NAME> with the deployment name, which we get from the previous commands.

    After finding the kodekloud/click-counter:latest image, use the helm uninstall to remove the deployed chart that are using this vulnerable image.
    helm uninstall <RELEASE-NAME> -n <NAMESPACE>

09. (概念其實很簡單! scale replicas) On cluster2, a new deployment called cube-alpha-apd has been created in the alpha-ns-apd namespace using the image kodekloud/webapp-color:v2. This deployment will test a newer version of the alpha app.

    Configure the deployment in such a way that the alpha-apd-service service routes **less than 40% of traffic to the new deployment**. 
    -> cube-alpha-apd 新的 v2 原本是50 %  題目要求: less than 40% of traffic 。 10 pods * 40 % = 4 pods -> 需要少於4pods, 那就scale down 為3 pods 或是更少(詳解設置為2 pods)
    -> ruby-alpha-apd 舊的 v1 原本是50%
    NOTE: - Do not increase the replicas of the ruby-alpha-apd deployment.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    
    Solution:

    In this task, we will use the kubectl command. Here are the steps: -
    a. The cube-alpha-apd and ruby-alpha-apd deployment has 5-5 replicas. The alpha-apd-service service now routes traffic to 10 pods in total (5 replicas on the ruby-alpha-apd deployment and 5 replicas from cube-alpha-apd deployment).

    Use the kubectl get command to list the following deployments: -
    kubectl get deploy -n alpha-ns-apd

    Since the service distributes traffic to all pods equally, in this case, approximately 50% of the traffic will go to cube-alpha-apd deployment.

    b. To reduce this below 40%, **scale down the pods on the cube-alpha-apd deployment to the minimum to 2**.

    **kubectl scale deployment --replicas=2 cube-alpha-apd -n alpha-ns-apd**

    Once this is done, only ~40% of traffic should go to the v2 version.


### SECTION: SERVICES AND NETWORKING
10. (Fix Ingress Controller Error) We have deployed an application in the green-space namespace. we also deployed the ingress controller and the ingress resource.

    However, currently, the ingress controller is not working as expected. Inspect the ingress definitions and troubleshoot the issue so that the services are accessible as per the ingress resource definition.

    Also, update the path for the app-wear-service to /app-wear and app-video-service to /app-video.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3


    Check the status of the ingress, pods, and application related services.
    cluster3-controlplane ~ ➜  **k get pods -n ingress-nginx**  # 在ingress-nginx裡面找到ingress-controller
    NAME                                        READY   STATUS      RESTARTS      AGE
    ingress-nginx-admission-create-l6fgw        0/1     Completed   0             11m
    ingress-nginx-admission-patch-sfgc4         0/1     Completed   0             11m
    ingress-nginx-controller-5f8964959d-278rc   0/1     Error       2 (26s ago)   29s

    You would see an Error or CrashLoopBackOff in the ingress-nginx-controller. Inspect the logs of the controller pod.

    cluster3-controlplane ~ ✖ k logs -n ingress-nginx ingress-nginx-controller-5f8964959d-278rc 
    /-------------------------------------------------------------------------------
    /--------
    F0316 08:03:28.111614      57 main.go:83] **No service with name default-backend-service found in namespace default**:


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
            - **--default-backend-service=green-space/default-backend-service**   #Changed to correct namespace
            - --controller-class=k8s.io/ingress-nginx
            - --ingress-class=nginx
            - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
            - --validating-webhook=:8443
            - --validating-webhook-certificate=/usr/local/certificates/cert
            - --validating-webhook-key=/usr/local/certificates/key
        
    Apply the manifest, it should be up and running.



11. (Create Ingress) For this scenario, we have already deployed an application in the global-space. Inspect them and create an ingress resource with name ingress-resource-xnz to make the application available at /eat on the Ingress service. Use ingress class of nginx.

    Also, make use of following annotation fields: -

    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"

    Ingress resource comes under the namespace scoped, so don't forget to create the ingress in the global-space namespace.

    Make sure the paths select the correct backend services.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    Switch to cluster3 by using the following command:

    kubectl config use-context cluster3

    To view the applications running on global-space namespace, run the following.

    cluster3-controlplane ~ ➜  **kubectl get pod,svc -n global-space**
    NAME                                 READY   STATUS    RESTARTS   AGE
    pod/default-backend-b46b9989-p9h28   1/1     Running   0          95s
    pod/webapp-food-dcd846f95-khhmw      1/1     Running   0          95s

    NAME                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    service/default-backend-service   ClusterIP   10.102.22.145   <none>        80/TCP     95s
    service/food-service              ClusterIP   10.99.255.77    <none>        8080/TCP   95s

    We have service food-service configured, this will act as backend service for /eat path respectively.

    Using Command Line

    Create an ingress resource using the following imperative command:

    kubectl create ingress ingress-resource-xnz \
    --namespace global-space \
    --rule='/eat'='food-service:8080' \
    --annotation='nginx.ingress.kubernetes.io/rewrite-target=/' \
    --annotation='nginx.ingress.kubernetes.io/ssl-redirect=false' \
    --class=nginx

    Using manifest file

    Solution manifest file to create a ingress resource as follows:
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: ingress-resource-xnz
    namespace: global-space
    annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
        nginx.ingress.kubernetes.io/ssl-redirect: "false"
    spec:
    ingressClassName: nginx
    rules:
    - http:
        paths:
        - **path: /eat**
            pathType: Prefix
            backend:
            service:
            name: food-service
            port:
                number: 8080

12. (**Tricky!!! 題目沒有指定port!!! -> 那就自定義**)Create a Deployment named ckad13-deployment with "two replicas" of nginx image and expose it using a service named ckad13-service.

    Please be noted that service needs to be accessed from both inside and outside the cluster (use port 31080).

    Create the service in the default namespace.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2


    我的配置:

    (沒有複製到depoloyment的配置，但我應該沒有設置containerPort)

    題目沒有指定port/containerPort -> 那就自定義!! 自定義為80
    所以:
        - name: nginx
            image: nginx
            ports:
            - containerPort: 80 # 自定義為80


    student-node ~ ➜  cat ckad13-deployment.yaml 
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        app: ckad13-deployment
    name: ckad13-deployment
    spec:
    ports:
    - port: ~~ckad13-service~~ # 這裡寫成80
        protocol: TCP
    selector:
        app: ckad13-deployment
    status:
    loadBalancer: {}

    然後題目要求:  service needs to be accessed from both inside and outside the cluster
    剛剛上方的配置默認為ClusterIP, 只能在cluster 內部訪問，如果要讓ckad13-deployment被cluster外所訪問，則要配置nodePort!!!
    修改如下:
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        app: ckad13-deployment
    name: ckad13-deployment
    spec:
        ports:
      - **port: 80**
        targetPort: 80 # 既然自定義port為80, 則targetPort也自定義為80
        protocol: TCP
        **nodePort: 31080** # 題目指定!!
    selector:
        app: ckad13-deployment
    status:
    loadBalancer: {}
    

    
    SOLUTION:
    The following manifest can be used to create an deployment ckad13-deployment with nginx image and 2 replicas.
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: ckad13-deployment
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: nginx
    template:
        metadata:
        labels:
            app: nginx
        spec:
        containers:
        - name: nginx
            image: nginx
            ports:
            - containerPort: 80

    To access from outside the cluster, we use nodeport type of service.

    apiVersion: v1
    kind: Service
    metadata:
    name: ckad13-service
    spec:
    selector:
        app: nginx
    type: NodePort
    ports:
        - name: http
        port: 80
        targetPort: 80
        nodePort: 31080


### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY
14.  (這題答對CRD) We have a sample CRD at /root/ckad10-crd-aecs.yaml which should have the following 

    validations:

    destinationName, country, and city must be string types.
    pricePerNight must be an integer between 50 and 5000.
    durationInDays must be an integer between 1 and 30.
    Update the file incorporating the above validations in a namespaced scope.
    Note: Remember to create the CRD after the required changes.
    
    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    student-node ~ ✖ vim ckad10-crd.yaml 
    student-node ~ ✖ cat ckad10-crd.yaml 
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
    name: holidaydestinations.destinations.k8s.io
    annotations:
        "api-approved.kubernetes.io": "unapproved, experimental-only"
    labels:
        app: holiday
    spec:
    group: destinations.k8s.io
    names:
        kind: HolidayDestination
        singular: holidaydestination
        plural: holidaydestinations
        shortNames:
        - hd
    scope: Namespaced
    versions:
        - name: v1alpha1
        served: true
        storage: true
        schema:
            # schema used for validation comes here
            openAPIV3Schema:
            type: object
            properties:
                spec:
                type: object
                properties:
                    durationInDays:
                    type: integer
                    minimum: 1
                    maximum: 30
                    pricePerNight:
                    type: integer
                    minimum: 50
                    maximum: 5000
                    destinationName:
                    type: string
                    country:
                    type: string
                    city:
                    type: string
                    availableRooms:
                    type: integer
                    minimum: 0
                    maximum: 1000
        # subresources for the custom resource
        subresources:
            # enables the status subresource
            status: {}
    student-node ~ ➜  kc ckad10-crd.yaml 
    customresourcedefinition.apiextensions.k8s.io/holidaydestinations.destinations.k8s.io created

15. (ClusterRole 設置non-resource-url錯誤) Create a ClusterRole named healthz-access that allows GET and POST requests to the non-resource endpoint /healthz and **all subpaths**.

    Bound this ClusterRole to a user healthz-user using a ClusterRoleBinding named healthz-access-binding.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    student-node ~ ➜  cat healthz-access.yaml 
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
    /# "namespace" omitted since ClusterRoles are not namespaced
    name: healthz-access
    /#rules:
    /#- apiGroups: [""]
    /#  resources: [""]
    rules:
    - nonResourceURLs: ["/healthz", "~~/subpaths/~~*"]  應該是: ["/healthz", "/healthz/*"]
    verbs: ["get", "post"]


    student-node ~ ➜  ka healthz-access.yaml 
    clusterrole.rbac.authorization.k8s.io/healthz-access created
        

    student-node ~ ➜  kr healthz-access.yaml --force
    clusterrole.rbac.authorization.k8s.io "healthz-access" deleted
    clusterrole.rbac.authorization.k8s.io/healthz-access replaced

    doc: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
    student-node ~ ➜  k get clusterrole|grep health
    healthz-access                                                         2025-02-26T15:33:26Z


    student-node ~ ✖ vim healthz-user.yaml

    student-node ~ ➜  cat healthz-user.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
    name: healthz-user
    subjects:
    - kind: Group
    name: healthz-user # Name is case sensitive
    apiGroup: rbac.authorization.k8s.io
    roleRef:
    kind: ClusterRole
    name: healthz-access
    apiGroup: rbac.authorization.k8s.io

    student-node ~ ➜  kc healthz-user.yaml
    clusterrolebinding.rbac.authorization.k8s.io/healthz-user created


    student-node ~ ➜  k get clusterrolebinding|grep healthz-user
    healthz-user                                             ClusterRole/healthz-access                                                         18s

    Solution:

    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  kubectl create clusterrole healthz-access --non-resource-url=/healthz,/healthz/* --verb=get,post
    clusterrole.rbac.authorization.k8s.io/healthz-access created


    student-node ~ ➜  kubectl create clusterrolebinding healthz-access-binding --clusterrole=healthz-access --user=healthz-user
    clusterrolebinding.rbac.authorization.k8s.io/healthz-access-binding created

17. (CRD status設置錯誤) Define a Kubernetes custom resource definition (CRD) for a new resource kind called Foo (plural form - foos) in the samplecontroller.k8s.io group.

    This CRD should have a version of v1alpha1 with a schema that includes two properties as given below:
    deploymentName (a string type) and replicas (an integer type with minimum value of 1 and maximum value of 5).

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    It should also include a status subresource which enables retrieving and updating the status of Foo object, including the availableReplicas property, which is an integer type.
    The Foo resource should be namespace scoped.

    student-node ~ ✖ vim Foo.yaml
    student-node ~ ✖ cat Foo.yaml
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
        name: foos.stable.example.com
        # 沒有設置annotation!!!! -> **annotations: "api-approved.kubernetes.io": "unapproved, experimental-only"**
        # 
    spec:
        group: samplecontroller.k8s.io
        scope: Namespaced
        names:
            kind: Foo
            plural: foos
            singular: foo
        versions:
        - name: v1alpha1
            served: true
            storage: true
            schema:
                openAPIV3Schema:
                    type: object
                    properties:
                        **spec**:
                            # 沒有設置 type: object
                            ~~status:~~ # status 應設置在下方
                            type: object
                            properties:
                                availableReplicas:
                                    type: integer
                                    minimum: 1
                                    maximum: 5
                                deploymentName:
                                    type: string
                        # **status** 應設置在此 # status 跟spec同層級
                                type: objectt
                                properties:
                                    availableReplicas:
                                        type: integar
                                # 不用設置 deplolymentName
            subresources:
                status: {}   # 開啟 status 子資源
    
    在 Kubernetes 的 CustomResourceDefinition（CRD）設計中，spec 與 status 各自承擔不同的角色：

    spec 用於描述用戶期望的狀態 (desired state)。
    status 用於反映當前觀察到的狀態，通常是系統自動填入的內容。
    為什麼 status 只需要設置 availableReplicas？
    因為 status 的作用是反映實際的資源狀態或執行結果（例如當前真正運行了多少個 Pod），而不是重新聲明你之前已經在 spec 中指定的期望狀態。

    spec：
    描述使用者想要達成的理想狀態（期望）。
    例如 deploymentName 與 replicas 描述了想要建立的部署名稱與所期望的副本數。
    
    status 則記錄當前實際的情況：
    availableReplicas 告訴你目前實際上有多少個 Pod 正在運行。
    你不需要在 status 裡面重新設定 deploymentName，因為這個值本來就存在於 spec 中，不會改變，也不需要重複聲明。


    student-node ~ ➜  kc Foo.yaml
    The CustomResourceDefinition "foos.stable.example.com" is invalid: 
    * metadata.name: Invalid value: "foos.stable.example.com": must be spec.names.plural+"."+spec.group
    * spec.validation.openAPIV3Schema.properties[spec].type: Required value: must not be empty for specified object fields
    * metadata.annotations[api-approved.kubernetes.io]: Required value: protected groups must have approval annotation "api-approved.kubernetes.io", see https://github.com/kubernetes/enhancements/pull/1111

    
    Solution:

    student-node ~ ➜  kubectl config use-context cluster1
    Switched to context "cluster1".

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


### SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE
20. (**k top pods指令**) Three pods hulk,thor and ironman were created on cluster1. Of the three pods, identify the following and copy them to below file,

    Pod with high memory usage
    Memory limit configured to the identified pod.

    copy them as Podname,Memorylimit to /root/pod-metrics file on student-node.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    
    
    Solution:
    Use the following command to find the pod metrics

    **kubectl top pods**  # 直接使用top 查看占用最多memory的pod
    OR
    **kubectl top pod | grep -E 'hulk|thor|ironman'**

    student-node ~ ➜  k top pods --sort-by memory
    NAME                             CPU(cores)   MEMORY(bytes)   
    hulk                             105m         250Mi           
    ironman                          14m          30Mi            
    nginx-resolver-ckad03-svcn       0m           23Mi            
    thor                             1m           16Mi            
    foo-controller-7b6956d98-zjmw8   2m           6Mi             
    kodekloud-logs-aom               1m           0Mi    

    Doc: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_top/kubectl_top_pod/

    Use below command to view the resource limits of pod .
    kubectl get pod hulk -o json | jq -r '.spec.containers[].resources.limits.memory'

    Write the content to/root/pod-metrics file.


    補充:
    
    倘若出現:  Metrics API not available 則需要先安裝Metrics API(但在考試時應該不會出現此狀況)
    controlplane:~$ k top pods
    error: Metrics API not available

    則有兩個方法可以處裡:
    法一: 使用 top + grep 過濾 Kubelet
    你可以透過 top 來查看所有進程的記憶體使用情況，然後手動過濾出與 Kubernetes Pod 相關的進程。

    a. 開啟 top
    b. 按 Shift + M（大寫 M）來根據記憶體使用量排序
    這樣會將佔用最多記憶體的進程排在最上面。
    c. 觀察 COMMAND 欄位，找到 kubelet 相關的容器
    例如，你可能會看到 containerd-shim、kubelet 或 /pause 相關的進程，它們是 Kubernetes 運行的 Pod。


    法二: 使用 ps 來列出佔用最多記憶體的 Pod
    如果 top 介面難以過濾，你也可以直接用 ps 來查看記憶體佔用最高的 Pod：
    ps aux --sort=-%mem | grep -E 'kubelet|containerd|pod'
    解釋：

    ps aux：列出所有進程
    --sort=-%mem：按記憶體使用率降序排序
    grep -E 'kubelet|containerd|pod'：篩選 Kubernetes 相關的進程
    這樣你可以在考試時肉眼快速查看哪些 Pod 佔用最多記憶體。