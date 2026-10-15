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


possiveis perguntas do exame
EXAM TASK Apply the label disktype=ssd to the worker node named kind
-worker, and schedule a pod named ssd-pod to the kind-worker node by
using a node selector.


node worker-1 tem 500MB de memoria e .5 de cpu disponivel. Schedule um pod que permita essa limitacao de memoria e cpu e coloque no no.