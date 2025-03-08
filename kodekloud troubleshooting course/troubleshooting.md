### Prerequisites
1. k get events
    ![get events](images/debug/get-events.png "get events")
    借由k get events指令檢查是否有Warning的events, 並且查看warning reason

    Q: You received reports of recent issues in the cluster. Investigate by using the appropriate command to view the events happening in the cluster. What is the most recent event?

    option1: RegisterdNode
    option2: Failed
    option3: NodeAllocatebleEnforced
    option4: InvalidDiskCapacity

    controlplane ~ ➜  k get events
    LAST SEEN   TYPE      REASON                    OBJECT              MESSAGE
    56m         Normal    NodeHasSufficientMemory   node/controlplane   Node controlplane status is now: NodeHasSufficientMemory
    56m         Normal    NodeHasNoDiskPressure     node/controlplane   Node controlplane status is now: NodeHasNoDiskPressure
    56m         Normal    NodeHasSufficientPID      node/controlplane   Node controlplane status is now: NodeHasSufficientPID
    56m         Normal    NodeAllocatableEnforced   node/controlplane   Updated Node Allocatable limit across pods
    56m         Normal    Starting                  node/controlplane   Starting kubelet.
    ...
    33s (x4 over 114s)   Warning   Failed                    Pod/webserver       Error: ErrImagePull
    8s (x6 over 113s)    Normal    BackOff                   Pod/webserver       Back-off pulling image "busybox1"
    8s (x6 over 113s)    Warning   Failed                    Pod/webserver       Error: ImagePullBackOff
    ...

    OR:
    controlplane ~ ➜  k get events --sort-by=.metadata.creationTimestamp| tail -n 1
    2m22s       Warning   Failed      pod/webserver   Error: ImagePullBackOff

    OR:
    controlplane ~ ➜  **k get events | tail -n 1**
    2m31s       Warning   Failed      pod/webserver   Error: ImagePullBackOff

    Answer: option2 Failed

2. k port-forward


3. k auth can-i
    ![auth whoami](images/debug/auth-whoami.png "auth whoami")

    由於未指定當前的user為何，使用auth can-i 卻顯示yes, 則透過k auth whoami指令查看當下測試的user

    ![auth sa](images/debug/auth-serviceaccount.png "auth sa")
    --as 指定serviceaccount
    k auth can-i get pods **--as=system:serviceaccount:default:default**


4. k top: (僅限於用在pods跟nodes)
   ![k top](images/debug/top.png "k top")

5. k expain --recursive: 加上recursive參數，直接展開所有字段(這就不用k explain一路查找至最底層字段)
    ![k explain with recursive](images/debug/explain.png "k explain with recursive")

    You just learned there’s a lifecycle spec defined in one of the pod manifests. Use the right command to learn more about that lifecycle spec. Write the output out to a file lifecycle.txt
    
    Hint: lifecycle spec is under containers

    controlplane ~ ➜  kubectl explain pod.spec.containers.lifecycle > lifecycle.txt

    controlplane ~ ➜  cat lifecycle.txt 
    KIND:       Pod
    VERSION:    v1

    FIELD: lifecycle <Lifecycle>


    DESCRIPTION:
    ...

    controlplane ~ ➜  kubectl explain pod.spec.containers.lifecycle > lifecycle.txt

6. k diff: 檢查本地現有的yaml配置文件與已建立的Resource之間的差異
    此命令會比較 my-deployment.yaml 文件中的資源和集群中的現有資源，並輸出差異
    ![k diff](images/debug/diff.png "k diff")

    You've made changes to the deployment configuration in the nginx-deployment.yml file, but you want to review the differences before applying them. Use the command to show the difference between local and cluster configurations. What has changed?

    option1: replicas
    option2: labels

    controlplane ~ ➜  k diff -f nginx-deployment.yml 
    diff -u -N /tmp/LIVE-1547354458/apps.v1.Deployment.app.nginx-deployment /tmp/MERGED-679631904/apps.v1.Deployment.app.nginx-deployment
    --- /tmp/LIVE-1547354458/apps.v1.Deployment.app.nginx-deployment        2025-03-08 05:22:15.080460346 +0000
    +++ /tmp/MERGED-679631904/apps.v1.Deployment.app.nginx-deployment       2025-03-08 05:22:15.080460346 +0000
    @@ -6,7 +6,7 @@
        kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-deployment","namespace":"app"},"spec":{"replicas":3,"selector":{"matchLabels":{"app":"nginx"}},"template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx","ports":[{"containerPort":80}]}]}}}}
    creationTimestamp: "2025-03-08T05:21:24Z"
    -  generation: 1
    +  generation: 2
    labels:
        app: nginx
    name: nginx-deployment
    @@ -15,7 +15,7 @@
    uid: e17ef8e6-3d54-476d-a15f-cb7de79a4ce3
    spec:
    progressDeadlineSeconds: 600
    -  replicas: 3
    +  replicas: 2
    revisionHistoryLimit: 10
    selector:
        matchLabels:

    Answer: option1

