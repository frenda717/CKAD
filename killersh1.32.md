First Attempt Score: 33% Out of 112 %

全對: 01, 02, 07, 10
半對: 03, 04, 13
寫不完: 14~22

不熟: 建立temporary pod
k run <pod-name> --restart=Never --rm --image=<image-name> -i/-it -- <command>
    -- curl command:
    curl http://<service-name>.<namespace>:<port:3333> 

(有些題目要求進行驗證，但我驗證沒做卻被識別為全部答對, eg. 10)


03. (Job半對，label沒有設置正確) Team Neptune needs a Job template located at /opt/course/3/job.yaml. This Job should run image busybox:1.31.0 and execute sleep 2 && echo done. It should be in namespace neptune, run a total of 3 times and should execute 2 runs in parallel.

    Start the Job and check its history. Each pod created by the Job should have the label id: awesome-job. The job should be named neb-new-job and the container neb-new-job-container.

    Solve this question on instance: ssh ckad7326

    答題情況: 3 of 6
    O Job created
    X Job has succceeded three times
    X Job has parallelism of two
    O Job has single container
    X Container has correct name
    O Container has correct image

    我的配置:
    andidate@ckad7326:~$ k get job -n neptune
    NAME          STATUS     COMPLETIONS   DURATION   AGE
    neb-new-job   Complete   1/1           4s         25h

    candidate@ckad7326:~$ k get job -n neptune -o yaml > neb-new-job.yaml
    candidate@ckad7326:~$ cat neb-new-job.yaml
    apiVersion: v1
    items:
    - apiVersion: batch/v1
    kind: Job
    metadata:
        name: neb-new-job
        namespace: neptune
    spec:
        completions: 3
        parallelism: 2
        selector:
            matchLabels:
                batch.kubernetes.io/controller-uid: 342ebfc8-169c-44a2-aba8-4e05003f8323
        template:
            metadata:
                creationTimestamp: null
                **labels:**
                    # 沒有設置**id: awesome-job**
                    # batch.kubernetes.io/controller-uid: 342ebfc8-169c-44a2-aba8-4e05003f8323
                    # batch.kubernetes.io/job-name: neb-new-job
                job-name: neb-new-job
            spec:
                containers:
                - image: busybox:1.31.0
                # 沒有設置command : ["/bin/sh","-c","sleep 2 && echo done"] 
                imagePullPolicy: IfNotPresent
                name: neb-new-job

    ...
    candidate@ckad7326:~$


    Solution:
    /# /opt/course/3/job.yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
        creationTimestamp: null
        name: neb-new-job
        namespace: neptune      # add
    spec:
        completions: 3          # add
        parallelism: 2          # add
        template:
            metadata:
            creationTimestamp: null
            labels:             # add
                id: awesome-job   # add
            spec:
            containers:
            - command:
                - sh
                - -c
                - sleep 2 && echo done
                image: busybox:1.31.0
                name: neb-new-job-container # update
                resources: {}
            restartPolicy: Never
    status: {}


    Check Job and Pods, you should see two running parallel at most but three in total: (同時有兩個pod一起運行但是最多生成三個pod)

    ➜ k -n neptune get pod,job | grep neb-new-job
    pod/neb-new-job-jhq2g              0/1     ContainerCreating   0          4s
    pod/neb-new-job-vf6ts              0/1     ContainerCreating   0          4s

    job.batch/neb-new-job   0/3           4s         5s


    ➜ k -n neptune get pod,job | grep neb-new-job
    pod/neb-new-job-gm8sz              0/1     ContainerCreating   0          0s
    pod/neb-new-job-jhq2g              0/1     Completed           0          10s
    pod/neb-new-job-vf6ts              1/1     Running             0          10s

    job.batch/neb-new-job   1/3           10s        11s


    ➜ k -n neptune get pod,job | grep neb-new-job
    pod/neb-new-job-gm8sz              0/1     ContainerCreating   0          5s

    pod/neb-new-job-jhq2g              0/1     Completed           0          15s
    pod/neb-new-job-vf6ts              0/1     Completed           0          15s
    job.batch/neb-new-job   2/3           15s        16s


    ➜ k -n neptune get pod,job | grep neb-new-job
    pod/neb-new-job-gm8sz              0/1     Completed          0          12s
    pod/neb-new-job-jhq2g              0/1     Completed          0          22s
    pod/neb-new-job-vf6ts              0/1     Completed          0          22s

    job.batch/neb-new-job   3/3           21s        23s
    Check history:

    ➜ k -n neptune describe job neb-new-job
    ...
    Events:
    Type    Reason            Age    From            Message
    ----    ------            ----   ----            -------
    Normal  SuccessfulCreate  2m52s  job-controller  Created pod: neb-new-job-jhq2g
    Normal  SuccessfulCreate  2m52s  job-controller  Created pod: neb-new-job-vf6ts
    Normal  SuccessfulCreate  2m42s  job-controller  Created pod: neb-new-job-gm8sz
    At the age column we can see that two pods run parallel and the third one after that. Just as it was required in the task.


