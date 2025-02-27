Attmept 2 
Score: 74 %
Pass Score: 75%
錯題: 10,13 (Servie 的占比較重)
忘了寫: 8, 12 (Attemp 1是對的，忽略)

01. **SecurityContext 應該放在 container 內，而不是 containers 內!!!**

    apiVersion: v1
    kind: Pod
    metadata:
        creationTimestamp: null
        labels:
            run: privileged-pod
        name: privileged-pod
        namespace: ckad-pod-design
    spec:
        containers:
            ~~securityContext:~~
                ~~privileged: true~~
    - image: nginx:1.17
        name: privileged-pod
        resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    **SecurityContext 應該放在 container 內，而不是 containers 內!!!**

    student-node ~ ➜  cat privileged-pod.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
        labels:
            run: privileged-pod
        name: privileged-pod
        namespace: ckad-pod-design
    spec:
        containers:
        - image: nginx:1.17
            name: privileged-pod
            securityContext:
                privileged: true
                    resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    student-node ~ ➜  kc privileged-pod.yaml 
    pod/privileged-pod created


06. <>

    student-node ~ ✖ helm list -a
    NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION


    student-node ~ ➜  helm list -A
    NAME                    NAMESPACE               REVISION        UPDATED                                 STATUS     CHART                            APP VERSION
    atlanta-page-apd        atlanta-page-04         1               2025-02-27 16:09:00.878518631 +0000 UTC deployed   atlanta-page-apd-0.1.0           1.16.0     
    digi-locker-apd         digi-locker-02          1               2025-02-27 16:08:59.442773811 +0000 UTC deployed   digi-locker-apd-0.1.0            1.16.0     
    security-alpha-apd      security-alpha-01       1               2025-02-27 16:08:58.673705613 +0000 UTC deployed   security-alpha-apd-0.1.0         1.16.0     
    traefik                 kube-system             1               2025-02-27 15:26:41.929353954 +0000 UTC deployed   traefik-25.0.2+up25.0.0          v2.10.5    
    traefik-crd             kube-system             1               2025-02-27 15:26:20.724559111 +0000 UTC deployed   traefik-crd-25.0.2+up25.0.0      v2.10.5    
    web-dashboard-apd       web-dashboard-03        1               2025-02-27 16:09:00.161272803 +0000 UTC deployed   web-dashboard-apd-0.1.0          1.16.0     

    student-node ~ ➜  k describe -n atlanta-page-04 deploy atlanta-page-apd |grep "Image:"
        Image:         kodekloud/webapp-color:v1

    student-node ~ ➜  k describe deploy -n security-alpha-01 security-alpha-apd |grep "Image:"
        Image:         **kodekloud/click-counter:latest** -> 要uninstall 的是這個

    student-node ~ ➜  k describe deploy -n kube-system traefik |grep "Image:"
        Image:       rancher/mirrored-library-traefik:2.10.5

    student-node ~ ➜  k describe deploy -n kube-system traefik-crd |grep "Image:"
    Error from server (NotFound): deployments.apps "traefik-crd" not found

    student-node ~ ✖ k describe deploy -n kube-system traefik |grep "Image:"
    traefik

    student-node ~ ✖ k describe deploy -n kube-system traefik |grep "Image:"
    traefik

    student-node ~ ✖ k describe deploy -n web-dashboard-03 web-dashboard-apd |grep "Image:"
        Image:         kodekloud/webapp-color:v3

    
    student-node ~ ➜  helm uninstall -n security-alpha-01 security-alpha-apd
    release "security-alpha-apd" uninstalled


