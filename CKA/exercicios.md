## Exercícios práticos

### **Capítulo 3**

#### RBAC

3.1. Create a new role named sa-creator that will allow creating Service Accounts.

3.2. Create a role binding that is associated with the previous sa-creator role named sa-creator-binding that will bind to the user Sandra.


**Resolução:**

```sh
kubectl create role sa-creator --verb=create,get,patch,delete,list --resource serviceaccount
```

```sh
kubectl create rolebinding sa-creator-binding --role=sa-creator --user=sandra
```

**Validação:**

```sh
kubectl auth can-i create serviceaccount --as sandra
```

#### Users and kubeconfig


3.3. Create a new user named Sandra, first creating the private key and then the certificate signing request. Then use the CSR resource in Kubernetes to generate the client certificate.

3.4. Add that new user Sandra to your local kubeconfig using the kubectl config command.

**Resolução:**


```sh
openssl genrsa -out sandra.key 3072

openssl req -new -key sandra.key -out sandra.csr -subj "/CN=sandra"

cat sandra.csr | base64 | tr -d "\n"

cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: sandra
spec:
  # This is an encoded CSR. Change this to the base64-encoded contents of myuser.csr
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJRFZqQ0NBYjRDQVFBd0VURVBNQTBHQTFVRUF3d0djMkZ1WkhKaE1JSUJvakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVk4QU1JSUJpZ0tDQVlFQXpyKy8reThaVkkybG13WVNCZktlY2tlUFZKUVh2L1NFUExiOFNvR2tLYXM1CmNBa245MGJlZnNHa2JDK3Jxa3I1MGh6TytvOVdrNmZ2dHRmaFMxR3Z2TUFDSUp3bmpmRG1BSXZwclRGN0JiQjYKd0R4UjdvazNPcjBjNXQ4L1kyNFlVVGt2TzFnOHFNandEU1ZqZW1lMGZyTm9pNTJTKzdsZ015ZmxVRTRueU1kMQo2cDhjVmtSaGtXNTBHMkFERjV6ZEcwcE5OajIwT1NydmhJUlNYSzcvUlpwUUdoNjFsTmhJS1F6U2NJbDhaaEdyCko5ZXdXaWFORkUwYTJmeWpyTzVvb0VlbEtFOHNDWUpLdkdGS0hEMWJ2NzlkWlFVSU1qTlE0RFl1Uk52M3ZPSDUKWFdPOUkwQWRnL20yb1J3T2VHQnVmL3RQcWlwYmtrMFE4cmVaUzhOQmZHbWtWQmtBOXE1QXVPK3JESDEyWXI1egpTdG5kbk8xY05hUHZ6L2NnSzJDMGd4Wm5ReWJVeUFjMytmOVQ0UnpENCtyUGRiSzhaOEpjTjhyR2ZlZWg4Y3djClJxVmdzRkRybEZkcHFhaSs0ajVSUzVHaEJYNTEwT3EzT2tITXczOUJsVnhHUDZMRUFHc3RYQTQ5TXRlN2RIcTUKb2hVbUE4Z0NjVis3YUFXRXh2MExBZ01CQUFHZ0FEQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FZRUFsV25BR0NESQpWOVpQZ29XVzlvdVpBM0lZR0FGVUtDNnRKZExVb3k1R2dQMHpuTXVqS2M1UUkzaFRzemhLVjNRT2IwVGVVZEFSCk1wYmR1M2FSUDU1VnFXY05mRVdIcUVwRzlyczFMaWhIcEJrNjRURlgxU0duNE9HcGRZT1lRRmJKeUNRUWZDTnAKdE9oUWlXU2tUVkJtVEwwUmMzRkZ2V2gxWVZEVkhMUis2TENkNi9KczV3eHJBSjRXZ1BKTkpDT0tiaEc0aUxPVwpsb0RrcGNITDVuQTVLRjZzZVlFbEJZbXd6elUvQUVqdHNmRUIxUDV0NllSK1gxbUtuMWNsWkdmY25FNktRYTFNCmUzZk1tZnpDWXNlc0JDdE5VOUw4R0RqWEd2L3hiV3ZleG4yNCtwQ0x1VFBzZkJEb2JDN0FudDdvU1lOSVlEc2gKVTdqQ2l5NW81aENwUEdiSkJwcVYyRDJ2T2RYWlppU2xsSGVGL1pnTWc5YzY2aHI0aEZVMUhTdzdmRHNTeEY5bwpPV21EcWxZZCtlanFxVmxTSGltczZoSGFKMXFKTm1mS0N6T0pFSEZ4M25TQ1JnU2J4RTB6Q2dqWHhFcVI4cGMvCmRWdnBGbStrVE5QSVNtc2V5dUI5emJjd0QrcmZMWU53S2xLWXVjQnhyN2NBUTdSU0F5dXVzUEZ4Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
EOF

kubectl get csr

kubectl certificate approve sandra

kubectl get csr sandra -o jsonpath='{.status.certificate}'| base64 -d > sandra.crt

kubectl config set-credentials sandra --client-key=sandra.key --client-certificate=sandra.crt --embed-certs=true

kubectl config get-clusters

kubectl config set-context sandra --cluster=cka --user=sandra
```

