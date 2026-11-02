## 1. Cluster architecture, installation and configuration

No Kubernetes existem dois tipos de nodes:

### Control plane node

Componentes:

- API server - é o ponto de entrada para todas as requisições de gerenciamento do cluster;
- DNS;
- Controller manager - literalmente um controller dos controllers, gerencia os controllers do cluster incluindo deployments, replicasets, etc. Um Controller é responsável por comparar o estado atual com o desejado ex: o controller vai escalar um deployment se o número de réplicas for menor do que o desejado;
- scheduler;
- `kube-proxy` - Não é exatamente um proxy, é um controller que observa a API pra novos services (kind: Service) e cria regras de iptables para rotear o tráfego para os pods corretos;
- `etcd` datastore - guarda todos os dados de configuração do kubernetes (o estado). É um banco de dados chave-valor distribuído que armazena todos os dados em memória.

### Worker nodes

Contendo o `kube-proxy` e o kubelet.

O diretório /etc/kubernetes/manifests contém arquivos YAML que descrevem os pods. Qualquer arquivo que esteja nesse diretório será criado automaticamente.

### Conhecendo o Kubernetes

- Deployments
- Replicasets
- Pods
- Services
- Storage

> Alguns tipos de storage e como eles são utilizados:
> - ext4 (Fourth Extended Filesystem) - sistema de arquivos padrão para a maioria das distribuições Linux guarda as informações em blocos e utiliza journals que registram as alterações antes de serem aplicadas, que garante a integridade do sistema de arquivos em caso de falhas;
> - xfs (eXtended File System) - sistema de arquivos de alto desempenho, utilizado para bancos de dados grandes e pesados, como MySQL e PostgreSQL, e ambientes que armazenam grandes quantidades de dados como data lakes e vídeos 4k/8k;
> - NTFS (New Technology File System) - sistema de arquivos padrão do Windows.