04. (Helm) Team Mercury asked you to perform some operations using Helm, all in Namespace mercury:

    a. Delete release internal-issue-report-apiv1
    b. Upgrade release internal-issue-report-apiv2 to any newer version of chart bitnami/nginx available
    c. Install a new release internal-issue-report-apache of chart bitnami/apache. The Deployment should have two replicas, set these via Helm-values during install
    d. There seems to be a **broken release, stuck in pending-install state. Find it and delete it**

    Solve this question on instance: ssh ckad7326

    答題情況: 2 of 5
    O Deleted Helm release internal-issue-report-apiv1
    O Upgraded Helm release internal-issue-report-apiv2
    X Installed Helm release internal-issue-report-apache
    X Helm release internal-issure-report-apache has two replicas
    X Deleted broken Helm release

    c. helm -n mercury install internal-issue-report-apache bitnami/apache --set replicaCount=2: **--set replicaCount**
    d. helm -n mercury **ls -a**: helm list -a: show broken release

    Solution:
    Helm Chart: Kubernetes YAML template-files combined into a single package, Values allow customisation
    Helm Release: Installed instance of a Chart
    Helm Values: Allow to customise the YAML template-files in a Chart when creating a Release

    

    Step 1
    First we should delete the required release:

    ➜ helm -n mercury ls
    NAME                            NAMESPACE    ...   STATUS      CHART           APP VERSION
    internal-issue-report-apiv1     mercury      ...   deployed    nginx-18.1.14   1.27.1     
    internal-issue-report-apiv2     mercury      ...   deployed    nginx-18.1.14   1.27.1     
    internal-issue-report-app       mercury      ...   deployed    nginx-18.1.14   1.27.1  

    ➜ helm -n mercury uninstall internal-issue-report-apiv1 # 這步我設置正確
    release "internal-issue-report-apiv1" uninstalled

    ➜ helm -n mercury ls
    NAME                            NAMESPACE    ...   STATUS      CHART           APP VERSION  
    internal-issue-report-apiv2     mercury      ...   deployed    nginx-18.1.14   1.27.1     
    internal-issue-report-app       mercury      ...   deployed    nginx-18.1.14   1.27.1  
    

    Step 2
    Next we need to upgrade a release, for this we could first list the charts of the repo:

    ➜ helm repo list
    NAME    URL                  
    bitnami http://localhost:6000

    ➜ helm repo update
    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "bitnami" chart repository
    Update Complete. ⎈Happy Helming!⎈

    ➜ **helm search repo nginx --versions**
    NAME            CHART VERSION   APP VERSION     DESCRIPTION                                       
    bitnami/nginx   18.2.0          1.27.1          NGINX Open Source is a web server that can be a...
    bitnami/nginx   18.1.15         1.27.1          NGINX Open Source is a web server that can be a...
    bitnami/nginx   18.1.14         1.27.1          NGINX Open Source is a web server that can be a...
    bitnami/nginx   13.0.0          1.23.0          NGINX Open Source is a web server that can be a...
    Here we see that two newer chart versions are available. But the question only requires us to upgrade to any newer chart version available, so we can simply run:

    ➜ helm -n mercury upgrade internal-issue-report-apiv2 bitnami/nginx # 這步我設置正確
    Release "internal-issue-report-apiv2" has been upgraded. Happy Helming!
    NAME: internal-issue-report-apiv2
    LAST DEPLOYED: Wed Oct  2 14:17:09 2024
    NAMESPACE: mercury
    STATUS: deployed
    REVISION: 2
    TEST SUITE: None
    NOTES:
    CHART NAME: nginx
    CHART VERSION: 18.2.0
    APP VERSION: 1.27.1
    ...

    ➜ helm -n mercury ls
    NAME                            NAMESPACE   ...   STATUS      CHART           APP VERSION
    internal-issue-report-apiv2     mercury     ...   deployed    nginx-18.2.0    1.27.1     
    internal-issue-report-app       mercury     ...   deployed    nginx-18.1.14   1.27.1  
    Looking good!

    INFO: Also check out helm rollback for undoing a helm rollout/upgrade

    

    Step 3
    Now we're asked to install a new release, with a customised values setting. For this we first list all possible value settings for the chart, we can do this via:

    **helm show values bitnami/apache** # will show a long list of all possible value-settings

    helm show values bitnami/apache | yq e # parse yaml and show with colors
    Huge list, if we search in it we should find the setting replicaCount: 1 on top level. This means we can run:

    ➜ **helm -n mercury install internal-issue-report-apache bitnami/apache --set replicaCount=2**
    NAME: internal-issue-report-apache
    LAST DEPLOYED: Wed Oct  2 14:18:32 2024
    NAMESPACE: mercury
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    CHART NAME: apache
    CHART VERSION: 11.2.20
    APP VERSION: 2.4.62
    ...
    If we would also need to set a value on a deeper level, for example image.debug, we could run:

    helm -n mercury install internal-issue-report-apache bitnami/apache \
    --set replicaCount=2 \
    --set image.debug=true
    Install done, let's verify what we did:

    ➜ helm -n mercury ls
    NAME                            NAMESPACE    ...   STATUS       CHART             APP VERSION
    internal-issue-report-apache    mercury      ...   deployed     apache-11.2.20    2.4.62     
    internal-issue-report-apiv2     mercury      ...   deployed     nginx-18.2.0      1.27.1     
    internal-issue-report-app       mercury      ...   deployed     nginx-18.1.14     1.27.1 

    ➜ k -n mercury get deploy internal-issue-report-apache
    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    internal-issue-report-apache   2/2     2            2           64s
    We see a healthy deployment with two replicas!

    

    Step 4
    By default releases in pending-upgrade state aren't listed, but we can show all to find and delete the broken release:

    ➜ **helm -n mercury ls -a**
    NAME                            NAMESPACE   ...    STATUS            CHART             APP VERSION
    internal-issue-report-apache    mercury     ...    deployed          apache-11.2.20    2.4.62     
    internal-issue-report-apiv2     mercury     ...    deployed          nginx-18.2.0      1.27.1     
    internal-issue-report-app       mercury     ...    deployed          nginx-18.1.14     1.27.1     
    internal-issue-report-daniel    mercury     ...    pending-install   nginx-18.1.14     1.27.1 

    ➜ helm -n mercury uninstall internal-issue-report-daniel
    release "internal-issue-report-daniel" uninstalled
    Thank you Helm for making our lives easier! (Till something breaks)

    

    


