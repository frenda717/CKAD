    k api-resources <Object> |grep <Object>

eg. kubectl api-resources
Output:
NAME                                SHORTNAMES   APIVERSION                        NAMESPACED   KIND
bindings                                         v1                                true         Binding
componentstatuses                   cs           v1                                false        ComponentStatus
configmaps                          cm           v1                                true         ConfigMap
endpoints                           ep           v1                                true         Endpoints
events                              ev           v1                                true         Event
limitranges                         limits       v1                                true         LimitRange
namespaces                          ns           v1                                false        Namespace
nodes                               no           v1                                false        Node
persistentvolumeclaims              pvc          v1                                true         PersistentVolumeClaim
persistentvolumes                   pv           v1                                false        PersistentVolume
pods                                po           v1                                true         Pod
podtemplates                                     v1                                true         PodTemplate
replicationcontrollers              rc           v1                                true         ReplicationController
resourcequotas                      quota        v1                                true         ResourceQuota
secrets                                          v1                                true         Secret
serviceaccounts                     sa           v1                                true         ServiceAccount
services                            svc          v1                                true         Service
mutatingwebhookconfigurations                    admissionregistration.k8s.io/v1   false        MutatingWebhookConfiguration
validatingadmissionpolicies                      admissionregistration.k8s.io/v1   false        ValidatingAdmissionPolicy
validatingadmissionpolicybindings                admissionregistration.k8s.io/v1   false        ValidatingAdmissionPolicyBinding
validatingwebhookconfigurations                  admissionregistration.k8s.io/v1   false        ValidatingWebhookConfiguration
customresourcedefinitions           crd,crds     apiextensions.k8s.io/v1           false        CustomResourceDefinition
apiservices                                      apiregistration.k8s.io/v1         false        APIService
controllerrevisions                              apps/v1                           true         ControllerRevision
daemonsets                          ds           apps/v1                           true         DaemonSet
deployments                         deploy       apps/v1                           true         Deployment
replicasets                         rs           apps/v1                           true         ReplicaSet
statefulsets                        sts          apps/v1                           true         StatefulSet
selfsubjectreviews                               authentication.k8s.io/v1          false        SelfSubjectReview
tokenreviews                                     authentication.k8s.io/v1          false        TokenReview
localsubjectaccessreviews                        authorization.k8s.io/v1           true         LocalSubjectAccessReview
selfsubjectaccessreviews                         authorization.k8s.io/v1           false        SelfSubjectAccessReview
selfsubjectrulesreviews                          authorization.k8s.io/v1           false        SelfSubjectRulesReview
subjectaccessreviews                             authorization.k8s.io/v1           false        SubjectAccessReview
horizontalpodautoscalers            hpa          autoscaling/v2                    true         HorizontalPodAutoscaler
cronjobs                            cj           batch/v1                          true         CronJob
jobs                                             batch/v1                          true         Job
certificatesigningrequests          csr          certificates.k8s.io/v1            false        CertificateSigningRequest
leases                                           coordination.k8s.io/v1            true         Lease
endpointslices                                   discovery.k8s.io/v1               true         EndpointSlice
events                              ev           events.k8s.io/v1                  true         Event
flowschemas                                      flowcontrol.apiserver.k8s.io/v1   false        FlowSchema
prioritylevelconfigurations                      flowcontrol.apiserver.k8s.io/v1   false        PriorityLevelConfiguration
helmchartconfigs                                 helm.cattle.io/v1                 true         HelmChartConfig
helmcharts                                       helm.cattle.io/v1                 true         HelmChart
addons                                           k3s.cattle.io/v1                  true         Addon
etcdsnapshotfiles                                k3s.cattle.io/v1                  false        ETCDSnapshotFile
nodes                                            metrics.k8s.io/v1beta1            false        NodeMetrics
pods                                             metrics.k8s.io/v1beta1            true         PodMetrics
ingressclasses                                   networking.k8s.io/v1              false        IngressClass
ingresses                           ing          networking.k8s.io/v1              true         Ingress
networkpolicies                     netpol       networking.k8s.io/v1              true         NetworkPolicy
runtimeclasses                                   node.k8s.io/v1                    false        RuntimeClass
poddisruptionbudgets                pdb          policy/v1                         true         PodDisruptionBudget
clusterrolebindings                              rbac.authorization.k8s.io/v1      false        ClusterRoleBinding
clusterroles                                     rbac.authorization.k8s.io/v1      false        ClusterRole
rolebindings                                     rbac.authorization.k8s.io/v1      true         RoleBinding
roles                                            rbac.authorization.k8s.io/v1      true         Role
priorityclasses                     pc           scheduling.k8s.io/v1              false        PriorityClass
csidrivers                                       storage.k8s.io/v1                 false        CSIDriver
csinodes                                         storage.k8s.io/v1                 false        CSINode
csistoragecapacities                             storage.k8s.io/v1                 true         CSIStorageCapacity
storageclasses                      sc           storage.k8s.io/v1                 false        StorageClass
volumeattachments                                storage.k8s.io/v1                 false        VolumeAttachment
ingressroutes                                    traefik.containo.us/v1alpha1      true         IngressRoute
ingressroutetcps                                 traefik.containo.us/v1alpha1      true         IngressRouteTCP
ingressrouteudps                                 traefik.containo.us/v1alpha1      true         IngressRouteUDP
middlewares                                      traefik.containo.us/v1alpha1      true         Middleware
middlewaretcps                                   traefik.containo.us/v1alpha1      true         MiddlewareTCP
serverstransports                                traefik.containo.us/v1alpha1      true         ServersTransport
tlsoptions                                       traefik.containo.us/v1alpha1      true         TLSOption
tlsstores                                        traefik.containo.us/v1alpha1      true         TLSStore
traefikservices                                  traefik.containo.us/v1alpha1      true         TraefikService
ingressroutes                                    traefik.io/v1alpha1               true         IngressRoute
ingressroutetcps                                 traefik.io/v1alpha1               true         IngressRouteTCP
ingressrouteudps                                 traefik.io/v1alpha1               true         IngressRouteUDP
middlewares                                      traefik.io/v1alpha1               true         Middleware
middlewaretcps                                   traefik.io/v1alpha1               true         MiddlewareTCP
serverstransports                                traefik.io/v1alpha1               true         ServersTransport
serverstransporttcps                             traefik.io/v1alpha1               true         ServersTransportTCP
tlsoptions                                       traefik.io/v1alpha1               true         TLSOption
tlsstores                                        traefik.io/v1alpha1               true         TLSStore
traefikservices                                  traefik.io/v1alpha1               true         TraefikService

