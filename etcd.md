Supplementary Content from Secrety:
(Udemy-Section3: Encryptng Secret Data at Rest)

See more in: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/


etcdctl
apt-get install etcd-client

etcdctl 

k get pods n kube-system
(etcd-controplan is in the list)


Copy command from the document:


ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/secret1 | hexdump -C


ps -aux| grep kube-api

(you can see the kube-api process is running)

Then follow the steps from the document


ls /etc/kubernetes/manifests/
etcd.yaml kube-apiserver.yaml kube-controller-manager.yaml kube-scheduler.yaml

cat /etc/kubernetes/manifests/kube-apiserver.yaml

(We'll see all the options provided but don't see the encryption option)


--encryption-provider-config
https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/


Example:
---
#
# CAUTION: this is an example configuration.
#          Do not use this for your own cluster!
#
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
      - configmaps
      - pandas.awesome.bears.example # a custom resource API
    providers:
      # This configuration does not provide data confidentiality. The first
      # configured provider is specifying the "identity" mechanism, which
      # stores resources as plain text.
      #
      - identity: {} # plain text, in other words NO encryption
      - aesgcm:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
            - name: key2
              secret: dGhpcyBpcyBwYXNzd29yZA==
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
            - name: key2
              secret: dGhpcyBpcyBwYXNzd29yZA==
      - secretbox:
          keys:
            - name: key1
              secret: YWJjZGVmZ2hpamtsbW5vcHFyc3R1dnd4eXoxMjM0NTY=
  - resources:
      - events
    providers:
      - identity: {} # do not encrypt Events even though *.* is specified below
  - resources:
      - '*.apps' # wildcard match requires Kubernetes 1.27 or later
    providers:
      - aescbc:
          keys:
          - name: key2
            secret: c2VjcmV0IGlzIHNlY3VyZSwgb3IgaXMgaXQ/Cg==
  - resources:
      - '*.*' # wildcard match requires Kubernetes 1.27 or later
    providers:
      - aescbc:
          keys:
          - name: key3
            secret: c2VjcmV0IGlzIHNlY3VyZSwgSSB0aGluaw==

There're 3 ways to encyrpt: aesgcm, aescbc & secretbox

These order matters becuase when the encyrption happens, it first use the 
top one to do encryption


According to the document, Encrypt your data:

    head -c 32 /dev/urandom | base64
    (It generates something like:  y0xT.....)

Then copy:
---
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
      - configmaps
      - pandas.awesome.bears.example
    providers:
      - aescbc:
          keys:
            - name: key1
              # See the following text for more details about the secret value
              secret: ~~<BASE 64 ENCODED SECRET>~~  **PASTE YOUR y0xT..... code hear**
      - identity: {} # this fallback allows reading unencrypted secrets;
                     # for example, during initial migration


into enc.yaml

vim enc.yaml

vi /etc/kubernetes/mainifests/kube-apiserver.yaml

Add (from the doc) 
        
        --encryption-provider-config=/etc/kubernetes/enc/enc.yaml

into enc.yaml


After doing above, we need to create a Secret Object by using the command:

    k create secret generic -h
    k create secret generic my-secret-2 --from-literal=key2=topsecret
    (secret/my-secret-2 created)

    k get secret
    (you can see: my-secret & my-secret-2)


Then rerun:
ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/~~secret1~~ CHANGE TO: my-secret-2 | hexdump -C

Ensre all Secrets are Encrypted:
(From the document)

    k get secret --all-namespace -o json| k replace -f -