05. Team Neptune has its own ServiceAccount named neptune-sa-v2 in Namespace neptune. A coworker needs the token from the Secret that belongs to that ServiceAccount.   Write the base64 decoded token to file /opt/course/5/token on ckad7326.

    Solve this question on instance: ssh ckad7326

    Secrets won't be created automatically for *ServiceAccounts, but it's possible to create a Secret manually and attach it to a ServiceAccount by setting the correct annotation on the Secret. This was done for this task.

    Solution:
    k -n neptune get sa # get overview
    k -n neptune get secrets # shows all secrets of namespace
    k -n neptune get secrets -oyaml | grep annotations -A 1 # shows secrets with first annotation
    If a Secret belongs to a ServiceAccount, it'll have the annotation kubernetes.io/service-account.name. Here the Secret we're looking for is neptune-secret-1.

    ➜ k -n neptune get secret neptune-secret-1 -o yaml
    apiVersion: v1
    data:
    ...
    token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkltNWFaRmRxWkRKMmFHTnZRM0JxV0haT1IxZzFiM3BJY201SlowaEhOV3hUWmt3elFuRmFhVEZhZDJNaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUp1WlhCMGRXNWxJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbTVsY0hSMWJtVXRjMkV0ZGpJdGRHOXJaVzR0Wm5FNU1tb2lMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObGNuWnBZMlV0WVdOamIzVnVkQzV1WVcxbElqb2libVZ3ZEhWdVpTMXpZUzEyTWlJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZ5ZG1salpTMWhZMk52ZFc1MExuVnBaQ0k2SWpZMlltUmpOak0yTFRKbFl6TXROREpoWkMwNE9HRTFMV0ZoWXpGbFpqWmxPVFpsTlNJc0luTjFZaUk2SW5ONWMzUmxiVHB6WlhKMmFXTmxZV05qYjNWdWREcHVaWEIwZFc1bE9tNWxjSFIxYm1VdGMyRXRkaklpZlEuVllnYm9NNENUZDBwZENKNzh3alV3bXRhbGgtMnZzS2pBTnlQc2gtNmd1RXdPdFdFcTVGYnc1WkhQdHZBZHJMbFB6cE9IRWJBZTRlVU05NUJSR1diWUlkd2p1Tjk1SjBENFJORmtWVXQ0OHR3b2FrUlY3aC1hUHV3c1FYSGhaWnp5NHlpbUZIRzlVZm1zazVZcjRSVmNHNm4xMzd5LUZIMDhLOHpaaklQQXNLRHFOQlF0eGctbFp2d1ZNaTZ2aUlocnJ6QVFzME1CT1Y4Mk9KWUd5Mm8tV1FWYzBVVWFuQ2Y5NFkzZ1QwWVRpcVF2Y3pZTXM2bno5dXQtWGd3aXRyQlk2VGo5QmdQcHJBOWtfajVxRXhfTFVVWlVwUEFpRU43T3pka0pzSThjdHRoMTBseXBJMUFlRnI0M3Q2QUx5clFvQk0zOWFiRGZxM0Zrc1Itb2NfV013
    kind: Secret
    ...
    This shows the base64 encoded token. To get the decoded one we could pipe it manually through base64 -d or we simply do:

    ➜ k -n neptune describe secret neptune-secret-1
    ...
    Data
    ====
    token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Im5aZFdqZDJ2aGNvQ3BqWHZOR1g1b3pIcm5JZ0hHNWxTZkwzQnFaaTFad2MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJuZXB0dW5lIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im5lcHR1bmUtc2EtdjItdG9rZW4tZnE5MmoiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoibmVwdHVuZS1zYS12MiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjY2YmRjNjM2LTJlYzMtNDJhZC04OGE1LWFhYzFlZjZlOTZlNSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpuZXB0dW5lOm5lcHR1bmUtc2EtdjIifQ.VYgboM4CTd0pdCJ78wjUwmtalh-2vsKjANyPsh-6guEwOtWEq5Fbw5ZHPtvAdrLlPzpOHEbAe4eUM95BRGWbYIdwjuN95J0D4RNFkVUt48twoakRV7h-aPuwsQXHhZZzy4yimFHG9Ufmsk5Yr4RVcG6n137y-FH08K8zZjIPAsKDqNBQtxg-lZvwVMi6viIhrrzAQs0MBOV82OJYGy2o-WQVc0UUanCf94Y3gT0YTiqQvczYMs6nz9ut-XgwitrBY6Tj9BgPprA9k_j5qEx_LUUZUpPAiEN7OzdkJsI8ctth10lypI1AeFr43t6ALyrQoBM39abDfq3FksR-oc_WMw
    ca.crt:     1066 bytes
    namespace:  7 bytes
    Copy the token (part under token:) and paste it using vim.

    vim /opt/course/5/token
    File /opt/course/5/token should contain the token:

    # /opt/course/5/token
    eyJhbGciOiJSUzI1NiIsImtpZCI6Im5aZFdqZDJ2aGNvQ3BqWHZOR1g1b3pIcm5JZ0hHNWxTZkwzQnFaaTFad2MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJuZXB0dW5lIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im5lcHR1bmUtc2EtdjItdG9rZW4tZnE5MmoiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoibmVwdHVuZS1zYS12MiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjY2YmRjNjM2LTJlYzMtNDJhZC04OGE1LWFhYzFlZjZlOTZlNSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpuZXB0dW5lOm5lcHR1bmUtc2EtdjIifQ.VYgboM4CTd0pdCJ78wjUwmtalh-2vsKjANyPsh-6guEwOtWEq5Fbw5ZHPtvAdrLlPzpOHEbAe4eUM95BRGWbYIdwjuN95J0D4RNFkVUt48twoakRV7h-aPuwsQXHhZZzy4yimFHG9Ufmsk5Yr4RVcG6n137y-FH08K8zZjIPAsKDqNBQtxg-lZvwVMi6viIhrrzAQs0MBOV82OJYGy2o-WQVc0UUanCf94Y3gT0YTiqQvczYMs6nz9ut-XgwitrBY6Tj9BgPprA9k_j5qEx_LUUZUpPAiEN7OzdkJsI8ctth10lypI1AeFr43t6ALyrQoBM39abDfq3FksR-oc_WMw


    candidate@ckad7326:~$ k get secret -n neptune neptune-secret-1 -o yaml
    apiVersion: v1
    data:
    ca.crt: LS0tLS1CRUd....
    namespace: bmVwdHVuZQ==
    token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklsSlBjalJQUVhCaU1teHJiV2hmVldONWFtOUdWMGhWTWpWaVIxVlBla3hGVDBoRk9EY3lNVUpJZWtFaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSW...nazZpMkFtRXA0V1h0aEMzd1dkQzdyX0g4QU9fcmFFdnBORjFwNkJ6dml2U3dRTzFoNWk3eEM0R2hpcGVOUVBQQ2s3QVVYMlYwenZQSGdwdVBlSkpzejEwelRJY1Y3Tm5haHBRYW54ZmJEc01zMWxGLTdJSTg3dXV1eEhUY0lHUnhfNDhLSzhIRl9CVDcxWG5MRnE2VXZWTmJ1T2ktNGkxMHR1ZnVvZU1jNE00MThnNURocElqbFJpS0t2TVBPWkkzZXZzeUJIOU1mZjNKZnJkYTJrQmdkdkJ5QmhNUlhYV0pQMVpPbUdRQWhn
    kind: Secret
    metadata:
    annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"Secret","metadata":{"annotations":{"kubernetes.io/service-account.name":"neptune-sa-v2"},"name":"neptune-secret-1","namespace":"neptune"},"type":"kubernetes.io/service-account-token"}
        kubernetes.io/service-account.name: neptune-sa-v2
        kubernetes.io/service-account.uid: 2ff9a03f-a379-4a2b-add5-14cf62ee6a8d
    ...

    candidate@ckad7326:~$ k get secret -n neptune neptune-secret-1 -o yaml |grep token:
    token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklsSlBjalJQUVhCaU1teHJiV2hmVldONWFtOUdWMGhWTWpWaVIxVlBla3hGVDBoRk9EY3lNVUpJZWtFaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUp1WlhCMGRXNWxJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbTVsY0hSMWJtVXRjMlZqY21WMExURWlMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObGNuWnBZMlV0WVdOamIzVnVkQzV1WVcxbElqb2libVZ3ZEhWdVpTMXpZUzEyTWlJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZ5ZG1salpTMWhZMk52ZFc1MExuVnBaQ0k2SWpKbVpqbGhNRE5tTFdFek56a3ROR0V5WWkxaFpHUTFMVEUwWTJZMk1tVmxObUU0WkNJc0luTjFZaUk2SW5ONWMzUmxiVHB6WlhKMmFXTmxZV05qYjNWdWREcHVaWEIwZFc1bE9tNWxjSFIxYm1VdGMyRXRkaklpZlEucDRzb2lZeWZWR0hHeUV6Y2Z6RTdxRkdXSndwTThQeVR4aklRZ045MklqNXE0ZzBPUjZSUlZueFE1UTdQSWlDY21hcjZCLWVIR2hYZkhNNTNxWXlodlZVazhaa0FwRzFJWDl2d04taS1acGtnazZpMkFtRXA0V1h0aEMzd1dkQzdyX0g4QU9fcmFFdnBORjFwNkJ6dml2U3dRTzFoNWk3eEM0R2hpcGVOUVBQQ2s3QVVYMlYwenZQSGdwdVBlSkpzejEwelRJY1Y3Tm5haHBRYW54ZmJEc01zMWxGLTdJSTg3dXV1eEhUY0lHUnhfNDhLSzhIRl9CVDcxWG5MRnE2VXZWTmJ1T2ktNGkxMHR1ZnVvZU1jNE00MThnNURocElqbFJpS0t2TVBPWkkzZXZzeUJIOU1mZjNKZnJkYTJrQmdkdkJ5QmhNUlhYV0pQMVpPbUdRQWhn