09. <>

    student-node ~ ➜  k get deploy -n alpha-ns-apd 
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    cube-alpha-apd   5/5     5            5           16s
    ruby-alpha-apd   5/5     5            5           16s

    student-node ~ ➜  k describe -n alpha-ns-apd deploy cube-alpha-apd |grep Replicas
    Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
    Available      True    MinimumReplicasAvailable

    student-node ~ ➜  k describe -n alpha-ns-apd deploy ruby-alpha-apd |grep Replicas
    Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
    Available      True    MinimumReplicasAvailable

    student-node ~ ➜  k edit -n alpha-ns-apd deploy cube-alpha-apd 
    deployment.apps/cube-alpha-apd edited

    student-node ~ ➜  k edit -n alpha-ns-apd deploy ruby-alpha-apd 
    deployment.apps/ruby-alpha-apd edited

    student-node ~ ➜  k describe -n alpha-ns-apd deploy ruby-alpha-apd |grep Replicas
    Replicas:               7 desired | 7 updated | 7 total | 7 available | 0 unavailable
    Available      True    MinimumReplicasAvailable

    student-node ~ ➜  k describe -n alpha-ns-apd deploy cube-alpha-apd |grep Replicas
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    Available      True    MinimumReplicasAvailable

    student-node ~ ➜  k get pod -n alpha-ns-apd 
    NAME                              READY   STATUS    RESTARTS   AGE
    cube-alpha-apd-7fb4bbcf5c-4fp9j   1/1     Running   0          106s
    ruby-alpha-apd-9dbcd485-bt6nr     1/1     Running   0          106s
    cube-alpha-apd-7fb4bbcf5c-6tfl5   1/1     Running   0          106s
    ruby-alpha-apd-9dbcd485-jb42k     1/1     Running   0          106s
    cube-alpha-apd-7fb4bbcf5c-snhn4   1/1     Running   0          106s
    ruby-alpha-apd-9dbcd485-99cpc     1/1     Running   0          106s
    ruby-alpha-apd-9dbcd485-w5gdk     1/1     Running   0          106s
    ruby-alpha-apd-9dbcd485-mtpqd     1/1     Running   0          106s
    ruby-alpha-apd-9dbcd485-ns86q     1/1     Running   0          24s
    ruby-alpha-apd-9dbcd485-qs8qd     1/1     Running   0          24s


