### Cluster architecture, installation and configuration

No Kubernetes existem dois tipos de nodes:

Controlplane node:
 
    Componentes:
        * API server - é o ponto de entrada para todas as requisições de gerenciamento do cluster;
        * DNS;
        * Controller manager - literalmente um controller dos controllers, gerencia os controllers do cluster incluindo deployments, replicasets, etc. Um Controller é resposavel por comparar o estado atual com o desejado ex: o controller vai escalar um deployment se o número de replicas for menor do que o desejado;
        * scheduler;
        * `kube-proxy`;
        * `etcd` datastore.

Workers nodes: contendo o `kube-proxy` e o kubelet.

Conheçendo o Kubernetes:
- Deployments
- Replicasets
- Pods
- Services
- Storage