06. **(readinessProbe command修正pod StAtuS為Running 但READY 不為1/1!**) Create a single Pod named pod6 in Namespace default of image busybox:1.31.0. The Pod should have a readiness-probe executing cat /tmp/ready. It should initially wait 5 and periodically wait 10 seconds. This will set the container ready only if the file /tmp/ready exists.

    The Pod should run the command touch /tmp/ready && sleep 1d, which will create the necessary file to be ready and then idles. Create the Pod and confirm it starts.

    Solve this question on instance: ssh ckad5601

    candidate@ckad5601:~$ kd run pod6 --image=busybox:1.31.0 -o yaml > pod6.yaml
    candidate@ckad5601:~$ cat pod6.yaml
    apiVersion: v1
    kind: Pod
    metadata:
        creationTimestamp: null
        labels:
            run: pod6
        name: pod6
    spec:
        containers:
        - image: busybox:1.31.0
            name: pod6
            resources: {}
            command: ["/bin/sh","-c","touch /tmp/ready && sleep 1d"]
            readinessProbe:
                exec:
                    command:
                    # **Probe 內的command最好也在sh shell 內運行!!**
                    # **寫成: /bin/sh -c**
                    - cat /tmp/ready
                initialDelaySeconds: 5
                periodSeconds: 10
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}

    candidate@ckad5601:~$ kc pod6.yaml
    pod/pod6 created
    
    candidate@ckad5601:~$ k get pod
    NAME   READY   STATUS              RESTARTS   AGE
    pod1   1/1     Running             0          27h
    pod6   0/1     ContainerCreating   0          3s

    candidate@ckad5601:~$ k get pod
    NAME   READY   STATUS    RESTARTS   AGE
    pod1   1/1     Running   0          27h
    pod6   0/1     Running   0          13s
  
    candidate@ckad5601:~$ k logs pod6
    
    candidate@ckad5601:~$ k describe pod6
    error: the server doesn't have a resource type "pod6"
    candidate@ckad5601:~$ k describe pod pod6
    Name:             pod6
    Namespace:        default
    Priority:         0
    Service Account:  default
    Node:             cluster1-controlplane1/192.168.100.11
    Start Time:       Fri, 28 Feb 2025 23:16:39 +0000
    Labels:           run=pod6
    Annotations:      <none>
    Status:           Running
    IP:               10.32.0.31
    IPs:
    IP:  10.32.0.31
    Containers:
    pod6:
        Container ID:  containerd://8351fca389af11e3e7f06b9f885e16129949ab57e0315b406725356ddabf712e
        Image:         busybox:1.31.0
        Image ID:      docker.io/library/busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e
        Port:          <none>
        Host Port:     <none>
        Command:
        /bin/sh
        -c
        touch /tmp/ready && sleep 1d
        State:          Running
        Started:      Fri, 28 Feb 2025 23:16:44 +0000
        Ready:          False
        Restart Count:  0
        Readiness:      exec [cat /tmp/ready] delay=5s timeout=1s period=10s #success=1 #failure=3
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vwnt5 (ro)
    Conditions:
    Type                        Status
    PodReadyToStartContainers   True
    Initialized                 True
    Ready                       False
    ContainersReady             False
    PodScheduled                True
    Volumes:
    kube-api-access-vwnt5:
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
    Type     Reason     Age   From               Message
    ----     ------     ----  ----               -------
    Normal   Scheduled  29s   default-scheduler  Successfully assigned default/pod6 to cluster1-controlplane1
    Normal   Pulling    26s   kubelet            Pulling image "busybox:1.31.0"
    Normal   Pulled     24s   kubelet            Successfully pulled image "busybox:1.31.0" in 1.419s (1.42s including waiting). Image size: 764658 bytes.
    Normal   Created    24s   kubelet            Created container: pod6
    Normal   Started    24s   kubelet            Started container pod6
    Warning  Unhealthy  11s   kubelet            Readiness probe errored: rpc error: code = Unknown desc = failed to exec in container: failed to start exec "2543ce36a631139c80a5b79b62c2317ecc16fad5f1ae903c71d4260ac10869b4": OCI runtime exec failed: exec failed: unable to start container process: exec: "cat /tmp/ready": stat cat /tmp/ready: no such file or directory: unknown
    Warning  Unhealthy  2s    kubelet            **Readiness probe errored: rpc error: code = Unknown desc = failed to exec in container: failed to start exec "e0cf2b96795cd4fc3419b744d92adcaed0733ddd063f69052883babdf14599cb": OCI runtime exec failed: exec failed: unable to start container process: exec: "cat /tmp/ready": stat cat /tmp/ready: no such file or directory: unknown**

    
    ~~待測試我的配置是否可以通過!! 是否可以忽略Events 內failed訊息，因為我的配置都已經遵照題目要求~~
    **我需要修正Events 內的錯誤訊息，因為雖然Pod 最終status 為Runninng, 但並非1/1 Ready!!!**

    candidate@ckad5601:~$ k get pod
    NAME   READY   STATUS    RESTARTS   AGE
    pod1   1/1     Running   0          27h
    pod6   0/1     Running   0          13s

    在 CKAD 考试 中，如果 Events 内有 failed 消息，你需要判断以下两种情况：

    ✅ 你可以忽略 failed 事件的情况
    Pod 最终进入 Running 状态，且 READY=1/1

    检查命令：
    kubectl get pod pod6
    结果类似：
    NAME   READY   STATUS    RESTARTS   AGE
    pod6   1/1     Running   0          15s
    READY=1/1 代表 Readiness Probe 已通过，即使 Events 里有 Unhealthy，它可能是探针在文件 /tmp/ready 还没创建之前执行，导致的短暂错误，可以忽略。
    错误只出现在 Events 中，但 describe pod 结果一切正常

    你可以用：
    kubectl describe pod pod6
    如果 Readiness 显示：
    Readiness:  exec [cat /tmp/ready] delay=5s timeout=1s period=10s #success=1 #failure=3
    State: Ready
    说明 Readiness Probe 最终通过，可以忽略之前的失败记录。
    
    ❌ 你必须修正 failed 事件的情况
    READY=0/1 并且 kubectl describe pod pod6 显示 Readiness Probe 持续失败

    检查命令：
    kubectl describe pod pod6
    如果 Readiness Probe 一直失败：
    Warning  Unhealthy  Xs  kubelet  Readiness probe failed: cat: can't open '/tmp/ready': No such file or directory
    需要修复，否则 Pod 不会进入 Ready 状态，可能影响后续任务（如 Service 负载均衡）。
    Pod 进入 CrashLoopBackOff 或 Error 状态

    检查命令：
    kubectl get pod pod6
    如果 STATUS=CrashLoopBackOff，说明你的 command 可能有问题，容器一直重启，需要修正。
    
    📌 总结
    状况	需要修正？	说明
    READY=1/1 且 STATUS=Running	❌ 不需要	Events 里有 failed 但最终通过，CKAD 不扣分
    READY=0/1 且 STATUS=Running	✅ 需要	Readiness Probe 一直失败，Pod 无法进入 Ready 状态
    STATUS=CrashLoopBackOff / Error	✅ 需要	代表 Pod 启动失败，可能是 command 或 image 有问题
    如果你的 Pod 最终进入 Ready (1/1 Running)，CKAD 考试中你 不需要修正 Events 内的失败记录，可以直接提交答案。🚀


    Solution:
    # 6.yaml
    apiVersion: v1
    kind: Pod
    metadata:
        creationTimestamp: null
        labels:
            run: pod6
        name: pod6
    spec:
        containers:
        - command:
            - sh
            - -c
            - touch /tmp/ready && sleep 1d
            image: busybox:1.31.0
            name: pod6
            resources: {}
            readinessProbe:                             # add
                exec:                                     # add
                    command:                                # add
                    - **sh**                                    # add
                    - **-c**                                    # add
                    - cat /tmp/ready                        # add
                initialDelaySeconds: 5                    # add
                periodSeconds: 10                         # add
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}
    Then:

    k -f 6.yaml create
    Running k get pod6 we should see the job being created and completed:

    ➜ k get pod pod6
    NAME   READY   STATUS              RESTARTS   AGE
    pod6   0/1     ContainerCreating   0          2s

    ➜ k get pod pod6
    NAME   READY   STATUS    RESTARTS   AGE
    pod6   0/1     Running   0          7s

    ➜ k get pod pod6
    NAME   READY   STATUS    RESTARTS   AGE
    pod6   1/1     Running   0          15s
    We see that the Pod is finally ready.


    我修正後的配置:
    candidate@ckad5601:~$ vim pod6.yaml
    candidate@ckad5601:~$ cat pod6.yaml
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: pod6
    name: pod6
    spec:
    containers:
    - image: busybox:1.31.0
        name: pod6
        resources: {}
        command: ["/bin/sh","-c","touch /tmp/ready && sleep 1d"]
        readinessProbe:
        exec:
            command: [**"/bin/sh","-c"**,"cat /tmp/ready"]
        initialDelaySeconds: 5
        periodSeconds: 10
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}


    candidate@ckad5601:~$ kr pod6.yaml --force
    pod "pod6" deleted
    pod/pod6 replaced
    candidate@ckad5601:~$ k get pod
    NAME   READY    STATUS    RESTARTS   AGE
    pod1   1/1      Running   0          27h
    pod6   **1/1**  Running   0          29s
    
    "/bin/sh", "-c", "cat /tmp/ready" 强制使用 sh 解析命令，避免 cat /tmp/ready 直接执行时报错。

    📌 总结
    配置	是否稳定	可能问题
    ["cat", "/tmp/ready"]	❌ 可能失败	cat 直接执行可能遇到 exec 问题或找不到文件
    ["/bin/sh", "-c", "cat /tmp/ready"]	✅ 更稳定	sh 解析命令，确保 cat 能正确运行



