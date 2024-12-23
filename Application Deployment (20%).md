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
    a. manualy change the YAML file and then type: 
            
            kubectl edit pod redis (Modify the generated pod!!)
            kubectl apply -f redis.yaml
            kubectl get pods

    """ ***Warning: Wrong Command***:
        controlplane ~ ➜  vim redis.yaml   (Thid is not directly modify the generated pod!!!!)
        controlplane ~ ✖ k apply -f redis.yaml
        Warning: resource pods/redis is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
        pod/redis configured (So it produce the above warning)

    You should use k edit pod (pod name)  ========> This can directly modify the generated pod
    controlplane ~ ➜  k edit pod redis
    pod/redis edited

    controlplane ~ ➜  k apply -f redis.yaml
    pod/redis configured

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

1. Scale the RepicaSet to 5 Pods:
    a. kubectl scale:
    controlplane ~ ➜  kubectl scale --replicas=5 
    error: You must provide one or more resources by argument or filename.
    Example resource specifications include:
    '-f rsrc.yaml'
    '--filename=rsrc.json'
    '<resource> <name>'
    '<resource>'

    controlplane ~ ➜  kubectl scale replicaset new-replica-set --replicas=3 replicaset.apps/new-replica-set scaled

    b. kubectl edit replicaset new-replica-set, vi replica column 


2. Now scale the ReplicaSet down to 2 PODs.


Use the kubectl scale command or edit the replicaset using kubectl edit replicaset.

controlplane ~ ➜  kubectl scale replicaset new-replica-set --replicas=2
replicaset.apps/new-replica-set scaled
