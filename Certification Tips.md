Certification Tip

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

1. Setting alias:

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


    Tips tutor gave:
    1) https://www.linkedin.com/pulse/my-ckad-exam-experience-atharva-chauthaiwale/

    2) https://medium.com/@harioverhere/ckad-certified-kubernetes-application-developer-my-journey-3afb0901014

    3) https://github.com/lucassha/CKAD-resources