07. (概念很簡單，yaml找出關鍵詞) The board of Team Neptune decided to take over control of one e-commerce webserver from Team Saturn. The administrator who once setup this webserver is not part of the organisation any longer. All information you could get was that the e-commerce system is called **my-happy-shop**. # 題目給出了線索，也就是說配置文件內應該包含這個字串 
    
    Search for the correct Pod in Namespace saturn and move it to Namespace neptune. It doesn't matter if you shut it down and spin it up again, it probably hasn't any customers anyways.

    Solve this question on instance: ssh ckad7326

    我的配置:

    我把所有pod從saturn 移到neptune了!!!!!

    candidate@ckad7326:~$ k describe pod -n neptune webserver-sat-001 |grep my-happy-shop
    candidate@ckad7326:~$ k describe pod -n neptune webserver-sat-002 |grep my-happy-shop
    candidate@ckad7326:~$ k describe pod -n neptune webserver-sat-003 |grep my-happy-shop
    Annotations:      description: this is the server for the E-Commerce System my-happy-shop
    candidate@ckad7326:~$ k describe pod -n neptune webserver-sat-004 |grep my-happy-shop
    candidate@ckad7326:~$ k describe pod -n neptune webserver-sat-005 |grep my-happy-shop
    candidate@ckad7326:~$ k describe pod -n neptune webserver-sat-006 |grep my-happy-shop

    可知只有webserver-sat-003 需要移動到neptune, 其他不用動