10. (錯題) **雖然修正了Ingress Controller 的問題，但是無法保證ingress controller 是否重啟成功!! 所以建議將舊的ingress controller 刪除再re deploy**
    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  k get deploy -n green-space 
    NAME              READY   UP-TO-DATE   AVAILABLE   AGE
    app-video         1/1     1            1           12s
    app-wear          1/1     1            1           12s
    default-backend   1/1     1            1           12s

    student-node ~ ➜  k describe depoly -n kube-
    kube-node-lease  kube-public      kube-system      

    student-node ~ ➜  k describe depoly -n kube-
    kube-node-lease  kube-public      kube-system      

    student-node ~ ➜  k describe depoly -n kube-system 
    error: the server doesn't have a resource type "depoly"

    student-node ~ ✖ k get deploy -n ingress-nginx ingress-nginx-controller
    NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
    ingress-nginx-controller   0/1     1            0           77s

    student-node ~ ➜  k get pod -n ingress-nginx
    NAME                                       READY   STATUS             RESTARTS     AGE
    ingress-nginx-admission-create-8l7g8       0/1     Completed          0            112s
    ingress-nginx-admission-patch-ghx8x        0/1     Completed          1            112s
    ingress-nginx-controller-86f88bb76-2dqd8   0/1     CrashLoopBackOff   4 (4s ago)   112s

    student-node ~ ➜  k logs -n ingress-nginx ingress-nginx-controller-86f88bb76-2dqd8 
    /-------------------------------------------------------------------------------
    NGINX Ingress controller
    Release:       v1.1.2
    Build:         bab0fbab0c1a7c3641bd379f27857113d574d904
    Repository:    https://github.com/kubernetes/ingress-nginx
    nginx version: nginx/1.19.9

    /-------------------------------------------------------------------------------

    W0227 16:21:27.093967      58 client_config.go:615] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
    I0227 16:21:27.094082      58 main.go:223] "Creating API client" host="https://172.20.0.1:443"
    I0227 16:21:27.099113      58 main.go:267] "Running in Kubernetes cluster" major="1" minor="29" git="v1.29.0" state="clean" commit="3f7a50f38688eb332e2a1b013678c6435d539ae6" platform="linux/amd64"
    F0227 16:21:27.101169      58 main.go:83] No service with name default-backend-service found in namespace default: services "default-backend-service" not found

    student-node ~ ➜  k edit deploy -n ingress-nginx ingress-nginx-controller

    spec:
      containers:
      - args:
        - /nginx-ingress-controller
        - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
        - --election-id=ingress-controller-leader
        - --watch-ingress-without-class=true
        - --default-backend-service=~~default/default-backend-service~~ # 改成:  /default-backend-service
        - --controller-class=k8s.io/ingress-nginx
        - --ingress-class=nginx
        - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
        - --validating-webhook=:8443
        - --validating-webhook-certificate=/usr/local/certificates/cert
        - --validating-webhook-key=/usr/local/certificates/key

    
    student-node ~ ➜  k get svc -n green-space 
    NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    app-video-service         ClusterIP   172.20.142.205   <none>        8080/TCP   7m26s
    app-wear-service          ClusterIP   172.20.221.203   <none>        8080/TCP   7m26s
    default-backend-service   ClusterIP   172.20.95.68     <none>        80/TCP     7m26s

    student-node ~ ➜  k get deploy -n green-space 
    NAME              READY   UP-TO-DATE   AVAILABLE   AGE
    app-video         1/1     1            1           7m48s
    app-wear          1/1     1            1           7m48s
    default-backend   1/1     1            1           7m48s

    student-node ~ ➜  k get deploy -n ingress-nginx 
    NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
    ingress-nginx-controller   1/1     1            1           7m59s

    student-node ~ ➜  k get ingress -n green-space 
    NAME                   CLASS   HOSTS   ADDRESS          PORTS   AGE
    ingress-resource-uxz   nginx   *       172.20.235.122   80      9m55s

    student-node ~ ➜  k edit ingress -n green-space ingress-resource-uxz 

    spec:
    ingressClassName: nginx
    rules:
    - http:
        paths:
        - backend:
            service:
                name: app-wear-service
                port:
                number: 8080
            path: /app-wear #/app1
            pathType: Prefix
        - backend:
            service:
                name: app-video-service
                port:
                number: 8080
            path: /app-video #/app2
            pathType: Prefix
    
    ingress.networking.k8s.io/ingress-resource-uxz edited


11. student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  k get deploy -n global-space 
    NAME              READY   UP-TO-DATE   AVAILABLE   AGE
    default-backend   1/1     1            1           34s
    webapp-food       1/1     1            1           34s

    student-node ~ ✖ k get svc -n global-space 
    NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    default-backend-service   ClusterIP   172.20.81.255   <none>        80/TCP     109s
    food-service              ClusterIP   172.20.106.17   <none>        8080/TCP   109s

    student-node ~ ➜  vim ingress-resource-xnz.yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: ingress-resource-xnz
    **namespace: global-space**
    annotations:
        **nginx.ingress.kubernetes.io/rewrite-target: /**
        **nginx.ingress.kubernetes.io/ssl-redirect: "false"**
    spec:
        **ingressClassName: ninx**
    rules:
    - http:
        paths:
        - path: **/eat**
            pathType: Prefix
            backend:
            service:
                name: **food-service**
                port:
                number: **8080**

    student-node ~ ➜  kc ingress-resource-xnz.yaml
    ingress.networking.k8s.io/ingress-resource-xnz created

    student-node ~ ➜  k get ingress -n global-space 
    NAME                   CLASS   HOSTS   ADDRESS   PORTS   AGE
    ingress-resource-xnz   ninx    *                 80      4s


12. (這題忘了寫，Attemp 1是對的，可省略)