7. How many pods are one the node01?

    controlplane ~ ➜  k get nodes
    NAME           STATUS   ROLES           AGE   VERSION
    controlplane   Ready    control-plane   69m   v1.32.0
    node01         Ready    <none>          68m   v1.32.0

    controlplane ~ ➜  ssh node01
    Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.4.0-1106-gcp x86_64)

    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/pro

    This system has been minimized by removing packages and content that are
    not required on a system that users do not log into.

    To restore this content, you can run the 'unminimize' command.

    node01 ~ ➜  k get pods -A
    E0308 05:24:04.427912   19038 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
    E0308 05:24:04.430511   19038 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
    E0308 05:24:04.432887   19038 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
    E0308 05:24:04.435062   19038 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
    E0308 05:24:04.438207   19038 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
    Error from server (NotFound): the server could not find the requested resource

    改執行:
    node01 ~ ✖ exit
    logout
    Connection to node01 closed.

    controlplane ~ ➜  k get pod -A **-o wide --field-selector spec.nodeName=node01**
    NAMESPACE     NAME                                READY   STATUS             RESTARTS   AGE     IP             NODE     NOMINATED NODE   READINESS GATES
    app           myapp                               0/1     Completed          0          9m40s   10.244.192.4   node01   <none>           <none>
    app           nginx-deployment-647677fc66-4mxzz   1/1     Running            0          5m41s   10.244.192.4   node01   <none>           <none>
    app           nginx-deployment-647677fc66-k4dwd   1/1     Running            0          5m41s   10.244.192.5   node01   <none>           <none>
    app           nginx-deployment-647677fc66-zgtt9   1/1     Running            0          5m41s   10.244.192.6   node01   <none>           <none>
    app           python-app                          1/1     Running            0          9m41s   10.244.192.3   node01   <none>           <none>
    default       backend                             1/1     Running            0          9m41s   10.244.192.2   node01   <none>           <none>
    default       webserver                           0/1     ImagePullBackOff   0          18m     10.244.192.1   node01   <none>           <none>
    kube-system   kube-proxy-vkdxd                    1/1     Running            0          71m     192.2.122.3    node01   <none>           <none>
    kube-system   weave-net-hbjxz                     2/2     Running            0          71m     192.2.122.3    node01   <none>           <none>

8. You are investigating an incident that happened on the pod called myapp and need to ensure logs display timestamps for accurate analysis. Write the logs to a file timestamps.txt


    Deployment can be in any of the namespaces

    controlplane ~ ➜  k logs -n app myapp --timestamps > timestamps.txt

    controlplane ~ ➜  cat timestamps.txt 
    2025-03-08T05:17:32.679262501Z map[id:2 message:message 2]
    2025-03-08T05:17:32.779410834Z map[id:1 message:message 1]
    2025-03-08T05:17:32.879673388Z map[id:2 message:message 2]
    2025-03-08T05:17:32.979954690Z map[id:1 message:message 1]
    2025-03-08T05:17:33.079151695Z map[id:3 message:message 3]
    2025-03-08T05:17:33.179393468Z map[id:1 message:message 1]
    2025-03-08T05:17:33.279638912Z map[id:3 message:message 3]
    2025-03-08T05:17:33.379843720Z map[id:2 message:message 2]
    2025-03-08T05:17:33.479028037Z map[id:2 message:message 2]
    2025-03-08T05:17:33.579274593Z map[id:1 message:message 1]
    2025-03-08T05:17:33.679566556Z map[id:2 message:message 2]
    2025-03-08T05:17:33.779797509Z map[id:3 message:message 3]
    2025-03-08T05:17:33.880039911Z map[id:3 message:message 3]
    2025-03-08T05:17:33.979322062Z map[id:1 message:message 1]
    2025-03-08T05:17:34.079547541Z map[id:3 message:message 3]

