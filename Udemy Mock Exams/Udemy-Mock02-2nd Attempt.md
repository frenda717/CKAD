01. 
    student-node ~ ➜  k describe pod -n ckad-multi-containers ckad-sidecar-pod 
    Name:             ckad-sidecar-pod
    Namespace:        ckad-multi-containers
    Priority:         0
    Service Account:  default
    Node:             cluster1-node02/192.168.36.200
    Start Time:       Mon, 24 Feb 2025 18:40:39 +0000
    Labels:           run=ckad-sidecar-pod
    Annotations:      <none>
    Status:           Running
    IP:               10.42.2.3
    IPs:
    IP:  10.42.2.3
    Containers:
    main-container:
        Container ID:  containerd://4e6ecdc85a6385e7bf353405a5d2774531ee6fb519c1a1b9812a82118a0da4f1
        Image:         nginx:1.16
        Image ID:      docker.io/library/nginx@sha256:d20aa6d1cae56fd17cd458f4807e0de462caf2336f0b70b5eeb69fcaaf30dd9c
        Port:          <none>
        Host Port:     <none>
        Args:
        /bin/sh
        -c
        i=0; while true; do echo "$i: $(date) Hi I am from Sidecar container" > index.html ; i=$((i+1)); sleep 5; done
        State:          Running
        Started:      Mon, 24 Feb 2025 18:40:42 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /usr/share/nginx/html from my-vol (rw)
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ccgwt (ro)
    sidecar-container:
        Container ID:   containerd://588aaab5f885bfd6f7e10e51bda87131e8372a00b7f0bbb8e002ffb6621c658f
        Image:          busybox:1.28
        Image ID:       docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
        Port:           <none>
        Host Port:      <none>
        State:          Waiting
        Reason:       CrashLoopBackOff
        Last State:     Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Mon, 24 Feb 2025 20:33:37 +0000
        Finished:     Mon, 24 Feb 2025 20:33:37 +0000
        Ready:          False
        Restart Count:  27
        Environment:    <none>
        Mounts:
        /var/log from my-vol (rw)
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ccgwt (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       False 
    ContainersReady             False 
    PodScheduled                True 
    Volumes:
    my-vol:
        Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
        Medium:     
        SizeLimit:  <unset>
    kube-api-access-ccgwt:
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
    Warning  BackOff  36s (x532 over 115m)  kubelet  Back-off restarting failed container sidecar-container in pod ckad-sidecar-pod_ckad-multi-containers(c4447c5a-af15-4818-aa28-ed3a93c267bf)

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
        name: main-container
        resources: {}
        args: [/bin/sh, -c,
                'i=0; while true; do echo "$i: $(date) Hi I am from Sidecar container" > index.html ; i=$((i+1)); sleep 5; done']
        volumeMounts:
        - name: my-vol
            mountPath: /usr/share/nginx/html
    - image: busybox:1.28
        name: sidecar-container
        resources: {}
        args:
        - /bin/sh
        - -c
        - "while true; do sleep 10; done" # 新增此行，仍顯示CrashLoopBackOff
        volumeMounts:
        - name: my-vol
            mountPath: /var/log
    volumes:
    - name: my-vol
        emptyDir: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}


    student-node ~ ✖ k replace -f  ckad-sidecar-pod.yaml  --force
        pod "ckad-sidecar-pod" deleted
        pod/ckad-sidecar-pod replaced

    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                 READY   STATUS             RESTARTS      AGE
    dos-containers-pod   1/2     CrashLoopBackOff   5 (32s ago)   3m22s
    ckad-sidecar-pod     2/2     Running            0             6s