13. (錯題) **port 跟 targetport題目如沒有指定，建議一律設置為80/8080這兩個 以符合規範!!**
    student-node ~ ➜  kubectl config use-context cluster2
    Switched to context "cluster2".

    student-node ~ ➜  k create deploy ckad13-deployment --replicas=2 --image=nginx
    deployment.apps/ckad13-deployment created

    student-node ~ ➜  k get deploy
    NAME                READY   UP-TO-DATE   AVAILABLE   AGE
    ckad13-deployment   2/2     2            2           4s

    student-node ~ ➜  kd expose deploy ckad13-deployment --port=31080 --target-port=31080 --name=ckad13-service -o yaml > ckad13-deployment.yaml

    student-node ~ ➜  vim ckad13-deployment.yaml 

    student-node ~ ➜  kc ckad13-deployment.yaml 
    service/ckad13-service created

    student-node ~ ➜  k get svc 
    NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
    kubernetes       ClusterIP   10.43.0.1       <none>        443/TCP           75m
    ckad13-service   NodePort    10.43.203.148   <none>        31080:31080/TCP   5s

    student-node ~ ➜  k describe svc ckad13-service 
    Name:                     ckad13-service
    Namespace:                default
    Labels:                   app=ckad13-deployment
    Annotations:              <none>
    Selector:                 app=ckad13-deployment
    Type:                     NodePort
    IP Family Policy:         SingleStack
    IP Families:              IPv4
    IP:                       10.43.203.148
    IPs:                      10.43.203.148
    Port:                     <unset>  31080/TCP
    TargetPort:               31080/TCP
    NodePort:                 <unset>  31080/TCP
    Endpoints:                10.42.0.15:31080,10.42.1.13:31080 # 有endpoint 且對應到pod的 IP則表示創建成功
    Session Affinity:         None
    External Traffic Policy:  Cluster
    Events:                   <none>

    
    student-node ~ ➜  k get pod -o wide
    NAME                                 READY   STATUS    RESTARTS   AGE     IP           NODE                    NOMINATED NODE   READINESS GATES
    ckad13-deployment-6457d497cd-lfx77   1/1     Running   0          5m29s   10.42.0.15   cluster2-controlplane   <none>           <none>
    ckad13-deployment-6457d497cd-qkl2l   1/1     Running   0          5m29s   10.42.1.13   cluster2-node01         <none>           <none>

    你在 ckad13-service 中的配置如下：
    port: 31080
    targetPort: 31080
    nodePort: 31080
    但這樣的設定可能不符合 KodeKloud 的預期，因為：

    port 是 Service 內部的端口，通常不應設為 nodePort 的值

    port 指的是 Service 內部 ClusterIP 服務的端口，通常是 80、443 這類的應用端口，而不是 NodePort 的範圍 (30000-32767)。
    一般建議設為 80 或 8080，這樣符合常見規範。
    targetPort 需要對應 Pod 內部的應用端口

    targetPort 是指向後端 Pod 的應用端口，如果 nginx 容器內部預設監聽 80，那麼這裡應該設為 80，而不是 31080，除非你明確修改了 Pod 內部的 nginx 配置。
    nodePort 只能用於 NodePort 或 LoadBalancer 類型的 Service

    nodePort 應該在 30000-32767 之間，這部分你設定正確，但 port 不應該設為 31080，應該是 80 或 8080。

