What are the valid way of creating a service in Kubernetes.
1. k expose deployment my-deployment --port 80 --target-port 80 --name my-service  (yes)
2. k create service my-test-service --tcp 80:80 (no)
3. k run my-pod --image nginx --expose --port 80 (yes)
4. k create service clusterip my-test-service --tcp 80:80