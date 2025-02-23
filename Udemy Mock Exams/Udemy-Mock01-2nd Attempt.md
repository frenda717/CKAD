Score: 43 % + 3題不明原因被判斷為錯題 大約55%
本次空題: 07 & 10 & 17

03. (粗心) 第一次Attempt是對的，這題可以忽略

    In the ckad-job namespace, create a cronjob named simple-node-job to run every 30 minutes to list all the running processes inside a container that used node image (the command needs to be run in a shell).

    In Unix-based operating systems, ps -eaf can be use to list all the running processes.

    Solution:

    apiVersion: batch/v1
    kind: CronJob
    metadata:
    name: simple-node-job
    namespace: ckad-job
    spec:
    schedule: "*/30 * * * *"
    jobTemplate:
        spec:
        template:
            spec:
            containers:
            - name: simple-node-job
                image: node
                imagePullPolicy: IfNotPresent
                command:
                - /bin/sh
                - -c
                - ps -eaf
            restartPolicy: OnFailure


    我的配置：
    spec:
          containers:
          - command:
            - ps -eaf
            - -n
            - ckad-job

    （應該是粗心）

05. (labels 沒有同步改)
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    labels:
        type-one: blue
    name: blue-apd
    spec:
    replicas: 7
    selector:
        matchLabels:
        type-one: blue
        version: v1
    template:
        metadata:
        labels:
            version: v1
            type-one: blue
        spec:
        containers:
            - image: kodekloud/webapp-color:v1
            name: blue-apd

    我的配置：---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    **labels:**
        type-one: blue　　　＃這裡的labels沒有改
    name: blue-apd
    spec:
    replicas: 7
    selector:
        matchLabels:
        type-one: blue
        version: v1
    template:
        metadata:
        labels:
            version: v1
            type-one: blue
        spec:
        containers:
            - image: kodekloud/webapp-color:v1
            name: blue-apd



    apiVersion: apps/v1
    kind: Deployment
    metadata:
    labels:
        type-two: green
    name: green-apd # 同上
    spec:
    replicas: 3
    selector:
        matchLabels:
        type-two: green
        version: v1
    template:
        metadata:
        labels:
            type-two: green
            version: v1
        spec:
        containers:
            - image: kodekloud/webapp-color:v2
            name: green-apd


    (Service 我配置正確)
    apiVersion: v1
    kind: Service
    metadata:
    labels:
        app: route-apd-svc
    name: route-apd-svc
    spec:
    type: NodePort
    ports:
        - port: 8080
        protocol: TCP
        targetPort: 8080
    selector:
        version: v1


