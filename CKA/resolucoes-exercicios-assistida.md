1. EXAM TASK In your cluster, change the IP addresses given to new Services to
a CIDR range of 100.96.0.0/16. Change the IP address associated with the
cluster DNS Service to match this new Service range. Proceed to change the
kubelet configuration so that new Pods can receive the new DNS Service IP
address and so they can resolve domain names. Edit the kubelet ConfigMap
so that the kubelet is updated in place and immediately reflected. Upgrade
the node to receive the new kubelet configuration. Finally, test this by creat-
ing a new pod and verifying that the pod has the new IP address of the DNS
Service.


Alguns conceitos:
- Nodes CIDR: tambem conhecidos como `internal-IP`, essa e a camada subjacente da cloud/vlan subnet que os workers e o controlplane se conectam com a infra. O Cloud provider gerencia esse range

- Pod CIDR: Tambem conhecido como `Cluster-CIDR`, durante a criacao os pods recebem um endereco de IP  e o range é configurado na instalacao do cluster kubernetes e gerenciada pelo container network interface (CNI, network plugin)

- Service CIDR: tambem conhecida como `Service-Cluster-IP-Range`, porque pods sao recriados a qualquer momento entao existe uma necessidade de abstrair o grupo de pods em um unico range de IP dentro do cluster. Esse range é gerenciado e configurado pelo API Server. ServiceCIDR é um tipo de recurso que nao fica em namesoace que define range de enderecos de virtualIPs que podem ser atribuidos a services tipo ClusterIP, sendo assim todo service desse tipo tem um IP virtual estavel dentro do range do CIDR especificado. 

Resolucao:

1. Atualizar o range do service-CIDR - que é controlado pelo API Server -> dentro do controlplane
```sh
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

alterando a linha - --service-cluster-ip-range=100.96.0.0/16

Aguardar o pod do APIServer restartar

2. 
```sh
kubectl get servicecidr
```

Precisamos deletar por esta com endereco de IP antigo mas antes deve-se deletar os svcs associados com ele

```sh
kubectl delete svc kube-dns -n kube-system
kubectl delete svc kubernetes
kubectl delete servicecidr kubernetes
```

3. Recriar o kubernetes service se nao for recriado automaticamente

```sh
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Service
metadata:
  name: kubernetes
  namespace: default
  labels:
    component: apiserver
    provider: kubernetes
spec:
  ports:
    - name: https
      port: 443
      protocol: TCP
      targetPort: 6443
  type: ClusterIP
EOF
```

4. Entao recriar o kube-dns service:

```sh
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: CoreDNS
  name: kube-dns
  namespace: kube-system
spec:
  clusterIP: 100.96.0.10
  clusterIPs:
  - 100.96.0.10
  ports:
  - name: dns
    port: 53 
    protocol: UDP
    targetPort: 53
  - name: metrics
    port: 9153 
    protocol: TCP
    targetPort: 9153
  selector:
    k8s-app: kube-dns
  type: ClusterIP
EOF
```
5. Ajustar o kubelet
pode ser feito em dois lugares dentro do arquivo yaml emm manifestos como o apiserver

```sh
vim /var/lib/kubelet/config.yaml
```

alterar
clusterDNS:
- 100.96.0.10

E Editar o configmap associado ao kubelet

```sh
kubectl edit cm kubelet-config -n kubes-system
```

alterar
clusterDNS:
- 100.96.0.10

Reload o daemon do kubelet e restart no servico do kubelet

6. Atualizar kubelet no node
```sh
kubeadm upgrade node phase kubelet-config
```

e restart com
```sh
systemctl daemon-reload
systemctl restart kubelet
```

> Adicionalmente se nao funcionar logar no node worker docker exec -it kind-worker e alterar o mesmo arquivo do kubelet e restartar via systemctl. Entao lancar o netshoot novamente e deve funcionar.


2. In your cluster, install an ingress controller to proxy communica-
tion into the cluster via an ingress resource. Then create a Deployment
named hello using the image nginxdemos/hello:plain-text. The container is
exposed on port 80. Create a Service named hello-svc that targets the hello
Deployment on port 80. Then create an ingress resource that will allow you to resolve the DNS name hello.com to the ClusterIP Service named hello-svc in Kubernetes.

Resolucao:

Criar um deployment e expor ele utilizando um service

```sh
kubectl create deploy hello --image=nginxdemos/hello:plain-text --port 80
kubectl expose deploy hello --name=hello-svc --port=80
````

Em seguida criar um ingress apotando pra o svc hello-svc

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello
  namespace: default
spec:
  rules:
  - host: hello.com
    http:
      paths:
      - backend:
          service:
            name: hello-svc
            port:
              number: 80
        pathType: ImplementationSpecific
```

Adicionar em /etc/hosts utilizando o mesmo IP do node com o host hello.com

```
root@kind-control-plane:/# cat  /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::	ip6-localnet
ff00::	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
*172.19.0.2*	kind-control-plane
fc00:f853:ccd:e793::2	kind-control-plane
*172.19.0.2*     hello.com
```

curl hello.com