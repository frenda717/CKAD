1.32 
First Attempt: 
Score: 46% Pass Score: 75%
Correct Answer: 01,03,06,08,11,13,15,16,18,19
難題: 
錯題: 02,04,07,10 ,14,16
1.32版本新Concept: projected volume (14)

### SECTION: APPLICATION DESIGN AND BUILD
02. (RestartPolicy:Never) In the ckad-pod-design namespace, start a ckad-nginx-uahsbcbdkl pod running the nginx:1.17 image.

    Configure the pod with a label:
    TRAINER: KODEKLOUD

    The pod should not be restarted in any case if it has already exited.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    我的配置: 幾乎都對，但是沒有設置restartPolicy為Never

    Solution: 

    Create a YAML file with the content as below:

    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        TRAINER: KODEKLOUD
    name: ckad-nginx-uahsbcbdkl
    namespace: ckad-pod-design
    spec:
    containers:
    - image: nginx:1.17
        name: ckad-nginx-uahsbcbdkl
        resources: {}
    dnsPolicy: ClusterFirst
    **restartPolicy: Never**
    status: {}

    Then use kubectl apply -f file_name.yaml to create the required object.

04. (CrashLoopBackOff) In the ckad-multi-containers namespace, create a pod named cuda-pod, which has 2 containers matching the below requirements:

    The first container named alpha runs alpine image and has release=stable environment variable.
    The second container named beta runs nginx:1.17 image and is exposed at port 8080.

    NOTE: All pod containers should be in the running state.
    
    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    我的配置: (一新建立pod 便出現CrashLoopBackOff)
    student-node ~ ➜  k get pod -n ckad-multi-containers 
    NAME       READY   STATUS             RESTARTS        AGE
    cuda-pod   1/2     CrashLoopBackOff   20 (5m2s ago)   82m

    student-node ~ ➜  k describe pod -n ckad-multi-containers cuda-pod 
    Name:             cuda-pod
    Namespace:        ckad-multi-containers
    Priority:         0
    Service Account:  default
    Node:             cluster2-node01/192.168.231.175
    Start Time:       Mon, 24 Feb 2025 23:12:26 +0000
    Labels:           run=cuda-pod
    Annotations:      <none>
    Status:           Running
    IP:               10.42.1.5
    IPs:
    IP:  10.42.1.5
    Containers:
    alpha:
        Container ID:   containerd://ede1603cf0eba5c70c685a9806cfce67bf7bac3b80ea525e757bc6449c0506f0
        Image:          alpine
        Image ID:       docker.io/library/alpine@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
        Port:           <none>
        Host Port:      <none>
        State:          Waiting
        Reason:       CrashLoopBackOff
        Last State:     Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Tue, 25 Feb 2025 00:34:49 +0000
        Finished:     Tue, 25 Feb 2025 00:34:49 +0000
        Ready:          False
        Restart Count:  21
        Environment:
        release:  stable
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rg8nm (ro)
    beta:
        Container ID:   containerd://f3acd35b2fcfc4816c873d8827cf1f64c99795562228964ba945299da37da7eb
        Image:          nginx:1.17
        Image ID:       docker.io/library/nginx@sha256:6fff55753e3b34e36e24e37039ee9eae1fe38a6420d8ae16ef37c92d1eb26699
        Port:           8080/TCP
        Host Port:      0/TCP
        State:          Running
        Started:      Mon, 24 Feb 2025 23:12:27 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rg8nm (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       False 
    ContainersReady             False 
    PodScheduled                True 
    Volumes:
    kube-api-access-rg8nm:
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
    Warning  BackOff  3m2s (x371 over 83m)  kubelet  Back-off restarting failed container alpha in pod cuda-pod_ckad-multi-containers(fca40a3f-fccd-4091-a95b-5c05e827029a)


05. (sh 進入容器) In the ckad-pod-design namespace, we created a pod named custom-nginx that runs the nginx:1.17 image.

    Take appropriate actions to update the index.html page of this NGINX container with below value instead of default NGINX welcome page:
    Welcome to CKAD mock exams!

    NOTE: By default NGINX web server default location is at /usr/share/nginx/html which is located on the default file system of the Linux.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    我忘記進入container 使用-- sh指令即可

    Solution:

    Exec to the pod container and update the index.html file content:
    student-node ~ ➜kubectl exec -it -n ckad-pod-design custom-nginx **-- sh**
    /# echo 'Welcome to CKAD mock exams!' > /usr/share/nginx/html/index.html

    Observe the result:
    student-node ~ ➜  kubectl exec -it -n ckad-pod-design custom-nginx -- cat /usr/share/nginx/html/index.html
    Welcome to CKAD mock exams!


### SECTION: APPLICATION DEPLOYMENT
07. (Helm) On the student-node, a Helm chart repository is given under the /opt/ path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. 
    
    The files have some issues. Fix those issues and deploy them with the following specifications: -

    The release name should be webapp-color-apd.
    All the resources should be deployed on the frontend-apd namespace.
    The service type should be node port.
    Scale the deployment to 3.
    Application version should be 1.20.0.
    NOTE: - Remember to make necessary changes in the values.yaml and Chart.yaml files according to the specifications, and, to fix the issues, inspect the template files.

    You can start new terminal to open student-node command.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    Run the following command to change the context: -
    kubectl config use-context cluster2


    In this task, we will use the helm commands. Here are the steps: -

    First, check the given namespace; if it doesn't exist, we must create it first; otherwise, it will give an error "namespaces not found" while installing the helm chart.
    To check all the namespaces in the cluster2, we would have to run the following command: -
    kubectl get ns


    It will list all the namespaces. If the given namespace doesn't exist, then run the following command: -
    kubectl create ns frontend-apd

    Now, on the student-node node and go to the /opt/ directory. We have given the helm chart directory webapp-color-apd that contains templates, values files, and the chart file etc.

    Update the values according to the given specifications as follows: -

    a.) Update the value of the appVersion to 1.20.0 in the Chart.yaml file.
    b.) Update the value of the replicaCount to 3 in the values.yaml file.
    c.) Update the value of the type to NodePort in the values.yaml file.

    These are the values we have to update.

    Now, we will use the helm lint command to check the Helm chart because it can identify errors such as missing or misconfigured values, invalid YAML syntax, and deprecated APIs etc.

    cd /opt/
    helm lint ./webapp-color-apd/


    If there is no misconfiguration, we will see the similar output: -

    helm lint ./webapp-color-apd/
    ==> Linting ./webapp-color-apd/
    [INFO] Chart.yaml: icon is recommended

    1 chart(s) linted, 0 chart(s) failed

    But in our case, there are some issues with the given templates.
        a. Deployment apiVersion needs to be correctly written. It should be apiVersion: apps/v1.
        b. In the service YAML, there is a typo in the template variable {{ .Values.service.name }} because of that, it's not able to reference the value of the name field defined in the values.yaml file for the Kubernetes service that is being created or updated.

    Now run the following command to install the helm chart in the frontend-apd namespace: -

    # Navigate to the directory having the chart
    cd /opt
    # Install the helm chart
    helm install webapp-color-apd -n frontend-apd ./webapp-color-apd

    Use the helm ls command to list the release deployed using helm.

    helm ls -n frontend-apd