07. (空題)
    
    student-node ~ ➜  cat /opt/
    .init/                      svc01-install-webserver.sh  webapp-color-apd/

    student-node ~ ➜  ls /opt/.init/

    student-node ~ ➜  cat /opt/webapp-color-apd/
    Chart.yaml   templates/   values.yaml  

    student-node ~ ➜  cat /opt/webapp-color-apd/Chart.yaml 
    apiVersion: v2
    name: webapp-color-apd
    description: A Helm chart for Webapp Color Application
    type: application

    # This is the chart version. This version number should be incremented each time you make changes
    # to the chart and its templates, including the app version.
    # Versions are expected to follow Semantic Versioning (https://semver.org/)
    version: 0.1.0

    # This is the version number of the application being deployed. This version number should be
    # incremented each time you make changes to the application. Versions are not expected to
    # follow Semantic Versioning. They should reflect the version the application is using.
    # It is reecommended to use it with quotes.
    appVersion: "v1"

    student-node ~ ➜  cat /opt/webapp-color-apd/values.yaml 
    # Default values for webapp-color.
    # This is a YAML-formatted file.
    # Declare variables to be passed into your templates.

    replicaCount: 0

    image:
    repository: kodekloud/webapp-color
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
    tag: "v1"

    imagePullSecrets: []

    name: webapp-color-apd

    serviceAccount:
    # Specifies whether a service account should be created
    create: false
    # Annotations to add to the service account
    annotations: {}
    # The name of the service account to use.
    # If not set and create is true, a name is generated using the fullname template
    name: webapp-sa-apd
    labels:
        app: webapp-color-apd

    podAnnotations: {}

    podSecurityContext: {}
    # fsGroup: 2000

    securityContext: {}
    # capabilities:
    #   drop:
    #   - ALL
    # readOnlyRootFilesystem: true
    # runAsNonRoot: true
    # runAsUser: 1000

    service:
    name: webapp-color-svc
    type: ClusterIP
    port: 8080


    environment: development


    configMap:
    name: webapp-configmap-apd

    resources: {}
    # We usually recommend not to specify default resources and to leave this as a conscious
    # choice for the user. This also increases chances charts run on environments with little
    # resources, such as Minikube. If you do want to specify resources, uncomment the following
    # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
    # limits:
    #   cpu: 100m
    #   memory: 128Mi
    # requests:
    #   cpu: 100m
    #   memory: 128Mi
    nodeSelector: {}

    tolerations: []

    affinity: {}



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


    Deployment apiVersion needs to be correctly written. It should be apiVersion: apps/v1.

    In the service YAML, there is a typo in the template variable {{ .Values.service.name }} because of that, it's not able to reference the value of the name field defined in the values.yaml file for the Kubernetes service that is being created or updated.


    Now run the following command to install the helm chart in the frontend-apd namespace: -

    helm install webapp-color-apd -n frontend-apd ./webapp-color-apd



    Use the helm ls command to list the release deployed using helm.
    helm ls -n frontend-apd

08. (粗心)

    To use the cluster 3, switch the context using:

    kubectl config use-context cluster3

    To expose the pod pod22-ckad-svcn at port 6335, we can use the following imperative command.

    kubectl expose pod pod22-ckad-svcn --name=svc22-ckad-svcn --port=6335

    It will create a service with name svc22-ckad-svcn and pod will be exposed at port 6335.

    我的配置: service name錯了

    student-node ~ ➜  k get svc
    NAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    external-webserver-ckad01-svcn   ClusterIP   172.20.34.95     <none>        80/TCP     91m
    kubernetes                       ClusterIP   172.20.0.1       <none>        443/TCP    129m
    **pod22-ckad-svcn**                 ClusterIP   172.20.205.175   <none>        6335/TCP   93m
    route-apd-svc                    ClusterIP   172.20.118.58    <none>        8080/TCP   99m

09. (不明原因被判斷不過)

10. (空題)
    Solution:
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

    此處的$IP_ADDR不知如何來

13. (粗心 or 文件名有問題) 

    Solution:

    student-node ~ ➜ kubectl config use-context cluster1
    Switched to context "cluster1".

    student-node ~ ➜  cat << EOF | kubectl apply -f -
    apiVersion: v1
    kind: Pod
    metadata:
    name: ckad17-qos-aecs-3
    namespace: ckad17-nqoss-aecs
    spec:
    containers:
    - name: ckad17-qos-ctr-3-aecs
        image: nginx
        resources:
        limits:
            memory: "200Mi"
        requests:
            memory: "100Mi"
    EOF

    pod/ckad17-qos-aecs-3 created

    student-node ~ ➜  kubectl --namespace=ckad17-nqoss-aecs get pod --output=custom-columns="NAME:.metadata.name,QOS:.status.qosClass"
    NAME                QOS
    ckad17-qos-aecs-1   BestEffort
    ckad17-qos-aecs-2   Guaranteed
    ckad17-qos-aecs-3   Burstable

    student-node ~ ➜  kubectl --namespace=ckad17-nqoss-aecs get pod --output=custom-columns="NAME:.metadata.name,QOS:.status.qosClass" > /root/qos_status_aecs


    student-node ~ ➜  cat /root/qos_status-aecs
    cat: /root/qos_status-aecs: No such file or directory

    (可能是文件名出問題導致判斷不成功)