02. student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                 READY   STATUS             RESTARTS      AGE
    ckad-sidecar-pod     1/2     CrashLoopBackOff   7 (26s ago)   11m
    dos-containers-pod   1/2     CrashLoopBackOff   2 (23s ago)   40s

    student-node ~ ➜  cat dos-containers-pod.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: dos-containers-pod
    name: dos-containers-pod
    namespace: ckad-multi-containers
    spec:
    containers:
    - image: nginx:1.17
        name: alpha
        resources: {}
        env:
        - name: ROLE
        value: SERVER
    - image: busybox:1.28
        name: beta
        command: ["/bin/sh","-c",echo "Hello multi-containers", "sleep 40000"]
        resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}


    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME                 READY   STATUS             RESTARTS        AGE
    ckad-sidecar-pod     1/2     CrashLoopBackOff   24 (4m5s ago)   101m
    dos-containers-pod   1/2     CrashLoopBackOff   22 (3m8s ago)   90m

    student-node ~ ➜  k describe pod -n ckad-multi-containers dos-containers-pod 
    Name:             dos-containers-pod
    Namespace:        ckad-multi-containers
    Priority:         0
    Service Account:  default
    Node:             cluster1-node01/192.168.78.182
    Start Time:       Mon, 24 Feb 2025 18:51:25 +0000
    Labels:           run=dos-containers-pod
    Annotations:      <none>
    Status:           Running
    IP:               10.42.1.3
    IPs:
    IP:  10.42.1.3
    Containers:
    alpha:
        Container ID:   containerd://14dd4b2a3c7e9fc1217e4f008b66a5c55155d3586c37d9d00baeff26d4d52268
        Image:          nginx:1.17
        Image ID:       docker.io/library/nginx@sha256:6fff55753e3b34e36e24e37039ee9eae1fe38a6420d8ae16ef37c92d1eb26699
        Port:           <none>
        Host Port:      <none>
        State:          Running
        Started:      Mon, 24 Feb 2025 18:51:28 +0000
        Ready:          True
        Restart Count:  0
        Environment:
        ROLE:  SERVER
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qx6kt (ro)
    beta:
        Container ID:  containerd://bd1ae733aa159e0fcbfc516f34d6228fa36fc5f7589f8e043ae598e91a7a5850
        Image:         busybox:1.28
        Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
        Port:          <none>
        Host Port:     <none>
        Command:
        /bin/sh
        -c
        echo "Hello multi-containers"
        sleep 40000
        State:          Waiting
        Reason:       CrashLoopBackOff
        Last State:     Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Mon, 24 Feb 2025 20:19:11 +0000
        Finished:     Mon, 24 Feb 2025 20:19:11 +0000
        Ready:          False
        Restart Count:  22
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qx6kt (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       False 
    ContainersReady             False 
    PodScheduled                True 
    Volumes:
    kube-api-access-qx6kt:
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
    Type     Reason   Age                  From     Message
    ----     ------   ----                 ----     -------
    Warning  BackOff  62s (x415 over 91m)  kubelet  Back-off restarting failed container beta in pod dos-containers-pod_ckad-multi-containers(2835398e-09c5-4c61-bdb5-15634e959f11)

    Pod dos-containers-pod 出現 CrashLoopBackOff，但 kubectl logs 沒有內容，這是因為：

    beta 容器的 exit code = 0（正常結束）
    beta 容器的 State: Waiting (CrashLoopBackOff)，這意味著它一直重新啟動

    問題點
    beta 容器的 exit code 是 0，表示它正常執行完畢，並非錯誤崩潰。
    但 Kubernetes 會重新啟動它，因為 Pod 的 restartPolicy 預設是 Always，導致它持續進入 CrashLoopBackOff。

    beta 容器的 Command：
    /bin/sh -c "echo 'Hello multi-containers'; sleep 40000"
    這個指令的執行順序：
    echo 'Hello multi-containers' → 馬上執行，然後結束
    sleep 40000 → 應該執行，但實際上容器結束了
    可能 sleep 40000 不是在 sh 的背景執行，因此容器執行完 echo 之後馬上退出，導致 Kubernetes 認為它異常結束，不斷重啟。

    方案 1：讓 sleep 持續執行
    目前 sleep 40000 可能因為 sh -c 機制導致它沒有正確運行，你可以改成：
    beta:
        Command:
        - /bin/sh
        - -c
        - "echo 'Hello multi-containers'; exec sleep 40000"
    方案 2：設置 restartPolicy: Never
    如果 beta 容器只是要執行一次並退出，你可以修改 Pod 設定：
    restartPolicy: Never
    並重新部署：

    sh
    Copy
    Edit
    kubectl delete pod dos-containers-pod -n ckad-multi-containers
    kubectl apply -f dos-containers-pod.yaml




09.  建立pod 並沒有設置containerPort, 用k expose 新增service之後，有需要再回到pod添加container port嗎   (**不用!!**)
    Create a pod with name pod21-ckad-svcn using the nginx:alpine image in the default namespace and also expose the pod using service pod21-ckad-svcn on port 80.
    Note: Use the imperative command for above scenario.

4.  CrashLoopBackOff:

    student-node ~ ➜  cat ckad15-memory.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: ckad15-memory
    name: ckad15-memory
    spec:
    containers:
    - image: polinux/stress
        name: ckad15-memory
        command: ["/bin/sh","-c","stress"]
        args: ["--vm", "1", "--vm-bytes", "10M", "--vm-hang", "1"]
        resources:
        requests:
            memory: "10Mi"
        limits:
            memory: "10Mi"
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    student-node ~ ➜  k logs ckad15-memory 
    `stress' imposes certain types of compute stress on your system

    Usage: stress [OPTION [ARG]] ...
    -?, --help         show this help statement
        --version      show version statement
    -v, --verbose      be verbose
    -q, --quiet        be quiet
    -n, --dry-run      show what would have been done
    -t, --timeout N    timeout after N seconds
        --backoff N    wait factor of N microseconds before work starts
    -c, --cpu N        spawn N workers spinning on sqrt()
    -i, --io N         spawn N workers spinning on sync()
    -m, --vm N         spawn N workers spinning on malloc()/free()
        --vm-bytes B   malloc B bytes per vm worker (default is 256MB)
        --vm-stride B  touch a byte every B bytes (default is 4096)
        --vm-hang N    sleep N secs before free (default none, 0 is inf)
        --vm-keep      redirty memory instead of freeing and reallocating
    -d, --hdd N        spawn N workers spinning on write()/unlink()
        --hdd-bytes B  write B bytes per hdd worker (default is 1GB)

    Example: stress --cpu 8 --io 4 --vm 2 --vm-bytes 128M --timeout 10s

    Note: Numbers may be suffixed with s,m,h,d,y (time) or B,K,M,G (size).


    student-node ~ ➜  k describe pod ckad15-memory 
    Name:             ckad15-memory
    Namespace:        default
    Priority:         0
    Service Account:  default
    Node:             cluster2-controlplane/192.168.231.161
    Start Time:       Mon, 24 Feb 2025 20:01:08 +0000
    Labels:           run=ckad15-memory
    Annotations:      <none>
    Status:           Running
    IP:               10.42.0.13
    IPs:
    IP:  10.42.0.13
    Containers:
    ckad15-memory:
        Container ID:  containerd://e70c786b33351f0760c3956dda1c3af4cdfd635624f087d7dffd9b0f16e1e175
        Image:         polinux/stress
        Image ID:      docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa
        Port:          <none>
        Host Port:     <none>
        Command:
        /bin/sh
        -c
        stress
        Args:
        --vm
        1
        --vm-bytes
        10M
        --vm-hang
        1
        State:          Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Mon, 24 Feb 2025 20:01:25 +0000
        Finished:     Mon, 24 Feb 2025 20:01:25 +0000
        Last State:     Terminated
        **Reason:       Completed**
        Exit Code:    0
        Started:      Mon, 24 Feb 2025 20:01:10 +0000
        Finished:     Mon, 24 Feb 2025 20:01:10 +0000
        Ready:          False
        Restart Count:  2
        Limits:
        memory:  10Mi
        Requests:
        memory:     10Mi
        Environment:  <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-x59k8 (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       False 
    ContainersReady             False 
    PodScheduled                True 
    Volumes:
    kube-api-access-x59k8:
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
    Type     Reason     Age               From               Message
    ----     ------     ----              ----               -------
    Normal   Scheduled  18s               default-scheduler  Successfully assigned default/ckad15-memory to cluster2-controlplane
    Normal   Pulled     17s               kubelet            Successfully pulled image "polinux/stress" in 692ms (692ms including waiting)
    Normal   Pulled     17s               kubelet            Successfully pulled image "polinux/stress" in 151ms (151ms including waiting)
    Normal   Pulling    3s (x3 over 18s)  kubelet            Pulling image "polinux/stress"
    Normal   Pulled     3s                kubelet            Successfully pulled image "polinux/stress" in 155ms (155ms including waiting)
    Normal   Created    3s (x3 over 17s)  kubelet            Created container ckad15-memory
    Normal   Started    2s (x3 over 17s)  kubelet            Started container ckad15-memory
    Warning  BackOff    2s (x3 over 16s)  kubelet            Back-off restarting failed container ckad15-memory in pod ckad15-memory_default(ce8aa62b-10bb-4828-82f5-d510f60715dd)

    student-node ~ ➜  vim ckad15-memory.yaml 

    student-node ~ ➜  k exec -it ckad15-memory -c ckad15-memory -- stress --vm 1 --vm-bytes 8M --vm-hang 1
    error: unable to upgrade connection: container not found ("ckad15-memory")

    Pod 已經 CrashLoopBackOff，無法執行 exec

    見Udemy-Mock03.md 第04題，同樣解法。



07. Solution
    Run the following command to change the context: -

    kubectl config use-context cluster1



    In this task, we will use the helm commands. Here are the steps: -


    Add the repostiory to Helm with the following command: -

    helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/



    The helm repo add command is used to add a new chart repository to Helm and this allows us to browse and install charts from the new repository using the Helm package manager.



    Use the helm repo ls command which is used to list all the currently configured Helm chart repositories on Kubernetes cluster.

    kubectl create ns cd-tool-apd

    helm upgrade --install kubernetes-dashboard-server kubernetes-dashboard/kubernetes-dashboard -n cd-tool-apd



    Before installing the chart, we have to create a namespace as given in the task description. Then we can install the kubernetes-dashboard chart on a Kubernetes cluster.

    kubectl create ns cd-tool-apd



10. 
    port, targetPort, nodePort都設置錯了

    student-node ~ ➜  k get svc -n app-ckad 
    NAME                 TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
    frontend-ckad-svcn   NodePort   172.20.216.105   <none>        31100:31518/TCP   96m

    應為:
    A service frontend-ckad-svcn to expose the frontend pods outside the cluster on port 31100.
    apiVersion: v1
    kind: Service
    metadata:
    name: frontend-ckad-svcn
    namespace: app-ckad
    spec:
    selector:
        app: frontend
    type: NodePort
    ports:
    - name: http
        port: 80
        targetPort: 80
        nodePort: 31100


    backend-ckad-svcn 我沒有設置到:
    A service backend-ckad-svcn to make backend pods to be accessible within the cluster.
    apiVersion: v1
    kind: Service
    metadata:
    name: backend-ckad-svcn
    namespace: app-ckad
    spec:
    selector:
        app: backend
    ports:
    - name: http
        port: 80
        targetPort: 80

    (這次我的networkpolicy對了)
    A policy database-ckad-netpol to limit access of database pods only to backend pods.
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
    name: database-ckad-netpol
    namespace: app-ckad
    spec:
    podSelector:
        matchLabels:
        app: database
    ingress:
    - from:
        - podSelector:
            matchLabels:
            app: backend


11. (設置成ClusterIP, 但題目要求NodePort)On student-node, use the command: kubectl create deployment  hr-web-app-ckad05-svcn --image=kodekloud/webapp-color --replicas=2

    Now we can run the command: kubectl expose deployment hr-web-app-ckad05-svcn --type=NodePort --port=8080 --name=hr-web-app-service-ckad05-svcn --dry-run=client -o yaml > hr-web-app-service-ckad05-svcn.yaml to generate a service definition file.

    student-node ~ ➜  k get svc 
    NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
    frontend-ckad-svcn               NodePort    172.20.7.240    <none>        31100:31135/TCP   96m
    hr-web-app-service-ckad05-svcn   **ClusterIP**   172.20.39.15    <none>        30082/TCP         83m
    kubernetes                       ClusterIP   172.20.0.1      <none>        443/TCP           147m
    pod21-ckad-svcn                  ClusterIP   172.20.44.129   <none>        80/TCP            99m

    Now, in generated service definition file add the nodePort field with the given port number under the ports section and create a service.




12.  沒有設置deployment!!!

    student-node ~ ➜  k get deploy -n nginx-deployment 
    No resources found in nginx-deployment namespace.

    kubectl apply -f - <<eof
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: nginx-ckad11
    namespace: nginx-deployment
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: nginx-ckad
    template:
        metadata:
        labels:
            app: nginx-ckad
        spec:
        containers:
            - name: nginx
            image: nginx
            ports:
                - containerPort: 80 # 這個忘了設置





20. A manifest file located at root/ckad-aom.yaml on cluster3-controlplane. Which can be used to create a multi-containers pod. There are issues with the manifest file, preventing resource creation. Identify the errors, fix them and create resource.

    You can access controlplane by ssh cluster3-controlplane if required.

    apiVersion: v1
    kind: Pod
    metadata:
    name: ckad-aom
    spec:
    containers:
        - name: nginx
        image: nginx
        volumeMounts:
            - name: flash-logs
            mountPath: ~~/usr/src~~ # 應改為: /var/log/nginx
        - name: busybox
        image: busybox
        command:
            - /bin/sh
            - -c
            - sleep 10000
        volumeMounts:
            - name: flash-logs
            mountPath: /usr/src
    volumes:
        - name: flash-logs
        emptyDir: {}
    ~                 