### SECTION: SERVICES AND NETWORKING

09.  We have deployed several applications in the ns-ckad17-svcn namespace that are exposed inside the cluster via ClusterIP.
    We have deployed several applications in the ns-ckad17-svcn namespace that are exposed inside the cluster via ClusterIP.
    Your task is to create a LoadBalancer type service that will serve traffic to the applications based on its labels. Create the resources as follows:
    Service lb1-ckad17-svcn for serving traffic at port 31890 to pods with labels "exam=ckad, criteria=location".
    Service lb2-ckad17-svcn for serving traffic at port 31891 to pods with labels "exam=ckad, criteria=cpu-high".

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    我的配置:
    student-node ~ ➜  cat lb1-ckad1.yaml 
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        exam: ckad
        criteria: location
    name: lb1-ckad17-svcn
    namespace: ns-ckad17-svcn
    spec:
    ports:
    - name: "31890"
        port: 31890
        protocol: TCP
        targetPort: 31890
    selector:
        exam: ckad
        criteria: location
    type: LoadBalancer
    status:
    loadBalancer: {}


    student-node ~ ✖ cat lb2-ckad17-svcn.yaml 
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        exam: ckad
        criteria: cpu-high
    name: lb2-ckad17-svcn
    namespace: ns-ckad17-svcn
    spec:
    ports:
    - name: "31891"
        port: 31891
        protocol: TCP
        targetPort: 31891
    selector:
        exam: ckad
        criteria: cpu-high
    type: LoadBalancer
    status:
    loadBalancer: {}


    Solution:
    To create the loadbalancer for the pods with the specified lables, first we need to find the pods with the mentioned lables.

    To get pods with labels "exam=ckad, criteria=location"
    kubectl -n ns-ckad17-svcn get pod -l exam=ckad,criteria=location
    -----
    NAME               READY   STATUS    RESTARTS   AGE
    geo-location-app   1/1     Running   0          10m

    Similarly to get pods with labels "exam=ckad,criteria=cpu-high".
    kubectl -n ns-ckad17-svcn get pod -l exam=ckad,criteria=cpu-high
    -----
    NAME           READY   STATUS    RESTARTS   AGE
    cpu-load-app   1/1     Running   0          11m

    Now we know which pods use the labels, we can create the LoadBalancer type service using the imperative command.
    kubectl -n ns-ckad17-svcn expose pod geo-location-app --type=LoadBalancer --name=lb1-ckad17-svcn

    Similarly, create the another service.
    kubectl -n ns-ckad17-svcn expose pod cpu-load-app --type=LoadBalancer --name=lb2-ckad17-svcn

    Once the services are created, you can edit the services to use the correct nodePorts as per the question using kubectl -n ns-ckad17-svcn edit svc lb2-ckad17-svcn.

    student-node ~ ➜  k get svc -n ns-ckad17-svcn 
    NAME               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
    cpu-load-app       ClusterIP      172.20.139.168   <none>        80/TCP            75m
    geo-location-app   ClusterIP      172.20.131.144   <none>        80/TCP            75m
    lb1-ckad17-svcn    LoadBalancer   172.20.246.34    <pending>     31890:31496/TCP   67m
    lb2-ckad17-svcn    LoadBalancer   172.20.215.245   <pending>     31891:32253/TCP   68m

    student-node ~ ➜  ls
    ckad14-api-pod.yaml           custom-nginx.yaml     mock-user-binding.yaml
    ckad-busybox-gsqodeuykj.yaml  goiproxy.yaml         my-anime.yaml
    ckad-multi-containers.yaml    goproxy.yaml          netpol-ckad13-svcn.yaml
    ckad-nginx-endpoint.yaml      lb1-ckad1             nginx-app-ckad.yaml
    ckad-nginx-pod-aom.yaml       lb1-ckad17-svcn.yaml  node-metrics
    ckad-nginx-service-aom.yaml   lb1-ckad1.yaml        service-3421-svcn.yaml
    ckad-nginx-uahsbcbdkl.yaml    lb2-ckad17-svcn.yaml  simple-python-job.yaml


