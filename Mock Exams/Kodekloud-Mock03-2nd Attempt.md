2nd Attempt Score: 58 % Pass Score: 75%

錯題: 
01 (粗心，image打錯字)
02 (沒有設置restartPolicy)
07 (沒有備份題目範例，導致不知道自己改了什麼)
09
10 (題目要求有陷阱!!podSelector應分開設置)
14 (見Mock03設置)
17 (slector內app: nginx拼錯!! label跟pod 不一致將導致Endpoint創建不成功)   




07. 

09. 
    我的配置:

    student-node ~ ➜  k get svc -n ns-ckad17-svcn 
    NAME               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
    cpu-load-app       ClusterIP      172.20.144.116   <none>        80/TCP            88m
    geo-location-app   ClusterIP      172.20.137.118   <none>        80/TCP            88m
    lb1-ckad17-svcn    LoadBalancer   172.20.62.211    <pending>     31890:31125/TCP   84m
    lb2-ckad17-svcn    LoadBalancer   172.20.92.188    <pending>     31891:30692/TCP   82m

    O Is service lb1-ckad17-svcn created? 
    O Is service lb2-ckad17-svcn created?
    O Are correct labels used for lb1 ?
    O Are correct labels used for lb2?
    X Is port 31890 configured for lb1 ?
    X Is port 31891 configured for lb2?

    student-node ~ ➜  k describe svc -n ns-ckad17-svcn lb1-ckad17-svcn 
    Name:                     lb1-ckad17-svcn
    Namespace:                ns-ckad17-svcn
    Labels:                   criteria=location
                            exam=ckad
    Annotations:              <none>
    Selector:                 criteria=location,exam=ckad
    Type:                     LoadBalancer
    IP Family Policy:         SingleStack
    IP Families:              IPv4
    IP:                       172.20.62.211
    IPs:                      172.20.62.211
    Port:                     <unset>  31890/TCP
    TargetPort:               31890/TCP
    NodePort:                 <unset>  31125/TCP
    Endpoints:                172.17.1.2:31890
    Session Affinity:         None
    External Traffic Policy:  Cluster
    Events:                   <none>

    student-node ~ ➜  k describe svc -n ns-ckad17-svcn lb2-ckad17-svcn 
    Name:                     lb2-ckad17-svcn
    Namespace:                ns-ckad17-svcn
    Labels:                   criteria=cpu-high
                            exam=ckad
    Annotations:              <none>
    Selector:                 criteria=cpu-high,exam=ckad
    Type:                     LoadBalancer
    IP Family Policy:         SingleStack
    IP Families:              IPv4
    IP:                       172.20.92.188
    IPs:                      172.20.92.188
    Port:                     <unset>  31891/TCP
    TargetPort:               31891/TCP
    NodePort:                 <unset>  30692/TCP
    Endpoints:                172.17.1.3:31891
    Session Affinity:         None
    External Traffic Policy:  Cluster
    Events:                   <none>

    student-node ~ ✖ k get svc -n ns-ckad17-svcn 
    NAME               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
    cpu-load-app       ClusterIP      172.20.144.116   <none>        80/TCP            91m
    geo-location-app   ClusterIP      172.20.137.118   <none>        80/TCP            91m
    lb1-ckad17-svcn    LoadBalancer   172.20.62.211    <pending>     31890:31125/TCP   88m
    lb2-ckad17-svcn    LoadBalancer   172.20.92.188    <pending>     31891:30692/TCP   86m

    student-node ~ ➜  k get svc -n ns-ckad17-svcn --show-labels 
    NAME               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE   LABELS
    cpu-load-app       ClusterIP      172.20.144.116   <none>        80/TCP            91m   criteria=cpu-high,exam=ckad
    geo-location-app   ClusterIP      172.20.137.118   <none>        80/TCP            91m   criteria=location,exam=ckad
    lb1-ckad17-svcn    LoadBalancer   172.20.62.211    <pending>     31890:31125/TCP   88m   criteria=location,exam=ckad
    lb2-ckad17-svcn    LoadBalancer   172.20.92.188    <pending>     31891:30692/TCP   86m   criteria=cpu-high,exam=ckad

    student-node ~ ➜  k get pod -n ns-ckad17-svcn --show-labels 
    NAME               READY   STATUS    RESTARTS   AGE   LABELS
    cpu-load-app       1/1     Running   0          92m   criteria=cpu-high,exam=ckad
    geo-location-app   1/1     Running   0          92m   criteria=location,exam=ckad


