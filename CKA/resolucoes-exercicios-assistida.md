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
- Nodes CIDR: também conhecidos como `internal-IP`, essa é a camada subjacente da cloud/VLAN subnet à qual os workers e o controlplane se conectam. O Cloud provider gerencia esse range.

- Pod CIDR: também conhecido como `Cluster-CIDR`, durante a criação os pods recebem um endereço IP e o range é configurado na instalação do cluster Kubernetes e gerenciado pelo container network interface (CNI, network plugin).

- Service CIDR: também conhecido como `Service-Cluster-IP-Range`, porque pods são recriados a qualquer momento, então existe a necessidade de abstrair o grupo de pods em um único range de IP dentro do cluster. Esse range é gerenciado e configurado pelo API Server. ServiceCIDR é um tipo de recurso que não fica em namespace e define um range de endereços de virtual IPs que podem ser atribuídos a services do tipo ClusterIP. Sendo assim, todo service desse tipo tem um IP virtual estável dentro do range do CIDR especificado.

Resolução:

1. Atualizar o range do service-CIDR — que é controlado pelo API Server — dentro do controlplane.
```sh
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

Alterando a linha `--service-cluster-ip-range=100.96.0.0/16`.

Aguardar o pod do APIServer reiniciar.

2. 
```sh
kubectl get servicecidr
```

Precisamos deletar porque está com o endereço IP antigo, mas antes devem-se deletar os svcs associados a ele.

```sh
kubectl delete svc kube-dns -n kube-system
kubectl delete svc kubernetes
kubectl delete servicecidr kubernetes
```

3. Recriar o kubernetes service se não for recriado automaticamente.

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

4. Então, recriar o kube-dns service:

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
5. Ajustar o kubelet.
Pode ser feito em dois lugares: dentro do arquivo YAML e em manifestos como o apiserver.

```sh
vim /var/lib/kubelet/config.yaml
```

Alterar:
clusterDNS:
- 100.96.0.10

E editar o configmap associado ao kubelet:

```sh
kubectl edit cm kubelet-config -n kube-system
```

Alterar:
clusterDNS:
- 100.96.0.10

Recarregar o daemon do kubelet e reiniciar o serviço do kubelet.

6. Atualizar kubelet no node
```sh
kubeadm upgrade node phase kubelet-config
```

E reiniciar com:
```sh
systemctl daemon-reload
systemctl restart kubelet
```

> Adicionalmente, se não funcionar, faça login no node worker com docker exec -it kind-worker, altere o mesmo arquivo do kubelet e reinicie via systemctl. Então, execute o netshoot novamente; deve funcionar.


2. In your cluster, install an ingress controller to proxy communica-
tion into the cluster via an ingress resource. Then create a Deployment
named hello using the image nginxdemos/hello:plain-text. The container is
exposed on port 80. Create a Service named hello-svc that targets the hello
Deployment on port 80. Then create an ingress resource that will allow you to resolve the DNS name hello.com to the ClusterIP Service named hello-svc in Kubernetes.

Resolução:

Criar um deployment e expô-lo utilizando um service.

```sh
kubectl create deploy hello --image=nginxdemos/hello:plain-text --port 80
kubectl expose deploy hello --name=hello-svc --port=80
````

Em seguida, criar um ingress apontando para o svc hello-svc.

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

Adicionar em /etc/hosts utilizando o mesmo IP do node, associado ao host hello.com.

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