10. For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3



    We have created a Network Policy netpol-ckad13-svcn that allows traffic only to specific pods and it allows traffic only from pods with specific labels.

    Your task is to edit the policy so that it allows traffic from pods with labels access = allowed.



    Do not change the existing rules in the policy.


    我的配置:
    ...
    spec:
    ingress:
    - from:
        - podSelector:
            matchLabels:
            ~~access: allowed~~
    podSelector:
        matchLabels:
        ~~app: kk-app~~
    policyTypes:
    - Ingress
    - Egress


    Solution
    To edit the existing network policy use the following command:

    kubectl edit netpol netpol-ckad13-svcn



    Edit the policy as follows:

    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
    name: netpol-ckad13-svcn
    namespace: default
    spec:
    podSelector:
        matchLabels:
        app: kk-app
    policyTypes:
    - Ingress
    - Egress
    ingress:
    - from:
        - podSelector:
            matchLabels:
            tier: server
    #add the following in the manifest
        - podSelector:
            matchLabels:
            access: allowed

SECTION: APPLICATION ENVIRONMENT, CONFIGURATION and SECURITY
12. (難題) We have a Kubernetes namespace called ckad12-ctm-sa-aecs, which contains a service account and a pod. Your task is to modify the pod so that it uses the service account defined in the same namespace.

    Additionally, you need to ensure that the pod has access to the API credentials associated with the service account by enabling the automounting feature for the credentials.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    Solution:
    Here we will do two things:

    Remove automountServiceAccountToken: false field to enable automount of creds and specifying our service account as: serviceAccountName: ckad12-my-custom-sa-aecs
    student-node ~ ➜  kubectl config use-context cluster2
    Switched to context "cluster2".

    student-node ~ ➜  kubectl get pods -n ckad12-ctm-sa-aecs ckad12-ctm-nginx-aecs -o yaml > ckad-custom-sa.yaml

    student-node ~ ➜  vim ckad-custom-sa.yaml 

    student-node ~ ➜  cat ckad-custom-sa.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
    name: ckad12-ctm-nginx-aecs
    namespace: ckad12-ctm-sa-aecs
    spec:
    # automountServiceAccountToken: false  *removed to enable automount of creds
    containers:
    - image: nginx
        imagePullPolicy: Always
        name: nginx
    serviceAccountName: ckad12-my-custom-sa-aecs # using custom sa

    student-node ~ ➜  kubectl replace -f ckad-custom-sa.yaml --force 
    pod "ckad12-ctm-nginx-aecs" deleted
    pod/ckad12-ctm-nginx-aecs replaced