10. 陷阱! 題目要求不改動原有的設置，應多新設置podSelector:

    spec:
    ingress:
    - from:
        - podSelector:
            matchLabels:
                access: allowed
                ~~tier: server~~
            
    podSelector:
        matchLabels:
            app: kk-app
    policyTypes:
    - Ingress
    - Egress

    
    **陷阱!!!**
    你的錯誤配置並沒有違反 Kubernetes Network Policy 的 YAML 語法，因此 kubectl create 不會報錯。然而，**KodeKloud 要求的是 在不修改現有規則的情況下，新增允許 access=allowed 的規則**，而你的錯誤配置其實 修改了 原本的規則，使其變成只有 access: allowed，而不是額外新增這個條件。

    應改為:
    spec:
    ingress:
    - from:
        - podSelector:
            matchLabels:
                access: allowed
        - podSelector:
            matchLabels:
                tier: server
    **podSelector**:
        matchLabels:
            app: kk-app
    policyTypes:
    - Ingress
    - Egress


14. (見Mock03) For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    In the ckad14-sa-projected namespace, configure the ckad14-api-pod Pod to include a projected volume named vault-token.

    Mount the service account token to the container at /var/run/secrets/tokens, with an expiration time of 7000 seconds.

    Additionally, set the intended audience for the token to vault and path to vault-token.

    我的配置:
    ...
    spec:
    containers:
    - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: vault-token # 變更
        readOnly: true
    ...
    serviceAccount: ckad14-sa # default
    serviceAccountName: ckad14-sa # defualt
   ...
    volumes:
      - name: vault-token # 變更
        projected:
          defaultMode: 420
          sources:
          - serviceAccountToken:
              expirationSeconds: 7000 # 變更
              path: /var/run/secrets/tokens # 變更
          - configMap:
              items:
              - key: ca.crt
                path: ca.crt
              name: kube-root-ca.crt
          - downwardAPI:
              items:
              - fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
                path: namespace


    Pod 一直停在containerCreating的狀態
    student-node ~ ➜  k get pod -n ckad14-sa-projected 
    NAME             READY   STATUS              RESTARTS   AGE
    ckad14-api-pod   0/1     ContainerCreating   0          51m

    k describe發現:
    Events:
    Type     Reason       Age                 From     Message
    ----     ------       ----                ----     -------
    Warning  FailedMount  70s (x33 over 52m)  kubelet  MountVolume.SetUp failed for volume "vault-token" : invalid path: must be relative path: /var/run/secrets/tokens

15. (正確ㄎ)
    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    Create a custom resource my-anime of kind Anime with the below specifications:


    Name of Anime: Naruto
    Episode Count: 220


    TIP: You may find the respective CRD with anime substring in it.


    student-node ~ ✖ cat my-anime.yaml
    apiVersion: animes.k8s.io/v1alpha1
    kind: Anime
    metadata: # my-anime
    name: 
    spec:
    animeName: Naruto
    episodeCount: 220

    student-node ~ ➜  kc my-anime.yaml
    error: unable to decode "my-anime.yaml": json: cannot unmarshal string into Go struct field metadataOnlyObject.metadata of type v1.ObjectMeta



17. 



student-node ~ ➜  kc ckad-endpoint-service-aom.yaml
error: unable to decode "ckad-endpoint-service-aom.yaml": json: cannot unmarshal string into Go struct field metadataOnlyObject.metadata of type v1.ObjectMeta

錯誤原因： metadata 內應該包含 name 屬性，而不是一個單獨的字串。



🔴 錯誤分析
❌ 你的錯誤 YAML：
yaml
Copy
Edit
apiVersion: v1
kind: Endpoints
metadata: 
  name: ckad-nginx-service-aom
subsets: 
  addresses: 10.43.212.119  # ❌ 這裡的 addresses 不是物件陣列，導致解析錯誤
✅ 正確的 YAML
yaml
Copy
Edit
apiVersion: v1
kind: Endpoints
metadata:
  name: ckad-nginx-service-aom
subsets:
  - addresses:
      - ip: 10.43.212.119
📌 修正重點
subsets 需要是陣列，所以前面要有 -
addresses 內的 IP 需要是物件陣列，寫法應該是：
yaml
Copy
Edit
addresses:
  - ip: 10.43.212.119
- student-node ~ ➜  vim ckad-nginx-endpoint-aom.yaml

student-node ~ ➜  kc ckad-nginx-endpoint-aom.yaml
Error from server (BadRequest): error when creating "ckad-nginx-endpoint-aom.yaml": Endpoints in version "v1" cannot be handled as a Endpoints: json: cannot unmarshal object into Go struct field Endpoints.subsets of type []v1.EndpointSubset