Pod


1. Find where the pod belongs to which node: (2 ways)
   a. 
        
        kubectl get pods: check the column "Node"
   b. 
        
        kubectl get pods **-o wide**:

2. To see the details of a pod (eg. Image, Node...):
   kubectl get pods (Receive all pods name)
   
        kubectl describe pod (pod name 1) (pod name 2)

3. How many containers are part of the pod webapp?
    
        k get pods
    controlplane ~ ➜  k get pods
    NAME            READY   STATUS             RESTARTS   AGE
    newpods-dg5n6   1/1     Running            0          15m
    newpods-fm774   1/1     Running            0          15m
    newpods-ldhm4   1/1     Running            0          15m
    pod             1/1     Running            0          13m
    webapp          1/2     ImagePullBackOff   0          3m14s

        k describe pod webapp

    ...
    Containers:
    nginx:
        Container ID:   containerd://83af519b11589dc3f848b2d39b4d2169efc0d9245c663bda9806e3fa1e98eaad
        Image:          nginx
        Image ID:       docker.io/library/nginx@sha256:fb197595ebe76b9c0c14ab68159fd3c08bd067ec62300583543f0ebda353b5be
        Port:           <none>
        Host Port:      <none>
        State:          Running
        Started:      Mon, 23 Dec 2024 13:26:51 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ntvlq (ro)
    agentx:
        Container ID:   
        Image:          agentx
        Image ID:       
        Port:           <none>
        Host Port:      <none>
        State:          Waiting
        Reason:       ErrImagePull
        Ready:          False
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ntvlq (ro)
    Conditions:
        ....
        (Ignore)

    
    Answer: 2 (define 2 containers called nginx & agentx)

