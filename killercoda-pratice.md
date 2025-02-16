### VIM setup

controlplane $ ls -a ~
.   .ICEauthority  .bash_history  .cache   .dbus   .kube   .profile  .theia  .wget-hsts  snap
..  .Xauthority    .bashrc        .config  .gnupg  .local  .ssh      .vnc    filesystem


vim ~/.vimrc

set expandtab
set tabstop=2
set shiftwidth=2

<!-- expandtab: use spaces for tab
tabstop: amount of spaces used for tab
shiftwidth: amount of spaces used during indentation -->

controlplane $ ls -a ~     
.   .ICEauthority  .bash_history  .cache   .dbus   .kube   .profile  .theia    **.vimrc**  .wget-hsts  snap
..  .Xauthority    .bashrc        .config  .gnupg  .local  .ssh      .viminfo  .vnc    filesystem


### SSH basics
During the exam you'll be provided with a command you need to run before every question to connect to a dedicated host

Create a new empty file /root/node01 on host node01

controlplane $ ssh node01
Last login: Sun Nov 13 17:27:09 2022 from 10.48.0.33
node01 $ pwd  
/root
node01 $ touch node01
node01 $ ls
filesystem  node01  snap

Now get back to the original host and create new empty file /root/controlplane there.
node01 $ exit
logout
Connection to node01 closed.
controlplane $ pwd
/root
controlplane $ touch controlplane
controlplane $ ls
controlplane  filesystem  snap

### Kubectl Contexts

A kubectl context contains connection information to a Kubernetes cluster. Different kubectl contexts can connect to different Kubernetes clusters, or to the same cluster but using different users or different default namespaces.

List all available kubectl contexts and write the output to /root/contexts .

controlplane $ k config -h
Modify kubeconfig files using subcommands like "kubectl config set current-context my-context".

 The loading order follows these rules:

  1.  If the --kubeconfig flag is set, then only that file is loaded. The flag may only be set once and no merging takes
place.
  2.  If $KUBECONFIG environment variable is set, then it is used as a list of paths (normal path delimiting rules for
your system). These paths are merged. When a value is modified, it is modified in the file that defines the stanza. When
a value is created, it is created in the first file that exists. If no files in the chain exist, then it creates the
last file in the list.
  3.  Otherwise, ${HOME}/.kube/config is used and no merging takes place.

Available Commands:
  current-context   Display the current-context
  delete-cluster    Delete the specified cluster from the kubeconfig
  delete-context    Delete the specified context from the kubeconfig
  delete-user       Delete the specified user from the kubeconfig
  get-clusters      Display clusters defined in the kubeconfig
  get-contexts      Describe one or many contexts
  get-users         Display users defined in the kubeconfig
  rename-context    Rename a context from the kubeconfig file
  set               Set an individual value in a kubeconfig file
  set-cluster       Set a cluster entry in kubeconfig
  set-context       Set a context entry in kubeconfig
  set-credentials   Set a user entry in kubeconfig
  unset             Unset an individual value in a kubeconfig file
  use-context       Set the current-context in a kubeconfig file
  view              Display merged kubeconfig settings or a specified kubeconfig file

Usage:
  kubectl config SUBCOMMAND [options]

Use "kubectl config <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
controlplane $ k config get-contexts > contexts
controlplane $ vim contexts 
controlplane $ cat contexts 
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   
          purple                        kubernetes   kubernetes-admin   purple
          yellow                        kubernetes   kubernetes-admin   yellow

CURRENT：標記當前使用的 context，* 表示當前啟用的 context。
NAME：context 的名稱。
CLUSTER：該 context 連接的 Kubernetes 叢集名稱。
AUTHINFO：使用的認證資訊（身份驗證方式）。
NAMESPACE：該 context 預設的命名空間（Namespace）。

We see three contexts all pointing to the same cluster.

Context kubernetes-admin@kubernetes will connect to the default Namespace.
Context purple will connect to the purple Namespace.
Context yellow will connect to the yellow Namespace.

controlplane $ k config current-context                             
kubernetes-admin@kubernetes

controlplane $ k get pods -n purple
NAME         READY   STATUS    RESTARTS   AGE
purple-pod   1/1     Running   0          8m5s

controlplane $ k get pods -n yellow
NAME         READY   STATUS    RESTARTS   AGE
yellow-pod   1/1     Running   0          8m12s

controlplane $ **k config use-context purple** (使用k config use-context <ns> 指令，可以直接切換到指定的namespace, CKAD考試時便可以不用頻繁輸入: -n <ns> 。這可以節省時間) 
Switched to context "purple".

controlplane $ k get pod
NAME         READY   STATUS    RESTARTS   AGE
purple-pod   1/1     Running   0          11m

controlplane $ k config use-context yellow
Switched to context "yellow".