08. There is an existing Deployment named api-new-c32 in Namespace neptune. A developer did make an update to the Deployment but the updated version never came online. Check the Deployment history and find a revision that works, then rollback to it. Could you tell Team Neptune what the error was so it doesn't happen again?
    Solve this question on instance: ssh ckad7326

    We see 5 revisions, let's check Pod and Deployment status:

    ➜ k -n neptune get deploy,pod | grep api-new-c32
    deployment.extensions/api-new-c32    3/3     1            3           141m

    pod/api-new-c32-65d998785d-jtmqq    1/1     Running            0          141m
    pod/api-new-c32-686d6f6b65-mj2fp    1/1     Running            0          141m
    pod/api-new-c32-6dd45bdb68-2p462    1/1     Running            0          141m
    pod/api-new-c32-7d64747c87-zh648    0/1     ImagePullBackOff   0          141m
    Let's check the pod for errors:

    ➜ k -n neptune describe pod api-new-c32-7d64747c87-zh648 | grep -i error
    ...  Error: ImagePullBackOff
    ➜ k -n neptune describe pod api-new-c32-7d64747c87-zh648 | grep -i image
        Image:          ngnix:1.16.3
        Image ID:
        Reason:       ImagePullBackOff
    Warning  Failed  4m28s (x616 over 144m)  kubelet, gke-s3ef67020-28c5-45f7--default-pool-248abd4f-s010  Error: ImagePullBackOff


    candidate@ckad7326:~$ k rollout history -n neptune deploy api-new-c32
    deployment.apps/api-new-c32
    REVISION  CHANGE-CAUSE
    1         <none>
    2         kubectl edit deployment api-new-c32 --namespace=neptune
    3         kubectl edit deployment api-new-c32 --namespace=neptune
    4         kubectl edit deployment api-new-c32 --namespace=neptune
    5         kubectl edit deployment api-new-c32 --namespace=neptune (當前第五版導致ImagePullBackOff)

    candidate@ckad7326:~$ k rollout undo -n neptune deploy api-new-c32 (取消rollout回到第四版看看Image有沒有問題)
    deployment.apps/api-new-c32 rolled back
    
    candidate@ckad7326:~$ k get deploy -n neptune
    NAME          READY   UP-TO-DATE   AVAILABLE   AGE
    api-new-c32   3/3     1            3           13d
    candidate@ckad7326:~$ k describe deploy -n neptune api-new-c32 |grep Image 
        Image:         nginx:1.17.3
    
    candidate@ckad7326:~$ k rollout history -n neptune deploy api-new-c32
    deployment.apps/api-new-c32
    REVISION  CHANGE-CAUSE
    1         <none>
    2         kubectl edit deployment api-new-c32 --namespace=neptune
    3         kubectl edit deployment api-new-c32 --namespace=neptune
    5         kubectl edit deployment api-new-c32 --namespace=neptune
    6         kubectl edit deployment api-new-c32 --namespace=neptune (已經rollout回第四版，也就是這裡的6)

    (倘若rollout回第四版還是有錯誤，則再rollout 到第3版, 第2版, 第一版逐一排查)
    candidate@ckad7326:~$ k rollout undo -n neptune deploy api-new-c32 --to-revision 4
    error: unable to find specified revision 4 in history
    candidate@ckad7326:~$ k rollout undo -n neptune deploy api-new-c32 --to-revision 3
    deployment.apps/api-new-c32 rolled back
    candidate@ckad7326:~$ k rollout history -n neptune deploy api-new-c32
    deployment.apps/api-new-c32
    REVISION  CHANGE-CAUSE
    1         <none>
    2         kubectl edit deployment api-new-c32 --namespace=neptune
    5         kubectl edit deployment api-new-c32 --namespace=neptune
    6         kubectl edit deployment api-new-c32 --namespace=neptune
    7         kubectl edit deployment api-new-c32 --namespace=neptune

    candidate@ckad7326:~$ k rollout undo -n neptune deploy api-new-c32 --to-revision 2
    deployment.apps/api-new-c32 rolled back
    candidate@ckad7326:~$ k rollout history -n neptune deploy api-new-c32
    deployment.apps/api-new-c32
    REVISION  CHANGE-CAUSE
    1         <none>
    5         kubectl edit deployment api-new-c32 --namespace=neptune
    6         kubectl edit deployment api-new-c32 --namespace=neptune
    7         kubectl edit deployment api-new-c32 --namespace=neptune
    8         kubectl edit deployment api-new-c32 --namespace=neptune

    candidate@ckad7326:~$ k rollout undo -n neptune deploy api-new-c32 --to-revision 1
    deployment.apps/api-new-c32 rolled back
    candidate@ckad7326:~$ k rollout history -n neptune deploy api-new-c32
    deployment.apps/api-new-c32
    REVISION  CHANGE-CAUSE
    5         kubectl edit deployment api-new-c32 --namespace=neptune
    6         kubectl edit deployment api-new-c32 --namespace=neptune
    7         kubectl edit deployment api-new-c32 --namespace=neptune
    8         kubectl edit deployment api-new-c32 --namespace=neptune
    9         <none>

    candidate@ckad7326:~$ k describe deploy -n neptune api-new-c32 |grep Image
    j    Image:         nginx:1.16.1


09. (Pod yaml貼到Deployment yaml 貼小心一點!!) In Namespace pluto there is single Pod named holy-api. It has been working okay for a while now but Team Pluto needs it to be more reliable.

    Convert the Pod into a Deployment named holy-api with 3 replicas and delete the single Pod once done. The raw Pod template file is available at /opt/course/9/holy-api-pod.yaml.

    In addition, the new Deployment should set allowPrivilegeEscalation: false and privileged: false for the security context on container level.

    Please create the Deployment and save its yaml under /opt/course/9/holy-api-deployment.yaml on ckad9043

    Solve this question on instance: ssh ckad9043
    
    這題省略。花在debug deployment file 太長時間。

    candidate@ckad9043:~$ cat holy-api-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    labels:
        id: holy-api
    name: holy-api
    namespace: pluto     (Deployment)
    ---------------------------- 
                         (將Pod 設置貼進來再改變縮排)

    spec:
        replicas: 3
        selector:
            matchLabels:
            id: holy-api
        template:
            metadata:
            labels:
                id: holy-api
            spec:
            containers:
            - env:
                - name: CACHE_KEY_1