student-node ~ ✖ cat ckad-nginx-endpoint-aom.yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: ckad-nginx-endpoint-aom
subsets:
  addresses:
  - ip: 10.43.212.119

student-node ~ ➜  vim ckad-nginx-endpoint-aom.yaml

student-node ~ ➜  kc ckad-nginx-endpoint-aom.yaml
Error from server (BadRequest): error when creating "ckad-nginx-endpoint-aom.yaml": Endpoints in version "v1" cannot be handled as a Endpoints: json: cannot unmarshal object into Go struct field Endpoints.subsets of type []v1.EndpointSubset

🔴 錯誤分析
你的 YAML：

yaml
Copy
Edit
apiVersion: v1
kind: Endpoints
metadata:
  name: ckad-nginx-endpoint-aom
subsets:
  addresses:
  - ip: 10.43.212.119
錯誤原因：

subsets 必須是一個陣列（list），但你的 addresses 直接放在 subsets 下，導致解析錯誤。
addresses 必須放在 - 代表的 subsets 內。
✅ 正確的 YAML
yaml
Copy
Edit
apiVersion: v1
kind: Endpoints
metadata:
  name: ckad-nginx-endpoint-aom
subsets:
  - addresses:
      - ip: 10.43.212.119

student-node ~ ✖ vim ckad-nginx-endpoint-aom.yaml

student-node ~ ➜  kc ckad-nginx-endpoint-aom.yaml
endpoints/ckad-nginx-endpoint-aom created

student-node ~ ➜  k get endpoints
NAME                      ENDPOINTS                                                    AGE
kubernetes                192.168.81.187:6443                                          91m
route-apd-svc             10.42.0.10:8080,10.42.0.11:8080,10.42.0.9:8080 + 7 more...   47m
nginx-svcn                10.42.0.12:443,10.42.1.10:443,10.42.2.13:443 + 3 more...     37m
ckad-nginx-service-aom    <none>                                                       15m
ckad-nginx-endpoint-aom   10.43.212.119                                                5s

student-node ~ ➜  k describe svc ckad-nginx-service-aom 
Name:              ckad-nginx-service-aom
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=ngnix
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.212.119
IPs:               10.43.212.119
Port:              80-80  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>


 Service 的 selector 與 Endpoints 不匹配
🔴 你的 Service ckad-nginx-service-aom 有 selector: app=ngnix

spec:
  clusterIP: 10.43.212.119
  clusterIPs:
  - 10.43.212.119
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: 80-80
    port: 80
    protocol: TCP
    targetPort: 80
  selector: {}
    # app: ngnix 
  sessionAffinity: None


  student-node ~ ➜  k edit svc ckad-nginx-service-aom 
service/ckad-nginx-service-aom edited

student-node ~ ➜  k describe svc ckad-nginx-service-aom 
Name:              ckad-nginx-service-aom
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.212.119
IPs:               10.43.212.119
Port:              80-80  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>


 確認 Service clusterIP 是否與 Endpoints IP 一致
🔴 你的 Service ckad-nginx-service-aom 的 ClusterIP 是 10.43.212.119，而 Endpoints 也用相同 IP (10.43.212.119)。

✅ 解決方案

不要 在 Endpoints 中使用 ClusterIP (10.43.212.119)，因為 Service 會自己負責負載均衡到 Endpoints。
**Endpoints 應該使用 Pod 的 IP，而不是 10.43.212.119**。




student-node ~ ➜  k get pod ckad-nginx-pod-aom --show-labels 
NAME                 READY   STATUS    RESTARTS   AGE   LABELS
ckad-nginx-pod-aom   1/1     Running   0          18m   app=nginx

student-node ~ ➜  k describe pod ckad-nginx-pod-aom 
Name:             ckad-nginx-pod-aom
Namespace:        default
Priority:         0
Service Account:  default
Node:             cluster1-node02/192.168.140.165
Start Time:       Tue, 25 Feb 2025 18:56:40 +0000
Labels:           app=nginx
Annotations:      <none>
Status:           Running
IP:               **10.42.1.11**
IPs:
  IP:  10.42.1.11


修改endppoints IP:
student-node ~ ➜  vim ckad-nginx-endpoint-aom.yaml