controlplane $ k get pod
NAME         READY   STATUS    RESTARTS   AGE
yellow-pod   1/1     Running   0          11m

controlplane $ k config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".

controlplane $ k get pod
No resources found in default namespace.


### Create ConfigMaps

1. Create the ConfigMap stored in existing file /root/cm.yaml

    controlplane $ cat cm.yaml 
    apiVersion: v1
    data:
    tree: birke
    level: "3"
    department: park
    kind: ConfigMap
    metadata:
    name: birke

    controlplane $ k create -f /root/cm.yaml 
    configmap/birke created

    controlplane $ k delete cm birke --force
    Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
    configmap "birke" force deleted

    controlplane $ **k -f /root/cm.yaml create**
    configmap/birke created

2. env, envFrom 跟 valueFrom的分別:
    env（手動設定環境變數）
    env 允許你在 Pod 規格（PodSpec） 中，直接為 container 設定環境變數，類似於 export VAR=value。

    適用情境
    ✅ 適合 設定固定值的環境變數（例如 APP_ENV=production）。
    ✅ 適用於測試或開發環境，當變數值不會改變時很方便。
    ❌ 不適用於機密資訊（例如密碼、Token），因為值會暴露在 kubectl describe pod 中。

    envFrom（從 ConfigMap 或 Secret 載入）
    envFrom 可以 一次性導入 ConfigMap 或 Secret 中的所有鍵值對作為環境變數，無需逐個指定。

    ✅ 適合批量導入環境變數，比如應用配置 (ConfigMap) 或機密 (Secret)。
    ✅ 使 Pod YAML 更簡潔，因為不需要逐個寫 env。
    ❌ 無法控制單個環境變數的值（如果你只想用 ConfigMap 的部分變數，應該用 env 的 valueFrom）。

    valueFrom（從 ConfigMap、Secret 或其他 Kubernetes 資源動態獲取變數）
    valueFrom 允許你 單獨指定某個 Kubernetes 資源的值作為環境變數，可以來自：
    ConfigMap
    Secret
    Pod 字段（例如 Pod 的名稱、節點名稱）
    Downward API（例如 Pod 資源限制、標籤）

    適用情境
    ✅ 適合精細控制環境變數，只提取 特定的 ConfigMap 或 Secret 鍵值。
    ✅ 適合需要根據 Pod 自身屬性來設置環境變數的情境（例如 metadata.name）。
    ✅ 適合應用程式需要根據 Pod 規格變數來調整行為（如資源請求）。
    ❌ 比 envFrom 需要更明確指定每個變數，所以如果需要大量變數可能會比較冗長。

    ---

    ## **📌 總結**
    | 方式          | 特點 | 主要用途 | 優點 | 缺點 |
    |--------------|------|--------|------|------|
    | `env` | **手動設定固定值** | 直接定義環境變數 | 簡單易用，適合小範圍變數 | 需要手動輸入值，不能動態變更 |
    | `envFrom` | **批量導入 ConfigMap/Secret** | 載入整個 ConfigMap/Secret 作為變數 | 省去逐個寫 `env`，YAML 更乾淨 | 無法選擇性載入某些變數 |
    | `valueFrom` | **動態從 ConfigMap/Secret/Pod 獲取** | 需要指定特定鍵的值 | 可選擇單個鍵值，能動態獲取 Pod 資訊 | 需要逐個變數定義，比 `envFrom` 冗長 |

    ---

    ## **📌 哪種方式最適合？**
    ✅ **如**（如 `APP_ENV=production`）→ 使用 **`env`**  
    ✅ **如果需要一次性載果環境變數是固定值入整個 ConfigMap/Secret** → 使用 **`envFrom`**  
    ✅ **如果只需要 ConfigMap/Secret 中的部分變數** → 使用 **`valueFrom`**  
    ✅ **如果環境變數來自 Pod 本身**（如 `metadata.name`）→ 使用 **`valueFrom` + `fieldRef`**  

    這三種方式可以 **混合使用**，例如：
    ```yaml
    env:
    - name: LOG_LEVEL
    value: "debug"  # 手動設置
    - name: DB_HOST
    valueFrom:
        configMapKeyRef:
        name: my-configmap
        key: db_host  # 從 ConfigMap 取得
    envFrom:
    - secretRef:
        name: my-secret  # 批量導入 Secret
    ```
    這樣可以同時滿足 **固定值、特定鍵值、批量導入** 的需求。

    ---

    Create a Pod named pod1 of image nginx:alpine
    Make key tree of ConfigMap trauerweide available as environment variable TREE1
    Mount all keys of ConfigMap birke as volume. The files should be available under **/etc/birke/*** (陷阱!! 配置的時候不可將*號配置在path裡)
    Test env+volume access in the running Pod
    
    controlplane $ k run pod1 --image=nginx:alpine --dry-run=client -o yaml > pod1.yaml

    vim pod1.yaml

    WRONG solution:

    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: pod1
    name: pod1
    spec:
    containers:
    - image: nginx:alpine
        name: pod1
        resources: {}
        env:
        - name: TREE1
        valueFrom:
            configMapKeyRef:  # Make key tree of ConfigMap trauerweide available as environment variable TREE1
            name: trauerweide
            key: tree
        ~~envFrom:~~
        ~~- configMapRef:~~
            ~~name: vol-cm-01~~
        volumeMounts:
        - name: vol-cm-01    # Mount all keys of ConfigMap birke as volume. The files should be available under /etc/birke/*
            mountPath: ~~"/etc/birke/*"~~ /etc/birke
    volumes:
    - name: vol-cm-01
        configMap:
        name: birke
    ~~- name: vol-cm-02~~
        ~~configMap:~~
        ~~name: trauerweide~~
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    controlplane $ vim pod1.yaml 
    controlplane $ k create -f pod1.yaml 
    pod/pod1 created
    controlplane $ k get pod 
    NAME   READY   STATUS                       RESTARTS   AGE
    pod1   0/1     CreateContainerConfigError   0          3s

    
    僅設置volumeMounts，而不用設置envFrom的原因:
    需求：「Mount all keys of ConfigMap birke as volume. The files should be available under /etc/birke/*」

    **這個需求的重點是 將 ConfigMap birke 內的所有 key-value 對映為檔案**，也就是讓 ConfigMap 的內容作為檔案，而不是環境變數。因此：
    volumeMounts 是必須的 → 它負責將 ConfigMap 掛載為檔案系統。
    envFrom 不是必須的 → 它用於把 ConfigMap 的 key-value 當作環境變數，而這與題目需求無關。


    CORRECT solution:

    apiVersion: v1
    kind: Pod
    metadata:
    name: pod1
    spec:
    containers:
    - image: nginx:alpine
        name: pod1
        resources: {}
        env:
        - name: TREE1
        valueFrom:
            configMapKeyRef:
            name: trauerweide
            key: tree
        volumeMounts:
        - name: vol-cm-01
            mountPath: **/etc/birke/**
    volumes:
    - name: vol-cm-01
        configMap:
        name: birke

    controlplane $ k get pod
    NAME   READY   STATUS    RESTARTS   AGE
    pod1   1/1     Running   0          2s


    **VERIFY**:
    kubectl exec pod1 -- env | grep "TREE1=trauerweide"
    kubectl exec pod1 -- cat /etc/birke/tree
    kubectl exec pod1 -- cat /etc/birke/level
    kubectl exec pod1 -- cat /etc/birke/department

    controlplane $ k exec pod1 -- env |grep TREE1
    TREE1=trauerweide

    controlplane $ k exec pod1 -- ls /etc/birke
    department
    level
    tree


### Build and Run a Container
(此題僅幫助考生熟悉容器鏡像的構建過程，從而讓考生更容易理解如何將應用打包並部署到 Kubernetes)
Create a new file /root/Dockerfile to build a container image from. It should:

use bash as base
run ping killercoda.com
Build the image and tag it as pinger .

Run the image (create a container) named my-ping .

Solution:
Create the /root/Dockerfile :


FROM bash
CMD ["ping", "killercoda.com"]

Build the image:
(killer coda上使用podman 替代docker,因podman可以用於無root全縣環境)
podman build -t pinger .
podman image ls

Run the image:
podman run --name my-ping pinger

controlplane $ touch Dockerfile
controlplane $ vim Dockerfile 
controlplane $ cat Dockerfile 
FROM bash
CMD ["ping","killercoda.com"]

controlplane $ podman build -t pinger -f Dockerfile 
STEP 1/2: FROM bash
Resolving "bash" using unqualified-search registries (/etc/containers/registries.conf)
Trying to pull docker.io/library/bash:latest...
Getting image source signatures
Copying blob 0c953549fc9a done  
Copying blob f18232174bc9 done  
Copying blob c0cbbbdde11b done  
Copying config a6f5c002ec done  
Writing manifest to image destination
Storing signatures
STEP 2/2: CMD ["ping","killercoda.com"]
COMMIT pinger
--> 333cd7cd3e3
Successfully tagged localhost/pinger:latest
333cd7cd3e3fa2b823aa745b9a844df607193277f3f6d9713b8511bfc580db0a
controlplane $ podman image ls
REPOSITORY                  TAG         IMAGE ID      CREATED         SIZE
localhost/pinger            latest      333cd7cd3e3f  16 seconds ago  15 MB
docker.io/library/bash      latest      a6f5c002ec83  2 months ago    15 MB
docker.io/library/registry  2           26b2eb03618e  16 months ago   26 MB
controlplane $ podman run --name my-ping pinger
PING killercoda.com (104.26.14.111): 56 data bytes
64 bytes from 104.26.14.111: seq=0 ttl=42 time=12.142 ms
64 bytes from 104.26.14.111: seq=1 ttl=42 time=12.335 ms
64 bytes from 104.26.14.111: seq=2 ttl=42 time=11.947 ms
^C
--- killercoda.com ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 11.947/12.141/12.335 ms


Tag the image, which is currently tagged as pinger , also as local-registry:5000/pinger .
Then push the image into the local registry.


### Rollout Green-Blue
Application "wonderful" is running in default Namespace.
You can call the app using curl wonderful:30080 .
The application YAML is available at /wonderful/init.yaml .

The app has a Deployment with image httpd:alpine , but should be switched over to nginx:alpine .

The switch should happen instantly. Meaning that from the moment of rollout, all new requests should hit the new image.
    1. Create a new Deployment wonderful-v2 which uses image nginx:alpine with 4 replicas. It's Pods should have labels app: wonderful and version: v2
    2. Once all new Pods are running, change the selector label of Service wonderful to version: v2
    3. Finally scale down Deployment wonderful-v1 to 0 replicas

controlplane $ curl wonderful:30080
<html><body><h1>It works!</h1></body></html>

controlplane $ k get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
wonderful-v1   4/4     4            4           27s

controlplane $ k get pod
NAME                            READY   STATUS    RESTARTS   AGE
wonderful-v1-74cffc64c9-hxxkh   1/1     Running   0          29s
wonderful-v1-74cffc64c9-nqv94   1/1     Running   0          29s
wonderful-v1-74cffc64c9-tvwlw   1/1     Running   0          29s
wonderful-v1-74cffc64c9-x8rw7   1/1     Running   0          29s

controlplane $ k get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        4d5h
wonderful    NodePort    10.100.31.228   <none>        80:30080/TCP   32s

controlplane $ k create deploy wonderful-v2 --image=nginx:alpine --replicas=4 --dry-run=client -o yaml > wonderful-v2.yaml

controlplane $ vim wonderful-v2.yaml 
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    creationTimestamp: null
    labels:
        app: wonderful
        **version: v2** # Checked
    name: wonderful-v2
    spec:
    **replicas: 4** # Checked
    selector:
        matchLabels:
        app: wonderful
        **version: v2** # Checked
    strategy: {}
    template:
        metadata:
        creationTimestamp: null
        labels:
            app: wonderful
            **version: v2** # Checked
        spec:
        containers:
        - image: nginx:alpine # Checked
            name: nginx
            resources: {}
    status: {}
    ~           


controlplane $ k create -f wonderful-v2.yaml 
deployment.apps/wonderful-v2 created

controlplane $ k get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
wonderful-v1   4/4     4            4           5m42s
wonderful-v2   1/4     4            1           6s


controlplane $ k describe deploy wonderful-v2|grep -i "Labels"
Labels:                 app=wonderful
Labels:  app=wonderful-v2

controlplane $ k get pod
NAME                            READY   STATUS    RESTARTS   AGE
wonderful-v1-74cffc64c9-2qvfx   1/1     Running   0          8m56s
wonderful-v1-74cffc64c9-7w5bl   1/1     Running   0          8m56s
wonderful-v1-74cffc64c9-kzbzc   1/1     Running   0          8m56s
wonderful-v1-74cffc64c9-ztkgc   1/1     Running   0          8m56s
wonderful-v2-6d77ff75-8lfrl     1/1     Running   0          3m20s
wonderful-v2-6d77ff75-q2shl     1/1     Running   0          3m20s
wonderful-v2-6d77ff75-swm74     1/1     Running   0          3m20s
wonderful-v2-6d77ff75-whzhg     1/1     Running   0          3m20s

controlplane $ k edit service wonderful 
...
spec:
  clusterIP: 10.96.25.66
  clusterIPs:
  - 10.96.25.66
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 30080
    port: 80
    protocol: TCP
    targetPort: 80
  **selector**: 變更
    app: wonderful
    version: ~~v1~~ **v2**
  sessionAffinity: None
  type: NodePort

service/wonderful edited

controlplane $ k scale deploy wonderful-v1 --replicas=0
deployment.apps/wonderful-v1 scaled

controlplane $ k get pod
NAME                            READY   STATUS        RESTARTS   AGE
wonderful-v1-74cffc64c9-7w5bl   0/1     Completed     0          13m
wonderful-v1-74cffc64c9-ztkgc   1/1     Terminating   0          13m
wonderful-v2-6d77ff75-8lfrl     1/1     Running       0          7m33s
wonderful-v2-6d77ff75-q2shl     1/1     Running       0          7m33s
wonderful-v2-6d77ff75-swm74     1/1     Running       0          7m33s
wonderful-v2-6d77ff75-whzhg     1/1     Running       0          7m33s

controlplane $ k get pod
NAME                          READY   STATUS    RESTARTS   AGE
wonderful-v2-6d77ff75-8lfrl   1/1     Running   0          7m45s
wonderful-v2-6d77ff75-q2shl   1/1     Running   0          7m45s
wonderful-v2-6d77ff75-swm74   1/1     Running   0          7m45s
wonderful-v2-6d77ff75-whzhg   1/1     Running   0          7m45s


Verify:

controlplane $ curl wonderful:30080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>


### Rollout Canary
Application "wonderful" is running in default Namespace.
You can call the app using curl wonderful:30080 .
The application YAML is available at /wonderful/init.yaml .

The app has a Deployment with image httpd:alpine , but should be switched over to nginx:alpine .

The switch should not happen fast or automatically, but using the Canary approach:

20% of requests should hit the new image
80% of requests should hit the old image
For this create a new Deployment wonderful-v2 which uses image nginx:alpine .

The total amount of Pods of both Deployments combined should be 10.

controlplane $ k edit deploy wonderful-v1 
deployment.apps/wonderful-v1 edited

spec:
  progressDeadlineSeconds: 600
  **replicas: 8** # Changed from 10 to 8
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: wonderful
  strategy:
  ...

controlplane $ k create deploy wonderful-v2 --image=nginx:alpine --dry-run=client -o yaml > wonderful-v2.yaml
controlplane $ vim wonderful-v2.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: wonderful
  name: wonderful-v2
spec:
  **replicas: 2** # Set replicas to 2
  selector:
    matchLabels:
      app: wonderful
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: wonderful
    spec:
      containers:
      - image: nginx:alpine # Checked
        name: nginx
        resources: {}
status: {}

controlplane $ k get pod
NAME                            READY   STATUS    RESTARTS   AGE
wonderful-v1-5d8586c8d4-4bh7m   1/1     Running   0          12m
wonderful-v1-5d8586c8d4-6fjk7   1/1     Running   0          12m
wonderful-v1-5d8586c8d4-9gfpx   1/1     Running   0          12m
wonderful-v1-5d8586c8d4-fssh6   1/1     Running   0          12m
wonderful-v1-5d8586c8d4-g64c6   1/1     Running   0          12m
wonderful-v1-5d8586c8d4-m4dds   1/1     Running   0          12m
wonderful-v1-5d8586c8d4-mjl8h   1/1     Running   0          12m
wonderful-v1-5d8586c8d4-r9lrp   1/1     Running   0          12m
wonderful-v2-dbf975874-94jl4    1/1     Running   0          50s
wonderful-v2-dbf975874-cmjv2    1/1     Running   0          50s

Verify:

watch -n 2 curl wonderful:30080 (每兩秒curl一次)


### Custom Resource Definitions (CRD)
Write the list of **all** installed CRDs into /root/crds .
Write the list of **all** DbBackup objects into /root/db-backups

controlplane $ k get crd -A > /root/crds
controlplane $ cat /root/crds 
NAME                                                  CREATED AT
bgpconfigurations.crd.projectcalico.org               2025-02-11T16:55:37Z
bgppeers.crd.projectcalico.org                        2025-02-11T16:55:37Z
blockaffinities.crd.projectcalico.org                 2025-02-11T16:55:37Z
caliconodestatuses.crd.projectcalico.org              2025-02-11T16:55:37Z
clusterinformations.crd.projectcalico.org             2025-02-11T16:55:37Z
crontabs.stable.killercoda.com                        2025-02-15T23:20:45Z
db-backups.stable.killercoda.com                      2025-02-15T23:20:45Z
felixconfigurations.crd.projectcalico.org             2025-02-11T16:55:37Z
globalnetworkpolicies.crd.projectcalico.org           2025-02-11T16:55:37Z
globalnetworksets.crd.projectcalico.org               2025-02-11T16:55:37Z
hostendpoints.crd.projectcalico.org                   2025-02-11T16:55:37Z
ipamblocks.crd.projectcalico.org                      2025-02-11T16:55:37Z
ipamconfigs.crd.projectcalico.org                     2025-02-11T16:55:37Z
ipamhandles.crd.projectcalico.org                     2025-02-11T16:55:37Z
ippools.crd.projectcalico.org                         2025-02-11T16:55:37Z
ipreservations.crd.projectcalico.org                  2025-02-11T16:55:37Z
kubecontrollersconfigurations.crd.projectcalico.org   2025-02-11T16:55:37Z
networkpolicies.crd.projectcalico.org                 2025-02-11T16:55:38Z
networksets.crd.projectcalico.org                     2025-02-11T16:55:38Z

controlplane $ k get db-backups -A > /root/db-backups 
controlplane $ cat /root/db-backups 
NAMESPACE     NAME                       AGE
default       postgres-sunny-sunshine    32m
default       postgres-walking-waveart   32m
kube-system   mysql-total-tolly          32m


The team worked really hard for months on a new Shopping-Items CRD which is currently in beta.
Install it from /code/crd.yaml .

Then create a ShoppingItem object named bananas in Namespace default . The dueDate should be tomorrow and the description should be buy yellow ones .


Install CRD yaml: 
controlplane $ cat /code/crd.yaml 
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: shopping-items.beta.killercoda.com
spec:
  group: beta.killercoda.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                description:
                  type: string
                dueDate:
                  type: string
  scope: Namespaced
  names:
    plural: shopping-items
    singular: shopping-item
    kind: ShoppingItem
controlplane $ k create -f  /code/crd.yaml 
customresourcedefinition.apiextensions.k8s.io/shopping-items.beta.killercoda.com created

controlplane $ k get crd shopping-items.beta.killercoda.com 
NAME                                 CREATED AT
shopping-items.beta.killercoda.com   2025-02-15T23:54:40Z

Create ShoppingItem object:
(以下為killercoda提供的object template)
apiVersion: "beta.killercoda.com/v1"
kind: 
metadata:
  name: bananas
spec:

改寫如下:
controlplane $ vim shoppingItem.yaml
apiVersion: "beta.killercoda.com/v1"
kind: ShoppingItem
metadata:
  name: bananas
spec:
  dueDate: tomorrow
  description: buy yellow ones


controlplane $ k create -f  shoppingItem.yaml
shoppingitem.beta.killercoda.com/bananas created
controlplane $ k get shopping-items                     
NAME      AGE
bananas   8s


You spent hours figuring out this "amazing" new CRD ShoppingItems created by the team.
But in the end you realised that it's pretty much the most useless CRD ever encountered.

If you need to run a Kubernetes cluster for managing your shopping list then things might have gone too far.
Delete the CRD and all ShoppingItem objects again.

controlplane $ k delete crd shopping-items.beta.killercoda.com 
customresourcedefinition.apiextensions.k8s.io "shopping-items.beta.killercoda.com" deleted


### Helm
Write the list of all Helm releases in the cluster into /root/releases.

helm ls = helm list

controlplane $ helm ls -A -a
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
apiserver       team-yellow     1               2025-02-16 21:30:45.085933707 +0000 UTC deployed        apache-11.3.2   2.4.63     
webserver       team-blue       1               2025-02-16 21:30:33.312674668 +0000 UTC deployed        apache-11.3.2   2.4.63     

controlplane $ helm ls -A > /root/releases

controlplane $ cat /root/releases 
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
apiserver       team-yellow     1               2025-02-16 21:30:45.085933707 +0000 UTC deployed        apache-11.3.2   2.4.63     
webserver       team-blue       1               2025-02-16 21:30:33.312674668 +0000 UTC deployed        apache-11.3.2   2.4.63 

Delete the Helm release apiserver.

controlplane $ helm -h
The Kubernetes package manager

Common actions for Helm:
...
Usage:
  helm [command]

Available Commands:
  completion  generate autocompletion scripts for the specified shell
  create      create a new chart with the given name
  dependency  manage a chart's dependencies
  env         helm client environment information
  get         download extended information of a named release
  help        Help about any command
  history     fetch release history
  install     install a chart
  lint        examine a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      install, list, or uninstall Helm plugins
  pull        download a chart from a repository and (optionally) unpack it in local directory
  push        push a chart to remote
  registry    login to or logout from a registry
  repo        add, list, remove, update, and index chart repositories
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  show        show information of a chart
  status      display the status of the named release
  template    locally render templates
  test        run tests for a release
  uninstall   uninstall a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the client version information
...

controlplane $ helm uninstall -n team-yellow apiserver
release "apiserver" uninstalled


Install the Helm chart falcosecurity/falco into Namespace team-yellow .

The release should've the name dev .

controlplane $ helm install -h
...
Usage:
  helm install [NAME] [CHART] [flags]
...

controlplane $ helm install dev falcosecurity/falco -n team-yellow
NAME: dev
LAST DEPLOYED: Sun Feb 16 21:40:53 2025
NAMESPACE: team-yellow
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Falco agents are spinning up on each node in your cluster. After a few
seconds, they are going to start monitoring your containers looking for
security issues.


No further action should be required.


Tip: 
You can easily forward Falco events to Slack, Kafka, AWS Lambda and more with falcosidekick. 
Full list of outputs: https://github.com/falcosecurity/charts/tree/master/charts/falcosidekick.
You can enable its deployment with `--set falcosidekick.enabled=true` or in your values.yaml. 
See: https://github.com/falcosecurity/charts/blob/master/charts/falcosidekick/values.yaml for configuration values.


### Ingress Create
There are two existing Deployments in Namespace world which should be made accessible via an Ingress.

First: create ClusterIP Services for both Deployments for port 80 . The Services should have the same name as the Deployments.

Tip: **k expose deploy -h**

controlplane $ k get deploy -n world
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
asia     2/2     2            2           6m32s
europe   2/2     2            2           6m32s

controlplane $ k -n world expose deploy asia --target-port=80
error: couldn't find port via --port flag or introspection

controlplane $ k -n world expose deploy asia --port=80                
service/asia exposed

controlplane $ k -n world expose deploy europe --port=80
service/europe exposed


The Nginx Ingress Controller has been installed.

Create a new Ingress resource called world for domain name world.universe.mine . The domain points to the K8s Node IP via /etc/hosts .

The Ingress resource should have two routes pointing to the existing Services:
http://world.universe.mine:30080/europe/
and
http://world.universe.mine:30080/asia/


controlplane $ k get deploy -n world
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
asia     2/2     2            2           6m32s
europe   2/2     2            2           6m32s


**TIPS: Check the NodePort Service for the Nginx Ingress Controller to see the ports**

controlplane $ k -n ingress-nginx get svc ingress-nginx-controller
NAME                       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller   NodePort   10.97.229.201   <none>        **80:30080**/TCP,443:30443/TCP   18m