9. 使用k debug替代 k exec，原因為:   (**k exec 適用於已運行的 Pod; k debug 主要用於問題排查**)
    A. 減少 Pod 中斷 (minimize pod disruptions)
    kubectl exec 直接在現有的 Pod 內執行命令，當你進行調試時，如果操作不慎（例如刪除文件、修改設定或影響運行的進程），可能會導致 Pod 崩潰，影響業務運行。
    kubectl debug 則可以創建一個新的臨時 Debug 容器，不影響原本的應用。例如：
        
        kubectl debug pod/my-pod -it --image=busybox --target=my-container
    
    這樣會在原有的 Pod 內部新增一個 busybox 容器，但不會影響 my-container 的運行，確保應用不中斷。
    調試完畢後，可以刪除 Debug 容器，而不影響原本的 Pod。
    ✅ 適用場景

    需要深入分析但不想影響應用運行的情境，如調試異常流量、記錄分析、測試指令等。

    B. Distroless 映像 (distroless images)
    什麼是 Distroless?

    **Distroless 是一種極簡化的容器映像**，它不包含 bash、sh、apt 或其他常見的工具，僅包含應用所需的最少組件，這有助於提高安全性和減少攻擊面。
    例如，官方的 gcr.io/distroless/base 或 gcr.io/distroless/static 映像沒有 shell，不能直接 kubectl exec -it my-pod -- sh 進入容器。
    為何 kubectl debug 更好？

    kubectl exec 失效： 在 Distroless 容器中，由於沒有 shell，執行以下指令可能會失敗：
    
        kubectl exec -it my-pod -- sh
    
    會返回類似 exec /bin/sh: no such file or directory 的錯誤。
    kubectl debug 提供完整工具： 你可以用 kubectl debug 添加一個帶有 bash 或 sh 的調試容器，例如：
        
        kubectl debug pod/my-pod -it --image=ubuntu
    
    這樣即使原本的 Pod 是 Distroless，調試容器仍然有完整的 Linux 環境，可用來診斷問題。
    ✅ 適用場景

    當 Pod 使用 Distroless 或極簡化映像時，kubectl exec 可能無法執行，但 kubectl debug 可幫助你添加一個完整的調試環境。


    ![k debug](images/debug/debug.png "k debug")
    ![k debug](images/debug/debug02.png "k debug")


    1️⃣ kubectl exec 適用於已運行的 Pod
    kubectl exec 用於執行命令到 正在運行中的容器，這適用於：

    需要在容器內執行某個指令，如 /bin/bash、sh、ls、cat /var/log/...。
    需要存取應用程式內部的日誌或環境，而不需要額外新增 Debug 容器。
    
    範例：
    kubectl exec -it webapp -- /bin/bash
    這行指令的目的是嘗試進入 webapp Pod 內的容器，執行 /bin/bash 指令。

    2️⃣ kubectl debug 主要用於問題排查
    kubectl debug 的用途：

    用於當 kubectl exec 不能解決問題時，例如：
    容器缺少 shell（這題的錯誤就是 /bin/bash 找不到）。
    **容器崩潰 (CrashLoopBackOff)，導致 kubectl exec 無法使用**。
    容器影像是 distroless，缺少 debug 工具，如 curl、ps、netstat。
    想要使用 Ephemeral Container 來附加額外的偵錯工具。
    
    如果這題允許使用 kubectl debug，可以用：
    kubectl debug -it webapp --image=busybox --target=webapp
    
    這會附加一個新的 ephemeral debug container，並且包含 shell，讓我們能夠進行進一步的偵錯。



    Which of the following actions can be performed using kubectl debug? Choose all valid answers:

    A. Running a new pod in the node’s host namespaces with access to its filesystem.
    B. Attaching a container to a running pod without restarting the pod.
    C. Copy files from a local machine to a pod.
    D. Create a new pod with a modified container image for debugging.

    正確答案是 **A、B 和 D**，以下是各選項的解釋，以及為什麼 **C 選項不正確**：


    /### **A) 在節點的主機命名空間 (host namespaces) 中運行一個新的 Pod，並能訪問其文件系統。✅**
    - **kubectl debug** 可用來在 **節點的主機命名空間** 中啟動一個新的偵錯 Pod，這對於排查節點級別的問題非常有用。
    - 可以使用 `--hostpid`、`--hostnetwork` 和 `--hostipc` 來讓偵錯容器運行於主機命名空間。
    - **示例**：
    ```sh
    kubectl debug node/<node-name> -it --image=busybox --hostpid --hostnetwork
    ```


    /### **B) 在不重啟 Pod 的情況下，附加一個容器到正在運行的 Pod。✅**
    - **kubectl debug** 允許在 **不重新啟動 Pod** 的情況下，**新增一個臨時的偵錯容器**。
    - 這在原本的容器中缺少一些偵錯工具時特別有用。
    - **示例**：
    ```sh
    kubectl debug -it <pod-name> --image=busybox --target=<container-name>
    ```
    - `--target=<container-name>` 參數讓偵錯容器運行於相同的網路與命名空間。

    /### **C) 從本地機器複製文件到 Pod。❌（錯誤）**
    - **kubectl debug 無法用來在本地機器與 Pod 之間傳輸文件**。
    - 這項操作應該使用 **kubectl cp** 命令來完成。
    - **正確的文件複製方式**：
    ```sh
    kubectl cp /local/path my-pod:/container/path
    ```
    - `kubectl debug` 的用途是 **偵錯與診斷 Pod**，而不是文件傳輸，因此 `kubectl cp` 才是正確的選擇。


    /### **D) 創建一個帶有修改後容器映像 (image) 的新 Pod 來進行偵錯。✅**
    - **kubectl debug** 允許基於原本的 Pod **創建一個新 Pod，並使用不同的鏡像來偵錯**。
    - 這對於需要使用帶有診斷工具的映像來排查問題時特別有用。
    - **示例**：
    ```sh
    kubectl debug --image=busybox --copy-to=my-debug-pod my-existing-pod
    ```
    - `--copy-to` 參數讓新的 Pod 基於原 Pod 創建，但允許修改，例如更換容器映像。


    Answer: A,D