4. What does the READY column in the output of the kubectl get pods command indicate?
    controlplane ~ ➜  k get pods
    NAME            READY   STATUS             RESTARTS       AGE
    newpods-dg5n6   1/1     Running            1 (6m5s ago)   22m
    newpods-fm774   1/1     Running            1 (6m5s ago)   22m
    newpods-ldhm4   1/1     Running            1 (6m5s ago)   22m
    pod             1/1     Running            0              20m
    webapp          1/2     ImagePullBackOff   0              10m

    Answer: Running Containers in POD / Total Containers in POD`


5. Create a new pod with the name redis and with the image redis123: (2 ways)
   (Recommand to use a pod-definition YAML file.)
    a. create a YAML file (manualy, type: apiVersion, kinds, metadata and spec...)
        
        1. create a redis.yaml file, and type as below:
        vim redis.yaml

        apiVersion: v1
        kind: Pod
        metadata:
        name: redis (======>Make the pod name as redis)
        labels:
            app: myapp (define by yourself)
            type: front-end (define by yourself)

        spec:
        containers:
            - name: nginx-container (define by yourself)
            image: redis123 (======> and Make the pod imgage as redis123)

        kubectl create -f redis.yaml
        (pod/myapp-pod created)
    b. kubectl run redis -image=redis123 --**dry-run -o yaml**:

        kubectl run redis --image=redis123 --**dry-run=client -o yaml** > redis.yaml
        kubectl create -f redis.yaml

6. Now change the image on this pod to redis. Once done, the pod should be in a running state.
    a. manualy change the YAML file and then use apply command to update:
            
            kubectl apply -f redis.yaml
            kubectl get pods
    controlplane ~ ➜  k apply -f redis.yaml
    pod/redis configured

    b. directly edit the generated pod:
        
        kubectl edit pod redis (Modify the generated pod directly!!)
    controlplane ~ ➜  k edit pod redis
    pod/redis edited

    """

"""
A Note on Editing Existing Pods
In any of the practical quizzes, if you are asked to edit an existing POD, please note the following:

If you are given a pod definition file, edit that file and use it to create a new pod.

If you are not given a pod definition file, you may extract the definition to a file using the below command:

kubectl get pod <pod-name> -o yaml > pod-definition.yaml

Then edit the file to make the necessary changes, delete, and re-create the pod.

To modify the properties of the pod, you can utilize the kubectl edit pod <pod-name> command. Please note that only the properties listed below are editable.

spec.containers[*].image

spec.initContainers[*].image

spec.activeDeadlineSeconds

spec.tolerations

spec.terminationGracePeriodSeconds


"""


ReplicaSet

1. How many PODS are DESIRED in the new-replica-set?
   
        k get replicaset new-replica-set (= k get rs new-replica-set)
        k describe replicaset new-replica-set (= k describe rs new-replica-set)
    
    controlplane ~ ➜  k get rs new-replica-set 
    NAME              DESIRED   CURRENT   READY   AGE
    new-replica-set   4         4         0       2m9s

    controlplane ~ ➜  k describe rs new-replica-set 
    Name:         new-replica-set
    Namespace:    default
    Selector:     name=busybox-pod
    Labels:       <none>
    Annotations:  <none>
    Replicas:     4 current / 4 desired (======> To know how many pods are under this replicaset, check the "desired" number, 4)
    Pods Status:  0 Running / 4 Waiting / 0 Succeeded / 0 Failed
    Pod Template:
    Labels:  name=busybox-pod
    Containers:
    busybox-container:
        Image:      busybox777
        Port:       <none>
        Host Port:  <none>
        Command:
        sh
        -c
        echo Hello Kubernetes! && sleep 3600
        Environment:   <none>
        Mounts:        <none>
    Volumes:         <none>
    Node-Selectors:  <none>
    Tolerations:     <none>
    Events:
    ....
    (Ignore)

    Answer: 4 pods


2. How many PODs are READY in the new-replica-set?

    WROWN ANSWER:
        k describe rs new-replica-set (x)
        "describe" 有點像是用來查看該資源底下的子資源狀態，像是用k describe pod pod01 -> 可以用來查看含有幾個container

    ... 
    (Ignore)
    Events: (Check from the events is WRWON!!!!!)
    Type    Reason            Age    From                   Message
    ----    ------            ----   ----                   -------
    Normal  SuccessfulCreate  2m23s  replicaset-controller  Created pod: new-replica-set-59vnm
    Normal  SuccessfulCreate  2m23s  replicaset-controller  Created pod: new-replica-set-x5g7j
    Normal  SuccessfulCreate  2m23s  replicaset-controller  Created pod: new-replica-set-8jvkx
    Normal  SuccessfulCreate  2m23s  replicaset-controller  Created pod: new-replica-set-sjxt8

    Answer: 4 (WRWON!!!!)

    CORRECT ANSWER:

        k get rs new-replica-set
        (欲看該replicaset 本身的狀態，使用k get)

    controlplane ~ ✖ k get rs new-replica-set 
    NAME              DESIRED   CURRENT   READY   AGE
    new-replica-set   4         4         0       9m42s

    Answer: 0 (CORRECT!!!)

3. Why do you think the PODs are not ready?
    If the replicaset cannot run smoothly, check the inner pod status and also the replicaset configuration file 

        Check the configuration file
        k describe pods

    Events:
    Type     Reason     Age                 From               Message
    ----     ------     ----                ----               -------
    Normal   Scheduled  15m                 default-scheduler  Successfully assigned default/new-replica-set-x5g7j to controlplane
    Normal   Pulling    13m (x4 over 15m)   kubelet            Pulling image "busybox777"
    Warning  Failed     13m (x4 over 15m)   kubelet            Failed to pull image "busybox777": failed to pull and unpack image "docker.io/library/busybox777:latest": failed to resolve reference "docker.io/library/busybox777:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
    Warning  Failed     13m (x4 over 15m)   kubelet            Error: ErrImagePull
    Warning  Failed     13m (x6 over 15m)   kubelet            Error: ImagePullBackOff
    Normal   BackOff    10s (x64 over 15m)  kubelet            Back-off pulling image "busybox777"

    Answer: The image BUSYBOX777 doesn't exist


4. Fix the original replica set new-replica-set to use the correct busybox image.
   
        K edit rs new-replica-set
        k delete pod (pod name))
   controlplane ~ ➜  k edit rs new-replica-set 
    replicaset.apps/new-replica-set edited

    controlplane ~ ➜  k delete pods
    error: resource(s) were provided, but no name was specified

    controlplane ~ ✖ k get pods 
    NAME                    READY   STATUS             RESTARTS   AGE
    new-replica-set-4xmdm   0/1     ImagePullBackOff   0          13m
    new-replica-set-9m76c   0/1     ImagePullBackOff   0          13m
    new-replica-set-k6dkn   0/1     ImagePullBackOff   0          10m
    new-replica-set-qgszd   0/1     ImagePullBackOff   0          13m

    controlplane ~ ➜  k delete pod new-replica-set-*
    \Error from server (NotFound): pods "new-replica-set-*" not found

    controlplane ~ ✖ k delete pod new-replica-set-4xmdm new-replica-set-9m76c new-replica-set-k6dkn new-replica-set-qgszd 
    pod "new-replica-set-4xmdm" deleted
    pod "new-replica-set-9m76c" deleted
    pod "new-replica-set-k6dkn" deleted
    pod "new-replica-set-qgszd" deleted

    controlplane ~ ➜  k get pods
    NAME                    READY   STATUS    RESTARTS   AGE
    new-replica-set-229sm   1/1     Running   0          6s
    new-replica-set-565cf   1/1     Running   0          6s
    new-replica-set-qpjr6   1/1     Running   0          6s
    new-replica-set-z6g92   1/1     Running   0          6s

5. Scale the RepicaSet to 5 Pods:

    controlplane ~ ✖ kubectl scale --replicas=5 rs/new-replica-set 
    replicaset.apps/new-replica-set scaled
    controlplane ~ ➜ ** k scale --help**
    
    a. kubectl scale:

        k scale --replicas=5

    controlplane ~ ➜  kubectl scale --replicas=5 
    error: You must provide one or more resources by argument or filename.
    Example resource specifications include:
    '-f rsrc.yaml'
    '--filename=rsrc.json'
    '<resource> <name>'
    '<resource>'

        
    controlplane ~ ➜  kubectl scale replicaset new-replica-set --replicas=3 replicaset.apps/new-replica-set scaled

    b. kubectl edit replicaset new-replica-set, vi replica column 

        k edit rs new-replicaset


6. Now scale the ReplicaSet down to 2 PODs.


Use the kubectl scale command or edit the replicaset using kubectl edit replicaset.

controlplane ~ ➜  kubectl scale replicaset new-replica-set --replicas=2
replicaset.apps/new-replica-set scaled


6. Create a ReplicaSet using the replicaset-definition-1.yaml file located at /root/.
    controlplane ~ ➜  k create rs -f replicaset-definition-1.yaml 
    error: Unexpected args: [rs]
    See 'kubectl create -h' for help and examples

    Above shows that it should be something wrong in the yaml file. 
    vim replicaset-definition-1.yaml

    Run the command: You can check for apiVersion of replicaset by command 
    
        kubectl api-resources | grep replicaset
        
        kubectl explain replicaset | grep VERSION 
        
        and correct the apiVersion for ReplicaSet.

    Then run the command: kubectl create -f /root/replicaset-definition-1.yaml

    controlplane ~ ➜  vim  replicaset-definition-1.yaml 
    controlplane ~ ➜  kubectl explain replicaset | grep VERSION
    VERSION:    v1
    controlplane ~ ➜  kubectl create -f /root/replicaset-definition-1.yaml
    replicaset.apps/replicaset-1 created


Deployment

1. What is the image used to create the pods in the new deployment?
    
        k get deploy/deployments
        k describe deployments frontend-deployment | grep  Image

2. Create a new Deployment using the deployment-definition-1.yaml file located at /root/.

        k create -f <yaml file> -> Check the result

    controlplane ~ ➜  k create -f deployment-definition-1.yaml 
    Error from server (BadRequest): error when creating "deployment-definition-1.yaml": deployment in version "v1" cannot be handled as a Deployment: no kind "deployment" is registered for version "apps/v1" in scheme "k8s.io/apimachinery@v1.31.0-k3s3/pkg/runtime/scheme.go:100"

    controlplane ~ ✖ vim deployment-definition-1.yaml 

    controlplane ~ ➜  k create -f deployment-definition-1.yaml 
    deployment.apps/deployment-1 created

3. Create a new Deployment with the below attributes using your own deployment definition file. (2 ways)
    Name: httpd-frontend;
    Replicas: 3;
    Image: httpd:2.4-alpine

    a. 

        k create -f deploy.yaml (new definition file)

    controlplane ~ ➜  ls
    deployment-definition-1.yaml  sample.yaml

    controlplane ~ ➜  cat deployment-definition-1.yaml > deploy.yaml

    controlplane ~ ➜  vim deploy.yaml 

    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: httpd-frontend
    spec:
    replicas: 3
    selector:
        matchLabels:
        name: httpd-frontend
    template:
        metadata:
        labels:
            name: httpd-frontend
        spec:
        containers:
        - name: httpd-frontend
            image: httpd:2.4-alpine
            command:
            - sh
            - "-c"
            - echo Hello Kubernetes! && sleep 3600
  
    controlplane ~ ➜  k create -f deploy.yaml 
    deployment.apps/httpd-frontend created

        k get deployment (or k get deploy)

    controlplane ~ ➜  k get deploy 
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    deployment-1     0/2     2            0           8m47s
    httpd-frontend   3/3     3            3           14s
    new-deployment   0/3     3            0           6m25s

    b. By the given conditions, use k help to create a deployment directly by command

        k create deployment/deploy --help
        (Choose the most fitted command line from Examples column)
        k create deploy <name> --image=<>  --replicas=<>
    controlplane ~ ✖ k create deploy httpd-backend --image=httpd:2.4-alpine --replicas=3
    deployment.apps/httpd-backend created


    **CAUTION**:
    k create deployment/deploy -h/--help 跟
    k create deployments -h/help  的輸出結果不一樣!!!!!


        k create deployment -h/--help:
    controlplane ~ ✖ k create deployment -h
    Create a deployment with the specified name.

    Aliases:
    deployment, deploy

    Examples:
    # Create a deployment named my-dep that runs the busybox image
    kubectl create deployment my-dep --image=busybox
    
    # Create a deployment with a command
    kubectl create deployment my-dep --image=busybox -- date
    
    # Create a deployment named my-dep that runs the nginx image with 3 replicas
    kubectl create deployment my-dep --image=nginx --replicas=3    **output detail usage !!!**
    
    # Create a deployment named my-dep that runs the busybox image and expose port 5701
    kubectl create deployment my-dep --image=busybox --port=5701
    
    # Create a deployment named my-dep that runs multiple containers
    kubectl create deployment my-dep --image=busybox:latest --image=ubuntu:latest --image=nginx

    Options: ......

        k create deployment**s** -help :

    controlplane ~ ➜  k create deployments --help
    Create a resource from a file or from stdin.

    JSON and YAML formats are accepted.

    Examples:
    # Create a pod using the data in pod.json
    kubectl create -f ./pod.json
    
    # Create a pod based on the JSON passed into stdin
    cat pod.json | kubectl create -f -      **didn't output detail usage !!!**
    
    # Edit the data in registry.yaml in JSON then create the resource using the edited data
    kubectl create -f registry.yaml --edit -o json



Namespace
1. How many Namespaces exist on the system?
        
        k get ns

    controlplane ~ ✖ k get ns
    NAME              STATUS   AGE
    default           Active   6m47s
    dev               Active   3m46s
    finance           Active   3m46s
    kube-node-lease   Active   6m47s
    kube-public       Active   6m47s
    kube-system       Active   6m47s
    manufacturing     Active   3m46s
    marketing         Active   3m46s
    prod              Active   3m46s
    research          Active   3m46s

    Answer: 10

2. How many pods exist in the research namespace?

        k get pods -h/--help
      
    ...
    
    # List all pods existing in all namespaces
    kubectl get pods --all-namespaces
    
        k get pods --namespace=research
    
    # List all deployments in namespace 'backend'
    kubectl get deployments.apps --namespace backend

        k get deployments/deploy --namespace=research / --namespace research

    controlplane ~ ✖ kubectl get pods --namespace=research  or -n=research
    NAME    READY   STATUS             RESTARTS       AGE
    dna-1   0/1     CrashLoopBackOff   6 (4m2s ago)   9m49s
    dna-2   0/1     CrashLoopBackOff   6 (4m2s ago)   9m49s

    Answer: 2

    錯誤示範:

    controlplane ~ ➜  kubectl get deployments.apps --namespace research 
    No resources found in research namespace. (因為Pod 並非由 Deployment 管理，可能是直接創建的或由其他控制器管理。)


3. Create a POD in the finance namespace.
    Use the spec given below.
    Name: redis
    Image name: redis

        k create redis -h (查無結果)
        k run redis -h 
    Examples:
    # Start a nginx pod
    kubectl run nginx --image=nginx
    ...

        kubectl options (List options common to all kubectl commands )
    
    -n, --namespace='':
    If present, the namespace scope for this CLI request
    ...

        k run redis --image=redis -n finance

    controlplane ~ ➜  kubectl run redis --image=redis -n finance
    pod/redis created

    controlplane ~ ➜  kubectl run nginx  --image=nginx -n finance
    pod/nginx created

    controlplane ~ ➜  kubectl get pods --namespace=finance
    NAME      READY   STATUS    RESTARTS   AGE
    nginx     1/1     Running   0          10s
    payroll   1/1     Running   0          12m
    redis     1/1     Running   0          23s

4. Which namespace has the blue pod in it?
    
        k get pods --all-namespaces

    controlplane ~ ➜  k get pods
    No resources found in default namespace.

    controlplane ~ ➜  k get ns
    NAME              STATUS   AGE
    default           Active   18m
    dev               Active   15m
    finance           Active   15m
    kube-node-lease   Active   18m
    kube-public       Active   18m
    kube-system       Active   18m
    manufacturing     Active   15m
    marketing         Active   15m
    prod              Active   15m
    research          Active   15m

    controlplane ~ ➜  k get pods --all-namespaces
    NAMESPACE       NAME                                      READY   STATUS             RESTARTS        AGE
    dev             redis-db                                  1/1     Running            0               15m
    finance         nginx                                     1/1     Running            0               2m55s
    finance         payroll                                   1/1     Running            0               15m
    finance         redis                                     1/1     Running            0               3m8s
    kube-system     coredns-5dd589bf46-k7wmn                  1/1     Running            0               18m
    kube-system     helm-install-traefik-499js                0/1     Completed          1               18m
    kube-system     helm-install-traefik-crd-fbc2c            0/1     Completed          0               18m
    kube-system     local-path-provisioner-846b9dcb6c-bn6t4   1/1     Running            0               18m
    kube-system     metrics-server-5dc58b587c-grb7j           1/1     Running            0               18m
    kube-system     svclb-traefik-c05f8a9b-cq7sh              2/2     Running            0               18m
    kube-system     traefik-7f4b44bf74-jx9zz                  1/1     Running            0               18m
    manufacturing   red-app                                   1/1     Running            0               15m
    marketing       blue                                      1/1     Running            0               15m
    marketing       redis-db                                  1/1     Running            0               15m
    research        dna-1                                     0/1     CrashLoopBackOff   7 (4m38s ago)   15m
    research        dna-2                                     0/1     CrashLoopBackOff   7 (4m38s ago)   15m

    Answer: marketing


5. What DNS name should the Blue application use to access the database db-service in its own namespace - marketing?

    In the same namespace: the 2 service can connet to each other by using service name directly.

    You can try it in the web application UI. Use port 6379.
    controlplane ~ ➜  k get ns
    NAME              STATUS   AGE
    default           Active   47m
    dev               Active   45m
    finance           Active   45m
    kube-node-lease   Active   47m
    kube-public       Active   47m
    kube-system       Active   47m
    manufacturing     Active   45m
    marketing         Active   45m
    prod              Active   45m
    research          Active   45m

    controlplane ~ ➜  k get service -n=marketing
    NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    blue-service   NodePort   10.43.76.105    <none>        8080:30082/TCP   45m
    db-service     NodePort   10.43.118.230   <none>        6379:30585/TCP   45m

    controlplane ~ ➜  k get svc -n=marketing
    NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    blue-service   NodePort   10.43.76.105    <none>        8080:30082/TCP   45m
    db-service     NodePort   10.43.118.230   <none>        6379:30585/TCP   45m

    (db-service & blue-service are under a same namespace, hence can connect to each other directly by service name & port)

    Answer: db-service

6. What DNS name should the Blue application use to access the database db-service in the dev namespace?
You can try it in the web application UI. Use port 6379.

To access service from different namespace, we need to directly reference the full namespace & service name

controlplane ~ ➜  k get svc -n=marketing
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
blue-service   NodePort   10.43.76.105    <none>        8080:30082/TCP   45m
db-service     NodePort   10.43.118.230   <none>        6379:30585/TCP   45m

controlplane ~ ➜  k get svc -n=dev
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
db-service   ClusterIP   10.43.170.37   <none>        6379/TCP   50m


Ansewr: db-service.dev.svc.cluster.local  <service-name>.<namespace-name>.svc.cluster.local