13. A pod named ckad-nginx-pod-aom is deployed and exposed with a service ckad-nginx-service-aom, but it seems the service is not configured properly and is not selecting the correct pod.

    Make the required changes to service and ensure the endpoint is configured for service.

    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

14. In the ckad14-sa-projected namespace, configure the ckad14-api-pod Pod to include a projected volume named vault-token.

    Mount the service account token to the container at /var/run/secrets/tokens, with an expiration time of 7000 seconds.

    Additionally, set the intended audience for the token to vault and path to vault-token.

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    Solution:

    student-node ~ ➜  kubectl config use-context cluster3
    Switched to context "cluster3".

    student-node ~ ➜  k get pod -n ckad14-sa-projected ckad14-api-pod -o yaml > ckad-pro-vol.yaml

    student-node ~ ➜  cat ckad-pro-vol.yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: ckad14-api-pod
    namespace: ckad14-sa-projected 
    spec:
    containers:
    - image: nginx
        imagePullPolicy: Always
        name: nginx
    .
    .
    .
    volumeMounts:                              # Added
        - mountPath: /var/run/secrets/tokens       # Added
        name: vault-token                        # Added
    .
    .
    .
    serviceAccount: ckad14-sa
    serviceAccountName: ckad14-sa
    volumes:
    - name: vault-token                   # Added
        projected:                          # Added
        sources:                          # Added
        - serviceAccountToken:            # Added
            path: vault-token             # Added
            expirationSeconds: 7000       # Added
            audience: vault               # Added

    student-node ~ ➜  k replace -f ckad-pro-vol.yaml --force 
    pod "ckad14-api-pod" deleted
    pod/ckad14-api-pod replaced


View the metrics (CPU and Memory) of the node cluster2-node01 and copy the output to the /root/node-metrics file in clustername,CPU and memory.

    For this question, please set the context to cluster2 by running:
    kubectl config use-context cluster2

    student-node ~ ➜  k describe node cluster2-node01 |grep -A6 "Capacity:"
    Capacity:
    cpu:                32
    ephemeral-storage:  1546531076Ki
    hugepages-1Gi:      0
    hugepages-2Mi:      0
    memory:             131992568Ki
    pods:               110

    student-node ~ ➜  k describe node cluster2-node01 |grep -A6 "Allocatable:"
    Allocatable:
    cpu:                32
    ephemeral-storage:  1504465429553
    hugepages-1Gi:      0
    hugepages-2Mi:      0
    memory:             131992568Ki
    pods:               110


    student-node ~ ✖ k get nodes
    NAME                    STATUS   ROLES                  AGE    VERSION
    cluster2-controlplane   Ready    control-plane,master   158m   v1.29.0+k3s1
    cluster2-node01         Ready    <none>                 158m   v1.29.0+k3s1

    student-node ~ ➜  k get nodes cluster2-node01 ^C

    student-node ~ ✖ echo "cluster2-node01" > /root/node-metrics

    student-node ~ ➜  k describe node cluster2-node01 |grep -A6 "Capacity:" >> /root/node-metrics 

    student-node ~ ➜  k describe node cluster2-node01 |grep -A6 "Allocatable:" >> /root/node-metrics 

    student-node ~ ➜  cat /root/node-metrics 
    cluster2-node01
    Capacity:
    cpu:                32
    ephemeral-storage:  1546531076Ki
    hugepages-1Gi:      0
    hugepages-2Mi:      0
    memory:             131992568Ki
    pods:               110
    Allocatable:
    cpu:                32
    ephemeral-storage:  1504465429553
    hugepages-1Gi:      0
    hugepages-2Mi:      0
    memory:             131992568Ki
    pods:               110

### SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE
17. A pod named ckad-nginx-pod-aom is deployed and exposed with a service ckad-nginx-service-aom, but it seems the service is not configured properly and is not selecting the correct pod.
    For this question, please set the context to cluster1 by running:
    kubectl config use-context cluster1

    Make the required changes to service and ensure the endpoint is configured for service.
    Check for service endpoint by using kubectl describe svc ckad-nginx-service-aom


    we can see below output as below

    kubectl describe svc ckad-nginx-service-aom
    Name:              ckad-nginx-service-aom
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          app=ngnix
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                10.43.134.231
    IPs:               10.43.134.231
    Port:              80-80  80/TCP
    TargetPort:        80/TCP
    Endpoints:         <none>
    Session Affinity:  None
    Events:            <none>


    we can see there endpoint value as none. Let's debug this we can see selector as app=ngnix . Lets check label value in pod.


    kubectl get pod ckad-nginx-pod-aom -o json | jq -r .metadata.labels
    "app": "nginx"

    we can see here selector is mis-spelled in service. so edit service and check for endpoint value.

    kubectl get ep ckad-nginx-service-aom
    NAME                     ENDPOINTS                   AGE
    ckad-nginx-service-aom   10.42.2.4:80,10.42.2.5:80   4m44s


### SECTION: SERVICE NETWORKING

20. Part I: Create a ClusterIP service .i.e. service-3421-svcn in the spectra-1267 ns which should expose the pods namely pod-23 and pod-21 with port set to 8080 and targetport to 80.

    Part II: Store the pod names and their ip addresses from the spectra-1267 ns at /root/pod_ips_cka05_svcn where the output is sorted by their IP's.

    Please ensure the format as shown below:
    POD_NAME        IP_ADDR
    pod-1           ip-1
    pod-3           ip-2
    pod-2           ip-3
    ...

    For this question, please set the context to cluster3 by running:
    kubectl config use-context cluster3

    Solution:
    Switching to cluster3:
    kubectl config use-context cluster3


    The easiest way to route traffic to a specific pod is by the use of labels and selectors . List the pods along with their labels:

    student-node ~ ➜  kubectl get pods --show-labels -n spectra-1267
    NAME     READY   STATUS    RESTARTS   AGE     LABELS
    pod-12   1/1     Running   0          5m21s   env=dev,mode=standard,type=external
    pod-34   1/1     Running   0          5m20s   env=dev,mode=standard,type=internal
    pod-43   1/1     Running   0          5m20s   env=prod,mode=exam,type=internal
    pod-23   1/1     Running   0          5m21s   env=dev,mode=exam,type=external
    pod-32   1/1     Running   0          5m20s   env=prod,mode=standard,type=internal
    pod-21   1/1     Running   0          5m20s   env=prod,mode=exam,type=external

    Looks like there are a lot of pods created to confuse us. But we are only concerned with the labels of pod-23 and pod-21.

    As we can see both the required pods have labels mode=exam,type=external in common. Let's confirm that using kubectl too:
    student-node ~ ➜  kubectl get pod -l mode=exam,type=external -n spectra-1267                                    
    NAME     READY   STATUS    RESTARTS   AGE
    pod-23   1/1     Running   0          9m18s
    pod-21   1/1     Running   0          9m17s


    Nice!! Now as we have figured out the labels, we can proceed further with the creation of the service:
    student-node ~ ➜  kubectl create service clusterip service-3421-svcn -n spectra-1267 --tcp=8080:80 --dry-run=client -o yaml > service-3421-svcn.yaml

    Now modify the service definition with selectors as required before applying to k8s cluster:
    student-node ~ ➜  cat service-3421-svcn.yaml 
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: null
    labels:
        app: service-3421-svcn
    name: service-3421-svcn
    namespace: spectra-1267
    spec:
    ports:
    - name: 8080-80
        port: 8080
        protocol: TCP
        targetPort: 80
    selector:
        app: service-3421-svcn  # delete 
        mode: exam    # add
        type: external  # add
    type: ClusterIP
    status:
    loadBalancer: {}


    Finally let's apply the service definition:

    student-node ~ ➜  kubectl apply -f service-3421-svcn.yaml
    service/service-3421 created

    student-node ~ ➜  k get ep service-3421-svcn -n spectra-1267
    NAME           ENDPOINTS                     AGE
    service-3421   10.42.0.15:80,10.42.0.17:80   52s


    To store all the pod name along with their IP's , we could use imperative command as shown below:

    student-node ~ ➜  kubectl get pods -n spectra-1267 -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP

    POD_NAME   IP_ADDR
    pod-12     10.42.0.18
    pod-23     10.42.0.19
    pod-34     10.42.0.20
    pod-21     10.42.0.21
    ...

    /# store the output to /root/pod_ips
    student-node ~ ➜  kubectl get pods -n spectra-1267 -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP > /root/pod_ips_cka05_svcn