10. Pod 必須處於 Running 狀態，才能使用 Ephemeral containers!
    
    When are ephemeral(短暫的) containers useful?

    A) Container is starting up
    B) When using distroless images
    C) Debugging a pending pod
    D) Images doesn’t contain debugging utilities
    E) Debugging container in a crashloop

    什麼是 Ephemeral Containers（短暫容器）？
    Ephemeral containers 是 Kubernetes 允許用來 暫時 附加到一個已經運行的 Pod 內的特殊類型容器，主要用於 偵錯。
    特點：
    **不能 定義 resources（CPU、內存限制）**。
    **不能 參與 PodSpec，即不會影響 Pod 正常運行的容器配置**。
    **不會自動重啟，當它結束時就會被刪除**。

    A) Container is starting up（容器正在啟動）❌
    錯誤，因為 Ephemeral containers **只能附加到已運行的 Pod**，無法影響啟動過程。
    如果容器尚未啟動完成，Ephemeral containers 也無法插入。

    B) When using distroless images（使用 Distroless 映像）✅
    正確，因為 distroless 映像通常不包含 shell 或調試工具，無法直接執行 bash 或 sh 來偵錯。
    使用 ephemeral containers，可以 **動態附加 一個帶有 sh 或 bash 的調試容器，如 busybox 或 debug-tools，來進行診斷**。
    
    示例：
    kubectl debug -it mypod --image=busybox --target=mycontainer

    C) Debugging a pending pod（偵錯處於 pending 狀態的 Pod）❌
    錯誤，**因為 Pod 必須處於 Running 狀態，才能使用 Ephemeral containers**。
    Pending 狀態的 Pod 可能還沒有被指派 (scheduled) 到節點，甚至還沒有拉取映像，這時候無法附加任何容器。
    
    正確的方法：
    使用 kubectl describe pod <pod-name> 檢查為何 Pod 處於 Pending 狀態（如資源不足、調度失敗）。
    使用 kubectl get events 來查看相關事件。

    D) Images doesn’t contain debugging utilities（映像不包含偵錯工具）✅
    正確，這與 B 選項相似，許多生產環境的 Docker 映像 為了安全性，不會包含 curl、vi、ps 等工具。
    **Ephemeral containers 允許動態加入一個包含這些工具的容器，以便進行診斷和偵錯**。
    
    示例：
    kubectl debug -it mypod --image=busybox --target=mycontainer


    E) Debugging container in a crashloop（偵錯陷入 CrashLoop 的容器）✅
    正確，**因為 CrashLoopBackOff 狀態的容器會不斷崩潰並重啟，導致難以進行標準偵錯**。
    **Ephemeral containers 允許附加到 Pod 而不影響原本的容器重啟行為，提供一個獨立的診斷環境**。

    示例：
    kubectl debug -it mypod --image=busybox --target=mycontainer
    這樣可以在容器崩潰時，使用 ps, top, cat /var/log/... 來進一步調查問題。
    
    Answer: B,~~C~~,E 是 **B,D,E**!


    Which of the following are **incorrect statements**? Choose all valid answers

    A) The ephemeral container is guaranteed to not eat up the pod’s resources. (X)
    B) Ephemeral containers can be restarted after exiting. (X)
    C) The --copy-to flag creates a pod that is not owned by the original workload
    D) The pid namespace of all containers in a pod is shared with the ephemeral container by default. (X 除非設置--target!)

    C) 為何 C 是正確的？
    --copy-to 旗標 (kubectl debug --copy-to) 確實會創建一個新的 Pod，且**這個 Pod 不屬於原本的 workload**（例如 Deployment、StatefulSet）。
    這個新的 Pod 不會受原始控制器（如 Deployment 或 ReplicaSet）管理，它只是基於原本的 Pod 創建一個複製版本，並允許修改鏡像、環境變數等設定。
    
    示例：
    kubectl debug mypod --copy-to=mypod-debug --image=busybox
    這樣會創建一個 mypod-debug，但它 不會 屬於原本的 Deployment，因此 Kubernetes 不會自動縮放或調整這個新的 Pod。

    D) The pid namespace of all containers in a pod is shared with the ephemeral container by default. ❌（錯誤 ✅）
    錯誤原因：

    預設情況下，Ephemeral Containers 並不會與 Pod 內的其他容器共享 PID namespace。
    如果要讓 Ephemeral Container 存取其他容器的進程（process），需要手動指定 --target。
    
    示例：
    kubectl debug -it mypod --image=busybox --target=mycontainer
    --target=mycontainer 會讓 Ephemeral Container 共享 mycontainer 的 PID namespace，否則它會有自己的獨立 namespace。
    
    為何這是錯誤的敘述？
    **Kubernetes 設計上不會讓所有容器自動共享 PID namespace，避免影響安全性**。
    **預設情況下，Ephemeral Containers 只能存取自己的進程，如果沒有 --target，則看不到 Pod 內其他容器的進程**。


    Answer: A,B,D 


    Your application webapp is running in the default namespace. It seems to be encountering some issues. WITHOUT the use of a debug container, attempt to find the cause of the issue. What error are you running into?

    option1: Error: command not found: kubectl exec 
    option2: Pod is not running or ready
    option3: OCI runtime exec failed: exec failed: unable to start container process: exec: "/bin/bash" : stat /bin/bash: no such file or directory

    Hint: run k exec command

    controlplane ~ ✖ k exec -it webapp -- /bin/bash
    error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "6aed24fc83383c463df4fe462c4400050bc95731b1305b4a305c2023adc935a3": OCI runtime exec failed: exec failed: unable to start container process: exec: "/bin/bash": stat /bin/bash: no such file or directory: unknown


    Answer: option3

    

    見第9. **k exec 適用於已運行的 Pod; k debug 主要用於問題排查**

    1️⃣ kubectl exec 適用於已運行的 Pod
    kubectl exec 用於執行命令到 正在運行中的容器，這適用於：
    **需要在容器內執行某個指令，如 /bin/bash**、sh、ls、cat /var/log/...。
    需要存取應用程式內部的日誌或環境，而不需要額外新增 Debug 容器。
    
    範例：
    kubectl exec -it webapp -- /bin/bash
    這行指令的目的是嘗試進入 webapp Pod 內的容器，執行 /bin/bash 指令。

    2️⃣ kubectl debug 主要用於問題排查
    kubectl debug 的用途：

    用於當 kubectl exec 不能解決問題時，例如：
    容器缺少 shell（這題的錯誤就是 /bin/bash 找不到）。
    **容器崩潰 (CrashLoopBackOff)，導致 kubectl exec 無法使用**。
    容器影像是 distroless，缺少 debug 工具，如 curl、ps、netstat。
    想要使用 Ephemeral Container 來附加額外的偵錯工具。
    
    如果這題允許使用 kubectl debug，可以用：
    kubectl debug -it webapp --image=busybox --target=webapp
    
    這會附加一個新的 ephemeral debug container，並且包含 shell，讓我們能夠進行進一步的偵錯。


    Add a debugging container to the ephemeral pod, use the busybox image. What’s the first message you see ?

    option1: failed to generate container "5f7b9313beae753..."
    optpion2: Targeting container "ephermeral". If you don't see process from this conatiner it may be because the conainer runtime doesn't support this feature

    controlplane ~ ➜  k debug ephermeral --image=busybox --target ephermeral
    Targeting container "ephermeral". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
    --profile=legacy is deprecated and will be removed in the future. It is recommended to explicitly specify a profile, for example "--profile=general".
    Error from server (NotFound): pods "ephermeral" not found

    為何 kubectl debug 指令需要 --target？
    kubectl debug 預設會創建一個新的 ephemeral container，但如果我們希望這個容器附加到特定的目標容器，就必須使用 --target 參數。

    主要原因
    確保 Ephemeral Container 能夠共享目標容器的 Namespace

    這樣可以存取該容器的 processes、network、filesystem。
    若沒有 --target，則 Kubernetes 不會知道要附加到哪個容器，尤其是 Pod 內部有多個容器時。
    解決 ps, top, netstat 等指令看不到其他進程的問題

    如果 --target 未指定，Ephemeral Container 可能無法看到原本容器的進程，因為它會啟動一個獨立的 PID 命名空間。
    有 --target 時，Ephemeral Container 會共享 目標容器的 PID namespace，讓 ps aux、top 可以顯示原本的進程。
    使用 --target 可以避免誤附加到錯誤的容器

    在某些情況下，Pod 內部可能包含多個應用程式容器，不指定 --target 可能會附加到錯誤的容器。
    --target ephermeral 確保我們的 busybox Debug 容器附加到 名為 "ephermeral" 的容器，而不是其他不相關的容器。

    Targeting container "ephermeral". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
    
    這個訊息的意思是：
    Ephemeral Container 會嘗試共享 "ephermeral" 容器的 PID Namespace。
    但是，有些容器運行時（Container Runtime，例如 Docker 或 containerd）可能不支援此功能。
    如果 ps aux 或 top 指令執行後 看不到原本容器的進程，可能是因為：
    Runtime 不支援 PID Namespace 共享。
    --target 容器內沒有運行中的進程。

    Answer: option2

    
    Use a debug container to inspect backend, what is the linux version used?

    controlplane ~ ✖ k get pod
    NAME                       READY   STATUS    RESTARTS      AGE
    backend-7ddfdc4fdc-vxvkc   2/2     Running   1 (15m ago)   32m
    ephemeral                  1/1     Running   0             10m
    webapp                     1/1     Running   0             32m


    First Try:
    controlplane ~ ➜  k debug pod/backend --target backend
    error: you must specify --image when not using --copy-to.
    (kubectl debug 預設不會自動選擇 debug 容器的映像，所以我們必須手動指定 --image，否則 Kubernetes 無法創建 Ephemeral Container。)


    Second Try:
    controlplane ~ ✖ k debug pod/backend --target backend --image=busybox
    Targeting container "backend". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
    --profile=legacy is deprecated and will be removed in the future. It is recommended to explicitly specify a profile, for example "--profile=general".
    Error from server (NotFound): pods "backend" not found

    --target backend 指定的 backend 容器不存在。
    可能的原因：
    backend-7ddfdc4fdc-vxvkc Pod 內的容器名稱可能不是 backend，而是其他名稱（如 app 或 main）。
    
    可以用以下指令檢查 Pod 內的容器名稱：
    kubectl get pod backend-7ddfdc4fdc-vxvkc -o jsonpath='{.spec.containers[*].name}'

    若 backend 容器名稱錯誤，應該使用正確的容器名稱：
    k debug backend-7ddfdc4fdc-vxvkc --target <正確的容器名稱> --image=busybox


    Thrid Try:
    controlplane ~ ✖ k debug pod/backend-7ddfdc4fdc-vxvkc --target backend --image=busybox
    Targeting container "backend". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
    --profile=legacy is deprecated and will be removed in the future. It is recommended to explicitly specify a profile, for example "--profile=general".
    Defaulting debug container name to debugger-4q7kw.
    The Pod "backend-7ddfdc4fdc-vxvkc" is invalid: spec.ephemeralContainers[0].targetContainerName: Not found: "backend"

    (此處的target要跟pod name一樣，應為: backend-7ddfdc4fdc-vxvkc )

    Forth Try: 成功
    controlplane ~ ✖ k debug backend-7ddfdc4fdc-vxvkc -it -c debugger --image=busybox
    --profile=legacy is deprecated and will be removed in the future. It is recommended to explicitly specify a profile, for example "--profile=general".
    If you don't see a command prompt, try pressing enter.
    / # **uname -a**
    Linux backend-7ddfdc4fdc-vxvkc 5.4.0-1106-gcp #115~18.04.1-Ubuntu SMP Mon May 22 20:46:39 UTC 2023 x86_64 GNU/Linux
    / # 


    Answer: Linux verion 5.4.0-1106-gcp

    <補充>: 為何我重新進入到debugg container 他直接退出了?? 
    controlplane ~ ➜  kubectl debug -it -c debugger --image=busybox backend-7ddfdc4fdc-vxvkc
    --profile=legacy is deprecated and will be removed in the future. It is recommended to explicitly specify a profile, for example "--profile=general".
    / # uname -a
    Linux backend-7ddfdc4fdc-vxvkc 5.4.0-1106-gcp #115~18.04.1-Ubuntu SMP Mon May 22 20:46:39 UTC 2023 x86_64 GNU/Linux
    / # exit

    controlplane ~ ➜  

    **為何重新進入 Debug 容器後，它直接退出？**

    當你使用 `kubectl debug` 來啟動一個 **Ephemeral Container**，它的 **行為與普通的容器不同**。以下是導致容器退出的原因：

    **🔍 原因 1: Ephemeral Container 在退出後不會自動重啟**
    - **Ephemeral Containers 是一次性的 (one-time use)**，當你執行 `exit` 時，它會直接終止並且 **不會重新啟動**。
    - Kubernetes **不會管理 Ephemeral Container 的生命週期**，它不像普通的 Pod 內部的容器可以自動重啟。

    - **證據：**
    ```
    Session ended, the ephemeral container will not be restarted but may be reattached using 'kubectl attach backend-7ddfdc4fdc-vxvkc -c debugger -i -t' if it is still running
    ```
    - **關鍵字**: `the ephemeral container will not be restarted`
    - 這明確表示 **Ephemeral 容器不會自動重新啟動**，但如果它仍然在運行，則可以用 `kubectl attach` 重新連接。

    
    **🔍 原因 2: `kubectl debug` 會每次都創建一個新的 Ephemeral Container**
    - **你執行 `kubectl debug` 時，Kubernetes 會重新創建一個新的 Ephemeral Container**。
    - **但如果上次的 Debug 容器已經退出，它不會被重新啟動**，新的 Ephemeral Container 也會立即終止，因為它 **沒有任何進程在運行**。

    **🔹 如何證明 Kubernetes 創建了一個新 Debug 容器？**
    你可以使用以下命令查看 Ephemeral Containers：
    ```sh
    kubectl get pod backend-7ddfdc4fdc-vxvkc -o jsonpath='{.spec.ephemeralContainers[*].name}'
    ```
    如果每次執行 `kubectl debug`，這個列表會不斷新增不同的 `debugger-*` 容器名稱。

    **🔍 為何新的 Debug 容器立即退出？**
    - 當 Kubernetes 創建一個新的 **Ephemeral Debug 容器**，如果沒有給定持續執行的命令，它會**立即退出**。
    - 預設情況下，`busybox` 容器的 `ENTRYPOINT` 是 `sh`，但如果沒有 `-it` 或 `sleep`，它會**直接結束**。

    **🔹 如何測試？**
    執行：
    ```sh
    kubectl debug backend-7ddfdc4fdc-vxvkc -it -c debugger --image=busybox -- sh
    ```
    如果 `sh` 被關閉，Ephemeral 容器就會退出。

    **✅ 解決方案**
    /### **方法 1: 使用 `kubectl attach` 重新連接已運行的 Debug 容器**
    如果 Debug 容器仍然運行，你可以使用：
    ```sh
    kubectl attach backend-7ddfdc4fdc-vxvkc -c debugger -i -t
    ```
    這樣就可以重新進入之前的 Ephemeral 容器，而不會創建新的。

    
    /### **方法 2: 讓 Debug 容器保持存活**
    如果你想讓 Debug 容器 **不會立即退出**，你可以讓它執行 `sleep`：
    ```sh
    kubectl debug backend-7ddfdc4fdc-vxvkc -it -c debugger --image=busybox -- sh -c "sleep 3600"
    ```
    這樣，它會運行 `sleep 3600`，並且持續存活 1 小時，你可以多次用 `kubectl attach` 進入它。

    /### **方法 3: 使用 `kubectl run` 來啟動獨立的 Debug Pod**
    如果你需要一個長時間運行的 Debug 容器，而不是 Ephemeral Container，你可以這樣做：
    ```sh
    kubectl run debug-container --rm -it --image=busybox -- /bin/sh
    ```
    這樣，它會創建一個新的 **獨立 Pod**，但當你 `exit` 時，它會被刪除。


    注意!! Kubernetes 不允許修改已經創建的 Ephemeral Container，所以 kubectl debug 不能變更 debugger 容器的 Command 或 Args。
    
    你之前已經創建了 debugger 容器，現在嘗試修改它，因此被拒絕。
    如何解決？
    ✅ 方案 1：刪除 Pod 並重新創建 Debug 容器（如果可以刪除）
    ✅ 方案 2：使用不同名稱的 Ephemeral 容器，例如 debugger2
    ✅ 方案 3：如果 Debug 容器還在運行，使用 kubectl attach 或 kubectl exec 進入它

    倘若不能刪除pod的話，則採用方案2，新建立一個debugger2容器，然後設置sleep 使其能運行久一點，接著k exec進入該容器

    controlplane ~ ✖ kubectl debug backend-7ddfdc4fdc-vxvkc -it -c debugger2 --image=busybox -- s
    h -c "sleep 3600"
    --profile=legacy is deprecated and will be removed in the future. It is recommended to explicitly specify a profile, for example "--profile=general".
    If you don't see a command prompt, try pressing enter.

    ^C



    ^C^C^C^C

    (新開視窗，使用k exec 便可進入新建立的容器)

    controlplane ~ ➜  kubectl exec -it backend-7ddfdc4fdc-vxvkc -c debugger2 -- sh
    / # 



    What happens when you exit the shell from the debug container?
    option1: The debug container is terminated
    option2: The pod restarts

    Answer: option1

11. k9s CLI 
    
    command line 輸入: k9s進入k9s介面
    ![k9S CLI](images/debug/k9s.png "k9S CLI")

    k9s visulization:
    ![k9S visualization](images/debug/k9s-pulse "k9S Visualization")

### Troubleshooting Scenario
1. Imageg Pull Errors


2. Crashing Pods


3. Pending Pods 


4. Missing Pods


5. Deployment


6. Create Container Errors


7. Config Out of Date


8. Reloader


9. Endlessly Terminating Pods


10. Field Immutability


11. Enable Service Links


12. RBAC Troubleshooting


13. Netowrk Troubleshooting


14. Ingress Troubleshooting


15. Storage Troubleshoting
