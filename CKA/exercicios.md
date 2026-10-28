exercicios praticos

Practice exercises
1 Create a new role named sa-creator that will allow creating Service Accounts.
2 Create a role binding that is associated with the previous sa-creator role named sa-creator-binding that will bind to the user Sandra.


rbac
Practice exercises
3 Create a new user named Sandra, first creating the private key and then the
certificate signing request. Then use the CSR resource in Kubernetes to gener-
ate the client certificate.
4 Add that new user Sandra to your local kubeconfig using the kubectl config
command.

serviceaccounts
5 Create a new Service Account named secure-sa, and create a pod that uses the
secure-sa Service Account. Make sure the token is not exposed to the pod.
6 Create a new cluster role named acme-corp-role that will allow the create
action on Deployments, ReplicaSets, and DaemonSets. Bind that cluster role
to the Service Account secure-sa, and make sure the Service Account can only
create the assigned resources within the default namespace and nowhere else.
Use auth can-i to verify that the secure-sa Service Account cannot create
Deployments in the kube-system namespace. Create an arbitrary file with the
command and output.


Apply the label disktype=ssd to a node. Create a pod named fast using
the nginx image, and make sure that it selects a node based on the label
disktype=ssd.
2 Edit the fast pod using kubectl edit po fast, and change the node selector
to disktype=slow. Notice that the pod cannot be changed and the YAML was
saved to a temporary location. Take the YAML in /tmp/, and apply it by force to
delete and recreate the pod using a single imperative command.
3 Create a new pod named ssd-pod using the nginx image, and use node affinity
to select nodes based on a weight of 1 to nodes that have a label disktype=ssd.
If the selection criteria doesn’t match, alternatively it can choose nodes that
have a label kubernetes.io/os=linux.a



 Create a pod named pod-limited with the image httpd, and set the resource
requests to 1 for CPU and 100Mi for memory.
5 Create a Deployment named overloaded running three replicas of pods. Inside
the pod is an initContainer with the native sidecar method (restartPolicy:
Always) and a main container. For the sidecar, use the busybox image, and keep
it running by issuing the command /bin/bash sleep 1d. Set the CPU requests
to 250 millicores and memory requests to 600 mebibytes. For the main con-
tainer, use the nginx image. Set the CPU requests to 250 millicores and memory
requests to 600 mebibytes. See if the pods within the Deployment get to a run-
ning state. If not, adjust the requests to get all three replicas up and running.
6 Create a ConfigMap named ui-data with the following key and value pairs.
Apply a ConfigMap to a pod named frontend with the image busybox:1.28, and
pass it to the pod via the following environment variables:
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice


 Using kubectl, create a Deployment named apache using the image httpd:latest
with one replica. After the Deployment is running, scale the replicas up to 5.
2 Update the image for the Deployment apache from httpd:latest to httpd:
2.4.63. Do not create a new YAML file or edit the existing resource (only use
kubectl).
3 Using kubectl, view the events of the ReplicaSet that was created as a result
of the image change from the previous exercise.
4 Using kubectl, roll back to the previous version of the Deployment named
apache.
5 For the existing Deployment named apache, change the rollout strategy for a
Deployment to Recreate.




possiveis perguntas do exame
EXAM TASK Apply the label disktype=ssd to the worker node named kind
-worker, and schedule a pod named ssd-pod to the kind-worker node by
using a node selector.


O node worker-1 tem 500MB de memória e 0.5 de CPU disponível. Schedule um pod que respeite essa limitação de memória e CPU e agende-o no nó.

Practice exercises
8 Install operator https://cert-manager.io/, a CNCF project used to generate
a TLS certificate for Kubernetes workloads. Navigate the documentation, and
check out different ways to install this operator. Install it on the namespace
my-cert-manager using the Helm method. Please note that installing CRDs
is part of the process as long as you pass the Helm argument --set
crds.enabled=true. After installation, check and list all CRDs installed that
belong to the cert-manager API group. Check whether the operator is running.
9 Install another operator—this time the popular Argo CD, a CNCF project used
for GitOps. See details at https://argo-cd.readthedocs.io/. Install the operator
via the manual way, following the steps at https://mng.bz/8XQ5. Choose the
installation option of Non High Availability, with the option of only requiring
namespace level privileges. In this mode, you will need to install the CRDs sep-
arately, which are at https://mng.bz/EwKX. The documentation provides the
installation command line using kubectl apply -k (the parameter -k uses the
Kustomize option). Once the operator and CRDs are installed, check the CRDs
of Argo CD, and see whether the operator is running.

Practice exercises
1 Exec into a pod, and cat out the DNS resolver file to see the IP address of the
DNS server that the pod uses to resolve domain names.
Open the file that contains the configuration for the kubelet, and change the
value for clusterDNS to 100.96.0.10. Save and quit the file.
2 Stop and reload the kubelet daemon. Verify that the Service is active and
running.
3 Locate the kube-dns Service. Try to edit the Service in place by changing the
value of both clusterIP and ClusterIPs to 100.96.0.10. When the values can-
not be updated, force a replacement of the Service with the correct kubectl
command-line argument.
4 Edit the ConfigMap that contains the kubelet configuration. Change the IP
address value that is set for clusterDNS to 100.96.0.10. Make sure to edit the
resource without writing a new YAML file.
5 Scale the CoreDNS Deployment to three replicas. Verify that the pods have
been created as a part of that Deployment.
6 Test access from a pod to a Service by first creating a Deployment with the
apache image, followed by exposing that Deployment. Create a pod from the
netshoot image, and verify that you can reach the Service that you just created.
7 Using the netshoot pod created in the previous exercise, locate the Service in
the default namespace by its DNS name. Use as few letters as possible for
DNS search functionality.




## exame
EXAM TASK The node named kind-worker in the cluster is experiencing prob-
lems with leaking memory. You must take down the node for maintenance by
Licensed to Julia Zuin <zuinjulia@hotmail.com>
122 CHAPTER 5 Running applications in Kubernetes
first disabling the scheduling of new pods to the node kind-worker. Then,
evict all pods that are currently running on kind-worker. Finally, once you’ve
verified that there are no pods running on kind-worker, enable scheduling
once again.

EXAM TASK There’s a third node in the cluster named ik8s, but it is not
appearing when you perform the kubectl get nodes command. The name of
the node is node02. Allow node02 to join the cluster by recreating the join
command, and ensure that the node is in a Ready state when you list all the
nodes in the cluster.

From a three-node cluster, cordon one of the worker nodes. Schedule a pod
without a NodeSelector. Uncordon the worker node and edit the pod, applying
a new node name to the YAML (set it to the node that was just uncordoned).
After replacing the YAML, see if the pod is scheduled to the recently uncor-
doned node.
7 Start a basic nginx Deployment, and remove the taint from the control plane
node so that pods don’t need a toleration to be scheduled to it. Add a node
selector to the pod spec within the Deployment, and see if the pod is now run-
ning on the control plane node


EXAM TASK In your cluster, change the IP addresses given to new Services to
a CIDR range of 100.96.0.0/16. Change the IP address associated with the
cluster DNS Service to match this new Service range. Proceed to change the
kubelet configuration so that new Pods can receive the new DNS Service IP
address and so they can resolve domain names. Edit the kubelet ConfigMap
so that the kubelet is updated in place and immediately reflected. Upgrade
the node to receive the new kubelet configuration. Finally, test this by creat-
ing a new pod and verifying that the pod has the new IP address of the DNS
Service.