17. (空題)

    Solution:

    student-node ~ ➜  kubectl config use-context cluster2
    Switched to context "cluster2".

    student-node ~ ➜  kubectl create namespace ckad16-rqc-ns
    namespace/ckad16-rqc-ns created

    student-node ~ ➜  cat << EOF | kubectl apply -f -
    apiVersion: v1
    kind: ResourceQuota
    metadata:
    name: ckad16-rqc
    namespace: ckad16-rqc-ns
    spec:
    hard:
        resourcequotas: "1"
    EOF

    resourcequota/ckad16-rqc created

    student-node ~ ➜  k get resourcequotas -n ckad16-rqc-ns
    NAME              AGE   REQUEST               LIMIT
    ckad16-rqc   20s   resourcequotas: 1/1   


    (我找不到resourcequotas 參數)
    student-node ~ ➜  k explain ResourceQuota.spec.hard
    KIND:       ResourceQuota
    VERSION:    v1

    FIELD: hard <map[string]Quantity>


    DESCRIPTION:
        hard is the set of desired hard limits for each named resource. More info:
        https://kubernetes.io/docs/concepts/policy/resource-quotas/
        Quantity is a fixed-point representation of a number. It provides convenient
        marshaling/unmarshaling in JSON and YAML, in addition to String() and
        AsInt64() accessors.
        
        The serialization format is:
        
        ``` <quantity>        ::= <signedNumber><suffix>
        
            (Note that <suffix> may be empty, from the "" case in <decimalSI>.)
        
        <digit>           ::= 0 | 1 | ... | 9 <digits>          ::= <digit> |
        <digit><digits> <number>          ::= <digits> | <digits>.<digits> |
        <digits>. | .<digits> <sign>            ::= "+" | "-" <signedNumber>    ::=
        <number> | <sign><number> <suffix>          ::= <binarySI> |
        <decimalExponent> | <decimalSI> <binarySI>        ::= Ki | Mi | Gi | Ti | Pi
        | Ei
        
            (International System of units; See:
        http://physics.nist.gov/cuu/Units/binary.html)
        
        <decimalSI>       ::= m | "" | k | M | G | T | P | E
        
            (Note that 1024 = 1Ki but 1000 = 1k; I didn't choose the capitalization.)
        
        <decimalExponent> ::= "e" <signedNumber> | "E" <signedNumber> ```
        
        No matter which of the three exponent forms is used, no quantity may
        represent a number greater than 2^63-1 in magnitude, nor may it have more
        than 3 decimal places. Numbers larger or more precise will be capped or
        rounded up. (E.g.: 0.1m will rounded up to 1m.) This may be extended in the
        future if we require larger or smaller quantities.
        
        When a Quantity is parsed from a string, it will remember the type of suffix
        it had, and will use the same type again when it is serialized.
        
        Before serializing, Quantity will be put in "canonical form". This means
        that Exponent/suffix will be adjusted up or down (with a corresponding
        increase or decrease in Mantissa) such that:
        
        - No precision is lost - No fractional digits will be emitted - The exponent
        (or suffix) is as large as possible.
        
        The sign will be omitted unless the number is negative.
        
        Examples:
        
        - 1.5 will be serialized as "1500m" - 1.5Gi will be serialized as "1536Mi"
        
        Note that the quantity will NEVER be internally represented by a floating
        point number. That is the whole point of this exercise.
        
        Non-canonical values will still parse as long as they are well formed, but
        will be re-emitted in their canonical form. (So always use canonical form,
        or don't diff.)
        
        This format is intended to make it difficult to use these numbers without
        writing some sort of special handling code in the hopes that that will cause
        implementors to also use a fixed point implementation.

19. (不明原因被判斷不過)

20. (不明原因被判斷不過)