**Validação:**

```sh
kubectl --context sandra auth whoami
```

[Material de apoio](https://kubernetes.io/docs/tasks/tls/certificate-issue-client-csr/)

#### Service Accounts

3.5. Create a new Service Account named secure-sa, and create a pod that uses the
secure-sa Service Account. Make sure the token is not exposed to the pod.

3.6. Create a new cluster role named acme-corp-role that will allow the create action on Deployments, ReplicaSets, and DaemonSets. Bind that cluster role to the Service Account secure-sa, and make sure the Service Account can only create the assigned resources within the default namespace and nowhere else. Use auth can-i to verify that the secure-sa Service Account cannot create Deployments in the kube-system namespace. Create an arbitrary file with the command and output.

**Resolução:**


```sh
kubectl create clusterrole acme-corp-role --verb=create --resource=deployments,replicasets,daemonsets

kubectl create rolebinding acme-corp-rolebinding-sa   --clusterrole=acme-corp-role   --serviceaccount=default:secure-sa   --namespace=default

```

**Validação:**

```sh
kubectl auth can-i create deployments --namespace=default --as system:serviceaccount:default:secure-sa
> yes

kubectl auth can-i create deployments --namespace=kube-system --as system:serviceaccount:default:secure-sa
> no
```

### **Capítulo 4**

#### Node labels and affinity

4.1. Apply the label disktype=ssd to a node. Create a pod named fast using the nginx image, and make sure that it selects a node based on the label disktype=ssd.

4.2. Edit the fast pod using kubectl edit po fast, and change the node selector to disktype=slow. Notice that the pod cannot be changed and the YAML was saved to a temporary location. Take the YAML in /tmp/, and apply it by force to delete and recreate the pod using a single imperative command.

4.3. Create a new pod named ssd-pod using the nginx image, and use node affinity to select nodes based on a weight of 1 to nodes that have a label disktype=ssd. If the selection criteria doesn’t match, alternatively it can choose nodes that have a label kubernetes.io/os=linux


**Resolução:**


```sh
kubectl label node cka-worker disktype=ssd
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: fast
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - image: nginx
    name: fast
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```sh
kubectl replace --force -f /tmp/kubectl-edit-1631014790.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: ssd-pod
  name: ssd-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
  containers:
  - image: nginx
    name: ssd-pod
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

**Validação:**

```sh
kubectl get pods -owide
```


#### Resource requests and ConfigMaps

4.4. Create a pod named pod-limited with the image httpd, and set the resource requests to 1 for CPU and 100Mi for memory.

4.5. Create a Deployment named overloaded running three replicas of pods. Inside the pod is an initContainer with the native sidecar method (restartPolicy:Always) and a main container. For the sidecar, use the busybox image, and keep it running by issuing the command
/bin/bash sleep 1d. Set the CPU requests to 250 millicores and memory requests to 600 mebibytes. For the main container, use the nginx image. Set the CPU requests to 250 millicores and memory requests to 600 mebibytes. See if the pods within the Deployment get to a running state. If not, adjust the requests to get all three replicas up and running.

**Resolução:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: overloaded
  name: overloaded
spec:
  replicas: 3
  selector:
    matchLabels:
      app: overloaded
  strategy: {}
  template:
    metadata:
      labels:
        app: overloaded
    spec:
      initContainers:
      - name: busybox
        image: busybox:1.28
        restartPolicy: Always
        command: ['sh','-c', 'sleep 1d']
        resources:
          requests:
            memory: "600Mi"
            cpu: "250m"
      containers:
      - image: nginx
        name: nginx
        resources:
          requests:
            memory: "600Mi"
            cpu: "250m"
status: {}
```

4.6. Create a ConfigMap named ui-data with the following key and value pairs. Apply a ConfigMap to a pod named frontend with the image busybox:1.28, and pass it to the pod via the following environment variables: color.good=purple color.bad=yellow allow.textmode=true how.nice.to.look=fairlyNice

**Resolução:**
```sh
k create cm ui-data --from-literal=color.good=purple --from-literal=color.bad=yellow --from-literal=allow.textmode=true --from-literal=how.nice.to.look=fairlyNice
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod
  name: pod
spec:
  containers:
  - command:
    - sh
    - -c
    - env; sleep 8200
    image: busybox:1.28
    name: pod
    resources: {}
    envFrom:
    - configMapRef:
        name: ui-data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### **Capítulo 5**

#### Deployments and rollout

5.1. Using kubectl, create a Deployment named apache using the image httpd:latest with one replica. After the Deployment is running, scale the replicas up to 5.

5.2. Update the image for the Deployment apache from httpd:latest to httpd:2.4.63. Do not create a new YAML file or edit the existing resource (only use kubectl).

**Resolução:**


```sh
kubectl set image deploy/apache (nome deploy) http=httpd:2.4.63 (nome container=nova versao)
```


5.3. Using kubectl, view the events of the ReplicaSet that was created as a result of the image change from the previous exercise.

5.4. Using kubectl, roll back to the previous version of the Deployment named apache.

**Resolução:**


```sh
kubectl rollout undo deployment/apache
```

5.5. For the existing Deployment named apache, change the rollout strategy for a Deployment to Recreate.

**Resolução:**


Editar o deployment em strategy

```yaml
  strategy:
    type: Recreate
```


5.6. From a three-node cluster, cordon one of the worker nodes. Schedule a pod without a NodeSelector. Uncordon the worker node and edit the pod, applying a new node name to the YAML (set it to the node that was just uncordoned).After replacing the YAML, see if the pod is scheduled to the recently uncordoned node.

5.7. Start a basic nginx Deployment, and remove the taint from the control plane node so that pods don’t need a toleration to be scheduled to it. Add a node selector to the pod spec within the Deployment, and see if the pod is now running on the control plane node.

**Resolução:**


Remover taint:
```sh
kubectl taint cka-control-plane node-role.kubernetes.io/control-plane:NoSchedule-
```

Editar deploy incluindo nodeSelector

```yaml
nodeSelector:
  kubernetes.io/hostname: cka-control-plane
```


#### Services and Ingress

5.8. Create a Deployment named hello using the image nginxdemos/hello:plaintext with the kubectl command line. Expose the Deployment to create a ClusterIP Service named hello-svc that can communicate over port 80 using the kubectl command line. Use the correct kubectl command to verify that it’s a ClusterIP Service with the correct port exposed.

5.9. Change the hello-svc Service created in the previous exercise to a NodePort Service, where the NodePort should be 30000. Be sure to edit the Service in place, without creating a new YAML or issuing a new imperative command. Communicate with the pods within the hello Deployment via the NodePort Service using curl.

5.10. Install an ingress controller in the cluster using the command kubectl apply -f https://raw.githubusercontent.com/chadmcrowell/acing-the-cka-exam/main/ch_06/nginx-ingress-controller.yaml. Change the hello-svc Service back to a ClusterIP Service, and create an ingress resource that will route to the hello-svc Service when a client requests hello.com.

5.11. Create a new kind cluster without a CNI. Install the bridge CNI, followed by the Calico CNI. After installing the CNI, verify that the CoreDNS pods are up and running and the nodes are in a ready state.

### **Capítulo 6**

#### Network Policies


6.1. Exec into a pod, and cat out the DNS resolver file to see the IP address of the DNS server that the pod uses to resolve domain names. Open the file that contains the configuration for the kubelet, and change the value for clusterDNS to 100.96.0.10. Save and quit the file.

6.2. Stop and reload the kubelet daemon. Verify that the Service is active and running.

6.3. Locate the kube-dns Service. Try to edit the Service in place by changing the value of both clusterIP and ClusterIPs to 100.96.0.10. When the values cannot be updated, force a replacement of the Service with the correct kubectl command-line argument.

6.4. Edit the ConfigMap that contains the kubelet configuration. Change the IP address value that is set for clusterDNS to 100.96.0.10. Make sure to edit the resource without writing a new YAML file.

6.5. Scale the CoreDNS Deployment to three replicas. Verify that the pods have been created as a part of that Deployment.

6.6. Test access from a pod to a Service by first creating a Deployment with the apache image, followed by exposing that Deployment. Create a pod from the netshoot image, and verify that you can reach the Service that you just created.

6.7. Using the netshoot pod created in the previous exercise, locate the Service in the default namespace by its DNS name. Use as few letters as possible for DNS search functionality.

6.8. Create a Deployment named hello using the image nginxdemos/hello:plaintext with the kubectl command line. Expose the Deployment to create a ClusterIP Service named hello-svc that can communicate over port 80 using the kubectl command line. Use the correct kubectl command to verify that it’s a ClusterIP Service with the correct port exposed.

6.9. Change the hello-svc Service created in the previous exercise to a NodePort Service, where the NodePort should be 30000. Be sure to edit the Service in place, without creating a new YAML or issuing a new imperative command. Communicate with the pods within the hello Deployment via the NodePort Service using curl.

6.10. Install an ingress controller in the cluster using the command kubectl apply -f https://raw.githubusercontent.com/chadmcrowell/acing-the-cka-exam/main/ch_06/nginx-ingress-controller.yaml. Change the hello-svc Service back to a ClusterIP Service, and create an ingress resource that will route to the hello-svc Service when a client requests hello.com.

6.11. Create a new kind cluster without a CNI. Install the bridge CNI, followed by the
Calico CNI. After installing the CNI, verify that the CoreDNS pods are up and running and the nodes are in a ready state.

6.12. Set up a kind cluster with Calico CNI (as explained in appendix C). Create two namespaces: one named backend and another called frontend. Create a NetworkPolicy object in each namespace with a policy that denies all ingress traffic by default to all pods but allows all egress traffic. In the backend namespace, create a Deployment using the nginx image. Expose the Deployment to create a ClusterIP Service on port 80. In the frontend namespace, create a pod with the image nicolaka/netshoot. Exec into the pod, and use wget to connect to the nginx Service on the backend namespace. Notice what you see. After this attempt, create a second NetworkPolicy object in the backend namespace that allows ingress traffic only from the frontend namespace over port 80. Try again. Notice what happened.


#### Gateway API


6.13. Create a kind cluster with an ingress controller and Gateway API CRDs added, as instructed in appendix B. Once the cluster is up and running, create a new namespace named app-space. In the app-space namespace, create a pod named simple-web that uses the nginx image. Expose the pod, creating a Service named simple-web on port 80. Create an ingress resource in the app-space namespace named simple-web-ingress that maps / to the simple-web Service on port 80. Use host web.example.com. Set up a Gateway resource named web-gateway that listens on port 80 (HTTP). Create an HTTPRoute resource named simple-web-route that matches the host web.example.com. Forward the traffic to the simple-web Service. Modify the HTTPRoute so that only traffic to the /app path is allowed. Compare the ingress resource and the HTTPRoute, and notice the differences.


### **Capítulo 7**

#### PV, PVC

7.1. Create a PV named volstore308 that reserves 22 MB of storage and does not specify a storage class. The volume will use the storage from the host at /mnt/data.

7.2. Create a PVC named pv-claim-vol that will claim the previously created PV
with the correct access mode.

7.3. Create a pod named pod-access with the image centos:7 that will use the PVC from the previous step and mount the volume inside the container at /tmp/persistence. Issue a command to keep the container alive. You can use sleep 3600.

7.4. Create a storage class named node-local that uses the provisioner kubernetes.io/no-provisioner. The volume binding mode should be WaitForFirstConsumer.

7.5. Create a PVC named claim-sc that will claim 39 MB of volume from the previously created class. The access mode should be ReadWriteOnce.

7.6. Create a pod named pod-sc with the image nginx that will use the PVC from the previous step, and mount the volume inside the container at /usr/nginx/www/html.

7.7. Create a pod named shared-pod-data with two containers. The first container will be named busybox and will use the image busybox:1.28 with a command sleep 3600. The second container will be named httpd and will use the image
httpd:alpine3.17. Both containers should access the same volume that is shared from local storage on the container itself.
Container1 will mount the volume to /html/, and Container2 will mount the volume to /var/www/html/. Start up the pod, and ensure everything is mounted and shared correctly.


## Possíveis perguntas do exame

**EXAM TASK:** Apply the label disktype=ssd to the worker node named kind
-worker, and schedule a pod named ssd-pod to the kind-worker node by
using a node selector.


O node worker-1 tem 500 MB de memória e 0.5 de CPU disponível. Schedule um pod que respeite essa limitação de memória e CPU e agende-o no nó.

### Operators

#### Practice exercises

8. Install operator https://cert-manager.io/, a CNCF project used to generate
a TLS certificate for Kubernetes workloads. Navigate the documentation, and
check out different ways to install this operator. Install it on the namespace
my-cert-manager using the Helm method. Please note that installing CRDs
is part of the process as long as you pass the Helm argument --set
crds.enabled=true. After installation, check and list all CRDs installed that
belong to the cert-manager API group. Check whether the operator is running.
9. Install another operator—this time the popular Argo CD, a CNCF project used
for GitOps. See details at https://argo-cd.readthedocs.io/. Install the operator
via the manual way, following the steps at https://mng.bz/8XQ5. Choose the
installation option of Non High Availability, with the option of only requiring
namespace level privileges. In this mode, you will need to install the CRDs sep-
arately, which are at https://mng.bz/EwKX. The documentation provides the
installation command line using kubectl apply -k (the parameter -k uses the
Kustomize option). Once the operator and CRDs are installed, check the CRDs
of Argo CD, and see whether the operator is running.

### DNS and kubelet configuration

#### Practice exercises

1. Exec into a pod, and cat out the DNS resolver file to see the IP address of the
DNS server that the pod uses to resolve domain names.
Open the file that contains the configuration for the kubelet, and change the
value for clusterDNS to 100.96.0.10. Save and quit the file.
2. Stop and reload the kubelet daemon. Verify that the Service is active and
running.
3. Locate the kube-dns Service. Try to edit the Service in place by changing the
value of both clusterIP and ClusterIPs to 100.96.0.10. When the values can-
not be updated, force a replacement of the Service with the correct kubectl
command-line argument.
4. Edit the ConfigMap that contains the kubelet configuration. Change the IP
address value that is set for clusterDNS to 100.96.0.10. Make sure to edit the
resource without writing a new YAML file.
5. Scale the CoreDNS Deployment to three replicas. Verify that the pods have
been created as a part of that Deployment.
6. Test access from a pod to a Service by first creating a Deployment with the
apache image, followed by exposing that Deployment. Create a pod from the
netshoot image, and verify that you can reach the Service that you just created.
7. Using the netshoot pod created in the previous exercise, locate the Service in
the default namespace by its DNS name. Use as few letters as possible for
DNS search functionality.




## Perguntas do exame

**EXAM TASK:** The node named kind-worker in the cluster is experiencing prob-
lems with leaking memory. You must take down the node for maintenance by
Licensed to Julia Zuin <zuinjulia@hotmail.com>
122 CHAPTER 5 Running applications in Kubernetes
first disabling the scheduling of new pods to the node kind-worker. Then,
evict all pods that are currently running on kind-worker. Finally, once you’ve
verified that there are no pods running on kind-worker, enable scheduling
once again.

**EXAM TASK:** There’s a third node in the cluster named ik8s, but it is not
appearing when you perform the kubectl get nodes command. The name of
the node is node02. Allow node02 to join the cluster by recreating the join
command, and ensure that the node is in a Ready state when you list all the
nodes in the cluster.

From a three-node cluster, cordon one of the worker nodes. Schedule a pod
without a NodeSelector. Uncordon the worker node and edit the pod, applying
a new node name to the YAML (set it to the node that was just uncordoned).
After replacing the YAML, see if the pod is scheduled to the recently uncor-
doned node.
7. Start a basic nginx Deployment, and remove the taint from the control plane
node so that pods don’t need a toleration to be scheduled to it. Add a node
selector to the pod spec within the Deployment, and see if the pod is now run-
ning on the control plane node


**EXAM TASK:** In your cluster, change the IP addresses given to new Services to
a CIDR range of 100.96.0.0/16. Change the IP address associated with the
cluster DNS Service to match this new Service range. Proceed to change the
kubelet configuration so that new Pods can receive the new DNS Service IP
address and so they can resolve domain names. Edit the kubelet ConfigMap
so that the kubelet is updated in place and immediately reflected. Upgrade
the node to receive the new kubelet configuration. Finally, test this by creat-
ing a new pod and verifying that the pod has the new IP address of the DNS
Service.

EXAM TASK In your cluster, configure a hostPath PV named vol02833 with a
size of 100 MB of local storage from the worker node host node01. The direc-
tory on the host should be /mnt/data. Create a PVC named claim-02833 that
will reserve 90 MB of storage from the volume vol02833. Mount it to a pod

within a Deployment named frontend0113; the mount path within the con-
tainer should be /usr/share/nginx/html. The name of the YAML file should
be deploy.yaml.