14. student-node ~ ➜  k create clusterrole healthz-access --verb=get,post --non-resource-url=/healthz,/healthz/*
    clusterrole.rbac.authorization.k8s.io/healthz-access created

    student-node ~ ➜  k get clusterrole |grep health
    healthz-access                                                         2025-02-27T16:46:55Z

    student-node ~ ➜  k create clusterrolebinding healthz-access-binding --clusterrole=healthz-access --user=healthz-user
    clusterrolebinding.rbac.authorization.k8s.io/healthz-access-binding created


    student-node ~ ➜  k get clusterrolebindings.rbac.authorization.k8s.io |grep healthz
    healthz-access-binding                                   ClusterRole/healthz-access                                                         12s

17. student-node ~ ➜  ka Foo.yaml
    The CustomResourceDefinition "foos.stable.example.com" is invalid: 
    * metadata.name: Invalid value: "foos.stable.example.com": must be spec.names.plural+"."+spec.group
    * metadata.annotations[api-approved.kubernetes.io]: Required value: protected groups must have approval annotation "api-approved.kubernetes.io", see https://github.com/kubernetes/enhancements/pull/1111

    student-node ~ ✖ vim Foo.yaml

    student-node ~ ➜  vim Foo.yaml

    student-node ~ ➜  cat Foo.yaml
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
    name: **foos.samplecontroller.k8s.io** #foos.stable.example.com # **name 必須要是: plural.API Group**  
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
            # enables the status subresource
            status: {}

    student-node ~ ➜  ka Foo.yaml
    The CustomResourceDefinition "foos.samplecontroller.k8s.io" is invalid: metadata.annotations[api-approved.kubernetes.io]: Required value: protected groups must have approval annotation "api-approved.kubernetes.io", see https://github.com/kubernetes/enhancements/pull/1111


    student-node ~ ➜  k api-resources |grep Foo
    foos                                           samplecontroller.k8s.io/v1alpha1   true         Foo

20. student-node ~ ✖ k top pod
    NAME                             CPU(cores)   MEMORY(bytes)   
    foo-controller-7b6956d98-9p9vx   2m           6Mi             

    student-node ~ ➜  **k top pods**
    NAME                             CPU(cores)   MEMORY(bytes)   
    foo-controller-7b6956d98-9p9vx   2m           6Mi             
    hulk                             102m         250Mi           
    ironman                          14m          30Mi            
    kodekloud-logs-aom               1m           0Mi             
    thor                             1m           16Mi            

    student-node ~ ✖ k top pods |grep -E "hulk|ironman|thor"
    hulk                             108m         161Mi           
    ironman                          15m          30Mi            
    thor                             1m           16Mi 

    student-node ~ ➜  k top pods  --s|grep -E "hulk|ironman|thor"
    --selector  (Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2). Matchi…)
    --server    (The address and port of the Kubernetes API server)
    --sort-by   (If non-empty, sort pods list using specified field. The field can be either 'cpu' or 'memory'.)
    --sum       (Print the sum of the resource usage)

    student-node ~ ➜  k top pods  --sort-by=memory |grep -E "hulk|ironman|thor"
    hulk                             107m         250Mi           
    ironman                          14m          30Mi            
    thor                             1m           16Mi   

    student-node ~ ➜  k get pod hulk -o custom-columns=Podname:metadata.name,"Memory Limit":spec.containers.resources.limits.m
    emory
    Podname   Memory Limit
    hulk      <none>

    student-node ~ ➜  k get pod hulk -o custom-columns=Podname:metadata.name,"Memory Limit":spec.conta^Cers.resources.limits.m
    emory

    student-node ~ ✖ kubectl get pod hulk -o custom-columns="Podname:metadata.name,Memory Limit:.spec.**containers[*]**.resources.limits.memory"
    Podname   Memory Limit
    hulk      400Mi

    student-node ~ ➜  kubectl get pod hulk -o custom-columns="Podname:metadata.name,Memory Limit:spec.containers[*].resources.
    limits.memory"
    Podname   Memory Limit
    hulk      400Mi


    <補充>
    這個 spec.containers.resources.limits.memory 可能不適用於多個 container 的 pod，因為 spec.containers 是一個 列表 (array)，你需要明確選擇某個 container，或者使用 jsonpath。

    改用 jsonpath 來確保輸出：
    kubectl get pod hulk -o jsonpath="{.spec.containers[*].resources.limits.memory}"
    或者：
    kubectl get pod hulk -o custom-columns="Podname:metadata.name,Memory Limit:.spec.containers[*].resources.limits.memory"

    student-node ~ ➜  kubectl get pod hulk -o custom-columns="Podname:metadata.name,Memory Limit:spec.containers[*].resources.limits.memory" > /root/pod-metrics

    student-node ~ ➜  cat /root/pod-metrics
    Podname   Memory Limit
    hulk      400Mi