10. (這題答對，但use temporary pod to curl 驗證沒做!)
    Team Pluto needs a new cluster internal Service. Create a ClusterIP Service named project-plt-6cc-svc in Namespace pluto. This Service should expose a single Pod named project-plt-6cc-api of image nginx:1.17.3-alpine, create that Pod as well. The Pod should be identified by label project: plt-6cc-api. The Service should use tcp port redirection of 3333:80. --> Pod, Service, Endpoint均已建立

    Finally use for example curl from a temporary nginx:alpine Pod to get the response from the Service. Write the response into /opt/course/10/service_test.html on ckad9043. Also check if the logs of Pod project-plt-6cc-api show the request and write those into /opt/course/10/service_test.log on ckad9043. --> 這步驟沒做!

    Solve this question on instance: ssh ckad9043


    candidate@ckad9043:~$ k get pod -n pluto
    NAME                              READY   STATUS    RESTARTS   AGE
    holy-api                          1/1     Running   0          13d
    holy-api-f769fd5c5-bmmxg          1/1     Running   0          6m42s
    holy-api-f769fd5c5-dtj4l          1/1     Running   0          6m42s
    holy-api-f769fd5c5-x9cbg          1/1     Running   0          6m42s
    project-23-api-6dd78df55d-fjh72   1/1     Running   0          13d
    project-23-api-6dd78df55d-hw87f   1/1     Running   0          13d
    project-23-api-6dd78df55d-p7dj6   1/1     Running   0          13d
    project-plt-6cc-api               1/1     Running   0          27h
    candidate@ckad9043:~$ ls
    earh-project-earchflower-pvc.yaml  holy-api-deployment.yaml  moon-retain.yaml
    earth-project-earchflower-pv.yaml  holy-api.yaml             project-earthflower.yaml
    earth-project-earthflower-pv.yaml  moon-pvc-126.yaml         web-moon.yaml
    candidate@ckad9043:~$ k get svc -n pluto
    NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    project-plt-6cc-svc   ClusterIP   10.104.83.95   <none>        3333/TCP   27h
    
    candidate@ckad9043:~$ k describe svc -n pluto project-plt-6cc-svc
    Name:                     project-plt-6cc-svc
    Namespace:                pluto
    Labels:                   project=plt-6cc-api
    Annotations:              <none>
    Selector:                 project=plt-6cc-api
    Type:                     ClusterIP
    IP Family Policy:         SingleStack
    IP Families:              IPv4
    IP:                       10.104.83.95
    IPs:                      10.104.83.95
    Port:                     <unset>  3333/TCP
    TargetPort:               80/TCP
    Endpoints:                10.32.0.26:80
    Session Affinity:         None
    Internal Traffic Policy:  Cluster
    Events:                   <none>
    candidate@ckad9043:~$ k describe pod -n pluto project-plt-6cc-api |grep IP
    IP:               10.32.0.26
    IPs:
    IP:  10.32.0.26
    
    k run <pod-name> --restart=Never --rm --image=<image-name> -i/-it -- <command>
    -- curl command:
    curl http://<service-name>.<namespace>:<port:3333> 

    candidate@ckad9043:~$ **k run tmp --restart=Never --rm --image=nginx:alpine -i -- curl http://project-plt-6cc-svc.pluto:3333**
    If you don't see a command prompt, try pressing enter.
    warning: couldn't attach to pod/tmp, falling back to streaming logs: error stream protocol error: unknown error
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100   612  100   612    0     0    190      0  0:00:03  0:00:03 --:--:--   190
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
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




11. There are files to build a container image located at /opt/course/11/image on ckad9043. The container will run a Golang application which outputs information to stdout. You're asked to perform the following tasks:

    ℹ️ Run all Docker and Podman commands as user root. Use sudo docker and sudo podman or become root with sudo -i

    Change the Dockerfile: set ENV variable SUN_CIPHER_ID to hardcoded value 5b9c1065-e39d-4a43-a04a-e59bcea3e03f
    Build the image using sudo docker, tag it registry.killer.sh:5000/sun-cipher:v1-docker and push it to the registry
    Build the image using sudo podman, tag it registry.killer.sh:5000/sun-cipher:v1-podman and push it to the registry
    Run a container using sudo podman, which keeps running detached in the background, named sun-cipher using image registry.killer.sh:5000/sun-cipher:v1-podman
    Write the logs your container sun-cipher produces into /opt/course/11/logs on ckad9043

    Solve this question on instance: ssh ckad9043


12. (Podman) Create a new PersistentVolume named earth-project-earthflower-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

    Next create a new PersistentVolumeClaim in Namespace earth named earth-project-earthflower-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

    Finally create a new Deployment project-earthflower in Namespace earth which mounts that volume at /tmp/project-data. The Pods of that Deployment should be of image httpd:2.4.41-alpine.

    Solve this question on instance: ssh ckad5601


13. (沒有建立StorageClass) Team Moonpie, which has the Namespace moon, needs more storage. Create a new PersistentVolumeClaim named moon-pvc-126 in that namespace. This claim should use a new StorageClass moon-retain with the provisioner set to moon-retainer and the reclaimPolicy set to Retain. The claim should request storage of 3Gi, an accessMode of ReadWriteOnce and should use the new StorageClass.

    The provisioner moon-retainer will be created by another team, so it's expected that the PVC will not boot yet. Confirm this by writing the event message from the PVC into file /opt/course/13/pvc-126-reason on ckad9043.

    Solve this question on instance: ssh ckad9043

    答題情況: 3 of 6
    X StorageClass exists
    X StorageClass correct defined
    O PersistentVolumeClaim exists
    O PersistentVolumeClaim correct defined
    O PersistentVolumeClaim status Pending
    X File /opt/course/13/pvc-126-reason valid


    Solution:
    vim 13_sc.yaml
    Head to https://kubernetes.io/docs, search for "storageclass" and alter the example code to this:

    # 13_sc.yaml # 我沒有先檢查StorageClass 是否存在，如果不存在就需要先定義!
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
        name: moon-retain
    provisioner: moon-retainer
    reclaimPolicy: Retain

    k create -f 13_sc.yaml
    Now the same for the PersistentVolumeClaim, head to the docs, copy an example and transform it into:

    vim 13_pvc.yaml
    # 13_pvc.yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
    name: moon-pvc-126            # name as requested
    namespace: moon               # important
    spec:
    accessModes:
        - ReadWriteOnce             # RWO
    resources:
        requests:
        storage: 3Gi              # size
    storageClassName: moon-retain # uses our new storage class

    k -f 13_pvc.yaml create
    Next we check the status of the PVC :

    ➜ k -n moon get pvc
    NAME           STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    moon-pvc-126   Pending                                      moon-retain    2m57s
    ➜ k -n moon describe pvc moon-pvc-126
    Name:          moon-pvc-126
    ...
    Status:        Pending
    ...
    Events:
    Type    Reason                Age                  From                         Message
    ----    ------                ----                 ----                         -------
    Normal  ExternalProvisioning  4s (x19 over 4m28s)  persistentvolume-controller    Waiting for a volume to be created either by the external provisioner 'moon-retainer' or manually by the system administrator. If volume creation is delayed, please verify that the provisioner is running and correctly registered.
    This confirms that the PVC waits for the provisioner moon-retainer to be created. Finally we copy or write the event message into the requested location:

    # /opt/course/13/pvc-126-reason
    Waiting for a volume to be created either by the external provisioner 'moon-retainer' or manually by the system administrator. If volume creation is delayed, please verify that the provisioner is running and correctly registered.
    
    (建立StorageClass之後，將pvc的Events寫到文件內)
    candidate@ckad9043:~$ k get storageclass -n moon
    NAME          PROVISIONER     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
    moon-retain   moon-retainer   Retain          Immediate           false                  6m49s


    candidate@ckad9043:~$ k describe pvc -n moon moon-pvc-126 |grep -A10 "Events"
    Events:
    Type     Reason                Age                    From                         Message
    ----     ------                ----                   ----                         -------
    Warning  ProvisioningFailed    8h (x261 over 9h)      persistentvolume-controller  storageclass.storage.k8s.io "moon-retain" not found
    Warning  ProvisioningFailed    7m25s (x102 over 32m)  persistentvolume-controller  storageclass.storage.k8s.io "moon-retain" not found
    Normal   ExternalProvisioning  10s (x19 over 4m40s)   persistentvolume-controller  Waiting for a volume to be created either by the external provisioner 'moon-retainer' or manually by the system administrator. If volume creation is delayed, please verify that the provisioner is running and correctly registered.
    candidate@ckad9043:~$ k describe pvc -n moon moon-pvc-126 |grep -A10 "Events" > /opt/course/13/pvc-126-reaso
    candidate@ckad9043:~$ cat /opt/course/13/pvc-126-reaso
    Events:
    Type     Reason                Age                    From                         Message
    ----     ------                ----                   ----                         -------
    Warning  ProvisioningFailed    8h (x261 over 9h)      persistentvolume-controller  storageclass.storage.k8s.io "moon-retain" not found
    Warning  ProvisioningFailed    7m35s (x102 over 32m)  persistentvolume-controller  storageclass.storage.k8s.io "moon-retain" not found
    Normal   ExternalProvisioning  5s (x20 over 4m50s)    persistentvolume-controller  Waiting for a volume to be created either by the external provisioner 'moon-retainer' or manually by the system administrator. If volume creation is delayed, please verify that the provisioner is running and correctly registered.