(failed reverse-i-search)`': kr^Ckad-nginx-endpoint-aom.yaml

student-node ~ ✖ kr ckad-nginx-endpoint-aom.yaml --force
endpoints "ckad-nginx-endpoint-aom" deleted
endpoints/ckad-nginx-endpoint-aom replaced

student-node ~ ➜  k describe svc ckad-nginx-service-aom 
Name:              ckad-nginx-service-aom
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.212.119
IPs:               10.43.212.119
Port:              80-80  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>




spec:
  clusterIP: 10.43.212.119
  clusterIPs:
  - 10.43.212.119
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: 80-80
    port: 80
    protocol: TCP
    targetPort: 80
  selector: 
    app: ngnix  # 改回來

student-node ~ ➜  k edit svc ckad-nginx-service-aom 
service/ckad-nginx-service-aom edited

student-node ~ ➜  k describe svc ckad-nginx-service-aom 
Name:              ckad-nginx-service-aom
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=ngnix
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.212.119
IPs:               10.43.212.119
Port:              80-80  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>


student-node ~ ➜  kubectl get pod ckad-nginx-pod-aom -o json | jq -r .metadata.labels
{
  "app": "nginx"
}


student-node ~ ✖ k get svc ckad-nginx-service-aom -o json | jq -r .metadata.labels
null

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
IP:                10.43.212.119
IPs:               10.43.212.119
Port:              80-80  80/TCP
TargetPort:        80/TCP
Endpoints:         10.42.1.11:80
Session Affinity:  None
Events:            <none>

student-node ~ ➜  k edit svc ckad-nginx-service-aom
...
spec:
  clusterIP: 10.43.212.119
  clusterIPs:
  - 10.43.212.119
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: 80-80
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx # nignx 拼錯!!!
  sessionAffinity: None
  type: ClusterIP
...


20.  (正確)

    student-node ~ ➜  k get pods -n spectra-1267 -o custom-columns=POD_NAME:.metadata.name,IP_ADDR:.spec.hostIP
    POD_NAME   IP_ADDR
    pod-12     <none>
    pod-21     <none>
    pod-23     <none>
    pod-32     <none>
    pod-34     <none>
    pod-43     <none>

    student-node ~ ➜  vim pod-12.yaml 

    student-node ~ ➜  cat  pod-12.yaml |grep -A15 status
    status:
    ...
    hostIP: 192.168.78.171
    hostIPs:
    podIP: 

    student-node ~ ➜  k get pods -n spectra-1267 -o custom-columns=POD_NAME:metadata.name,IP_ADDR:stauts.po # 拼錯
    dIP
    POD_NAME   IP_ADDR
    pod-12     <none>
    pod-21     <none>
    pod-23     <none>
    pod-32     <none>
    pod-34     <none>
    pod-43     <none>

    student-node ~ ➜  k get pods -n spectra-1267 -o custom-columns=POD_NAME:metadata.name,IP_ADDR:status.po
    dIP
    POD_NAME   IP_ADDR
    pod-12     172.17.1.8
    pod-21     172.17.1.12
    pod-23     172.17.1.9
    pod-32     172.17.1.11
    pod-34     172.17.1.10
    pod-43     172.17.1.13

    student-node ~ ➜  k get pods -n spectra-1267 -o custom-columns=POD_NAME:metadata.name,IP_ADDR:status.podIP --s
    --selector             (Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l ke…)
    --server-print         (If true, have the server return the appropriate table output. Supports extens…)
    --server               (The address and port of the Kubernetes API server)
    --show-kind            (If present, list the resource type for the requested object(s).)
    --show-labels          (When printing, show all labels as the last column (default hide labels column))
    --show-managed-fields  (If true, keep the managedFields when printing objects in JSON or YAML format.)
    --sort-by              (If non-empty, sort list types using this field specification.  The field spec…)
    --subresource          (If specified, gets the subresource of the requested object. Must be one of [s…)

    student-node ~ ➜  k get pods -n spectra-1267 -o custom-columns=POD_NAME:metadata.name,IP_ADDR:status.podIP --sort-by=status
    F0225 19:41:50.391948   27199 sorter.go:331] Field {.status} in *unstructured.Unstructured is an unsortable type: interface, err: unsortable type: map[string]interface {}

    student-node ~ ✖ k get pods -n spectra-1267 -o custom-columns=POD_NAME:metadata.name,IP_ADDR:status.podIP --sort-by=status.podIP
    POD_NAME   IP_ADDR
    pod-12     172.17.1.8
    pod-23     172.17.1.9
    pod-34     172.17.1.10
    pod-32     172.17.1.11
    pod-21     172.17.1.12
    pod-43     172.17.1.13


    student-node ~ ➜  cat /root/pod_ips_cka05_svcn 
    POD_NAME   IP_ADDR
    pod-12     172.17.1.8
    pod-23     172.17.1.9
    pod-34     172.17.1.10
    pod-32     172.17.1.11
    pod-21     172.17.1.12
    pod-43     172.17.1.13