We can reach the NodePort Service via the k8s Node IP: 170.30.1.2

controlplane $ curl http://172.30.1.2:30080
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>

controlplane $ k get nodes -A
NAME           STATUS   ROLES           AGE    VERSION
controlplane   Ready    control-plane   5d5h   v1.31.0

controlplane $ k describe node controlplane
...
Addresses:
  InternalIP:  **172.30.1.2**
  Hostname:    controlplane
...

And:
controlplane $ curl http://world.universe.mine:30080
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>

**Tips: Find out the ingressClassName with: k get ingressclass**
由於ingress controll 將會在考試時提供，但務必檢查是否有任何ingress classes!! (這是陷阱)，有的話就需要加進ingress配置文件
controlplane $ k get ingressclasses                   
NAME        CONTROLLER             PARAMETERS   AGE
**nginx**   k8s.io/ingress-nginx   <none>       20m

以下兩個service均監聽80端口:
controlplane $ k get svc -n world
NAME     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
asia     ClusterIP   10.106.105.114   <none>        **80**/TCP    32m
europe   ClusterIP   10.101.10.253    <none>        **80**/TCP    31m
(Ingress 會將來自 http://world.universe.mine/europe 或 http://world.universe.mine/asia 的請求，透過 Ingress Controller 轉發到對應的 Service)

controlplane $ vim ingress-world.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: world
  namespace: world
  **annotations**: # 最好加一下
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  **ingressClassName: nginx** 
  rules:
  - host: "world.universe.mine"
    http:
      paths:
      - **pathType: Prefix**
        path: "/europe"  
        backend:
          service:
            name: europe
            port:
              number: **80** # Checked
      - **pathType: Prefix**
        path: "/asia"  
        backend:
          service:
            name: asia
            port:
              number: **80** Checked

controlplane $ k create -f  ingress-world.yaml 
ingress.networking.k8s.io/world created

controlplane $ k get ingress -n world 
NAME    CLASS           HOSTS                 ADDRESS   PORTS   AGE
world   nginx-example   world.universe.mine             80      7s



### NetworkPolicy Namespace Selector
There are existing Pods in Namespace space1 and space2 .

We need a new NetworkPolicy named np that restricts all Pods in Namespace space1 to only have outgoing traffic to Pods in Namespace space2 . Incoming traffic not affected.

The NetworkPolicy should still allow outgoing DNS traffic on port 53 TCP and UDP.


我的配置:

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: ~~default~~ # 應為:space1
spec:
  podSelector:
    ~~matchLabels:~~ # 無須配置podSelector
      ~~kubernetes.io/metadata.name: space1~~
  policyTypes:
  - Egress
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: space2
      ports:
        - protocol: TCP
          port: 53
        - protocol: UDP
          port: 53


需要在networkpolicy中設置namespace為space1的原因為: networkpolicy為基於namespace的配置

controlplane $ k api-resources |grep -i "networkpolicy"
globalnetworkpolicies                            crd.projectcalico.org/v1          false        GlobalNetworkPolicy
networkpolicies                                  crd.projectcalico.org/v1          true         NetworkPolicy
networkpolicies                     netpol       networking.k8s.io/v1              **true**     NetworkPolicy

正確配置:

controlplane $ vim np.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: space2
      ports:
        - protocol: TCP
          port: 53
        - protocol: UDP
          port: 53

controlplane $ k create -f  np.yaml
networkpolicy.networking.k8s.io/np created
controlplane $ k get networkpolicies -n space1
NAME   POD-SELECTOR   AGE
np     <none>         12s

controlplane $ k get pod -n space1
NAME     READY   STATUS    RESTARTS   AGE
app1-0   1/1     Running   0          20m
controlplane $ k get pod -n space2
NAME              READY   STATUS    RESTARTS   AGE
microservice1-0   1/1     Running   0          20m
microservice2-0   1/1     Running   0          20m
controlplane $ k get pod
NAME       READY   STATUS    RESTARTS   AGE
tester-0   1/1     Running   0          22m

**Verify**:  (space1 ) 

測試app1-0連接 microservice1-0跟microservice2-0 應成功，app1-0 連接tester-0應失敗

已知 Pod's hostname and subdomain fields : <pod-name>.<namespace>.svc.cluster.local
eg. busybox-1.busybox-subdomain.my-namespace.svc.cluster-domain.example
(doc: )


故space2的microservice1的hostname為: microservice1.space2.svc.cluster.local
  space2的microservice2hostname為: microservice2.space2.svc.cluster.local

執行以下指令: (-m 1: 請求最多允許執行 1 秒)
k -n space1 exec app1-0 -- curl -m 1 microservice1.space2.svc.cluster.local
k -n space1 exec app1-0 -- curl -m 1 microservice2.space2.svc.cluster.local
k -n space1 exec app1-0 -- curl -m 1 tester.default.svc.cluster.local

(發現 DNS 解析超時 而導致curl失敗: )
controlplane $ k exec -n space1 app1-0 -- curl microservice1.space2.svc.cluster.local
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:04 --:--:--     0curl: (6) Could not resolve host: microservice1.space2.svc.cluster.local
command terminated with exit code 6
controlplane $ ^C
controlplane $ k exec -n space1 app1-0 -- curl -m 1 microservice1.space2.svc.cluster.local
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:05 --:--:--     0
curl: (28) Resolving timed out after 1000 milliseconds
command terminated with exit code 28

此問題有瑕疵:

DNS 服務不在 space2，而在 kube-system

你的 NetworkPolicy 只允許 space1 的 Pod 訪問 space2，但 Kubernetes 的 CoreDNS 服務通常運行在 kube-system Namespace。
如果 app1-0 的 DNS 查詢需要訪問 kube-system.kube-dns，但 NetworkPolicy 沒有允許 kube-system，DNS 解析會失敗，導致 curl 無法找到 microservice1.space2.svc.cluster.local。
NetworkPolicy 的 DNS 端口可能設錯

你的 NetworkPolicy 允許的 egress 目標是 space2，但 kube-dns 在 kube-system。
你只允許了 53/TCP 和 53/UDP 到 space2，但 kube-system 內的 CoreDNS 沒有被允許

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: space2
    - to:
      - **namespaceSelector**:
          matchLabels:
            kubernetes.io/metadata.name: kube-system # k get ns --show-labels 獲取
      ports:
        - protocol: TCP
          port: 53
        - protocol: UDP
          port: 53

controlplane $ k -n space1 exec app1-0 -- curl -m 1 microservice1.space2.svc.cluster.local
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   615  100   615    0     0   112k      0 --:--:-- --:--:-- --:--:--  120k

(多設置namespaceSelector: kube-system才可以使curl成功執行，這與題目的: all Pods in Namespace space1 to only have outgoing traffic to Pods in Namespace space2. 不符合)

controlplane $ k -n space1 exec app1-0 -- curl -m 1 tester.default.svc.cluster.local
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
curl: (28) Connection timed out after 1000 milliseconds
command terminated with exit code 28


### Api Deprecations

Checked the installed K8s version:

controlplane $ k version
Client Version: v1.31.0
Kustomize Version: v5.4.2
Server Version: v1.31.0

Answer: v1.31.0 (Major: 1, Minor: 31, Patch:0)


Write the Api Group of Deployments into /root/group .

controlplane $ k explain deploy| grep -i "VERSION"
VERSION:    v1
  apiVersion    <string>
    APIVersion defines the versioned schema of this representation of an object.

controlplane $ k explain deploy | grep -i "VERSION:" | awk '{print $2}'
v1

(awk '{print $2}'：只提取第二個欄位，即 v1)

controlplane $ k explain deploy | grep -i "VERSION:" | awk '{print $2}' > /root/group 
controlplane $ cat /root/group 
v1


There is a CronJob file at /apps/cronjob.yaml which uses a deprecated Api version.
Update the file to use the non deprecated one


controlplane $ k api-resources |grep cronjob
cronjobs                            cj           batch/v1                          true         CronJob

controlplane $ cat /apps/cronjob.yaml 
apiVersion:  batch/v1  # batch/v1beta1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "5 4 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: bash
            command:
            - /bin/sh
            - -c
            - date
          restartPolicy: OnFailure

EE