14. You need to make changes on an existing Pod in Namespace moon called secret-handler. Create a new Secret secret1 which contains user=test and pass=pwd. The Secret's content should be available in Pod secret-handler as environment variables SECRET1_USER and SECRET1_PASS. The yaml for Pod secret-handler is available at /opt/course/14/secret-handler.yaml.

    There is existing yaml for another Secret at /opt/course/14/secret2.yaml, create this Secret and mount it inside the same Pod at /tmp/secret2. Your changes should be saved under /opt/course/14/secret-handler-new.yaml on ckad9043. Both Secrets should only be available in Namespace moon.

    Solve this question on instance: ssh ckad9043

15. Team Moonpie has a nginx server Deployment called web-moon in Namespace moon. Someone started configuring it but it was never completed. To complete please create a ConfigMap called configmap-web-moon-html containing the content of file /opt/course/15/web-moon.html under the data key-name index.html.

    The Deployment web-moon is already configured to work with this ConfigMap and serve its content. Test the nginx configuration for example using curl from a temporary nginx:alpine Pod.

    Solve this question on instance: ssh ckad9043

16. The Tech Lead of Mercury2D decided it's time for more logging, to finally fight all these missing data incidents. There is an existing container named cleaner-con in Deployment cleaner in Namespace mercury. This container mounts a volume and writes logs into a file called cleaner.log.

    The yaml for the existing Deployment is available at /opt/course/16/cleaner.yaml. Persist your changes at /opt/course/16/cleaner-new.yaml on ckad7326 but also make sure the Deployment is running.

    Create a sidecar container named logger-con, image busybox:1.31.0 , which mounts the same volume and writes the content of cleaner.log to stdout, you can use the tail -f command for this. This way it can be picked up by kubectl logs.

    Check if the logs of the new container reveal something about the missing data incidents.

    Solve this question on instance: ssh ckad7326


17. Last lunch you told your coworker from department Mars Inc how amazing InitContainers are. Now he would like to see one in action. There is a Deployment yaml at /opt/course/17/test-init-container.yaml. This Deployment spins up a single Pod of image nginx:1.17.3-alpine and serves files from a mounted volume, which is empty right now.

    Create an InitContainer named init-con which also mounts that volume and creates a file index.html with content check this out! in the root of the mounted volume. For this test we ignore that it doesn't contain valid html.

    The InitContainer should be using image busybox:1.31.0. Test your implementation for example using curl from a temporary nginx:alpine Pod.

    Solve this question on instance: ssh ckad5601


18. There seems to be an issue in Namespace mars where the ClusterIP service manager-api-svc should make the Pods of Deployment manager-api-deployment available inside the cluster.

    You can test this with curl manager-api-svc.mars:4444 from a temporary nginx:alpine Pod. Check for the misconfiguration and apply a fix.

    Solve this question on instance: ssh ckad5601


19. In Namespace jupiter you'll find an apache Deployment (with one replica) named jupiter-crew-deploy and a ClusterIP Service called jupiter-crew-svc which exposes it. Change this service to a NodePort one to make it available on all nodes on port 30100.

    Test the NodePort Service using the internal IP of all available nodes and the port 30100 using curl, you can reach the internal node IPs directly from your main terminal. On which nodes is the Service reachable? On which node is the Pod running?

    Solve this question on instance: ssh ckad5601


20. In Namespace venus you'll find two Deployments named api and frontend. Both Deployments are exposed inside the cluster using Services. Create a NetworkPolicy named np1 which restricts outgoing tcp connections from Deployment frontend and only allows those going to Deployment api. Make sure the NetworkPolicy still allows outgoing traffic on UDP/TCP ports 53 for DNS resolution.

    Test using: wget www.google.com and wget api:2222 from a Pod of Deployment frontend.

    Solve this question on instance: ssh ckad7326

21. Team Neptune needs 3 Pods of image httpd:2.4-alpine, create a Deployment named neptune-10ab for this. The containers should be named neptune-pod-10ab. Each container should have a memory request of 20Mi and a memory limit of 50Mi.

    Team Neptune has it's own ServiceAccount neptune-sa-v2 under which the Pods should run. The Deployment should be in Namespace neptune

    Solve this question on instance: ssh ckad7326

22. Team Sunny needs to identify some of their Pods in namespace sun. They ask you to add a new label protected: true to all Pods with an existing label type: worker or type: runner. 
    Also add an annotation protected: do not delete this pod to all Pods having the new label protected: true.
    
    Solve this question on instance: ssh ckad9043