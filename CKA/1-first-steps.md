## 1. Cluster architecture, installation and configuration

No Kubernetes existem dois tipos de nodes:

Controlplane node:
 
    Componentes:
        * API server - é o ponto de entrada para todas as requisições de gerenciamento do cluster;
        * DNS;
        * Controller manager - literalmente um controller dos controllers, gerencia os controllers do cluster incluindo deployments, replicasets, etc. Um Controller é resposavel por comparar o estado atual com o desejado ex: o controller vai escalar um deployment se o número de replicas for menor do que o desejado;
        * scheduler;
        * `kube-proxy` - Não é exatamente um proxy, é um controller que observa a API pra novos services (kind: Service) e cria regras de iptables para rotear o tráfego para os pods corretos;
        * `etcd` datastore - guarda todos os dados de configuração do kubernetes (o estado). É um banco de dados chave-valor distribuído que armazena todos os dados em memória.

Workers nodes: contendo o `kube-proxy` e o kubelet.

O diretório /etc/kuberntes/manifests contem arquivos YAML que descrevem os pods. Qualquer arquivo que esteja nesse diretorio sera criado automaticamente

Conheçendo o Kubernetes:
- Deployments
- Replicasets
- Pods
- Services
- Storage

> Alguns tipos de storage e como eles são utilizados:
> - ext4 (Fourth Extended Filesystem) - sistema de arquivos padrão para a maioria das distribuições Linux guarda as informacoes em blocos e utiliza journals que registram as alteracoes antes de serem aplicadas, que garante a integridade do sistema de arquivos em caso de falhas;
> - xfs (eXtended File System) - sistema de arquivos de alto desempenho, utilizar pra bancos de dados grandes e pesados mysql, postegreSQL, e ambientes que armazenam grandes quantidades de dados como data lakes, dados de video 4k/8k, etc;
> - NTFS (New Technology File System) - sistema de arquivos padrão do Windows.


## 2. Kubernetes Cluster

Como atualizar o cluster:

Depois de subir o cluster com kind, utilizamos o kubectl de dentro do container pois ja existem todas as ferramentas necessarias para executar essa acao.

1. Verificar a versão atual do cluster:
```bash
docker exec -it kind-control-plane bash
```

2. Verificar a disponibilidade de atualizações do cluster:
```bash
kubeadm upgrade plan
```

3. Se for necessario atualizar o kubeadm conforme output do comando anterior, execute:

```bash
apt update
apt install -y gpg
```

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key \
  | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

```bash
printf '%s\n' \
'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' \
> /etc/apt/sources.list.d/kubernetes.list
```

```bash
apt update

apt install -y kubeadm=1.35.0-*
```

> note que a versao do kubeadm muda e o comando de curl e o printf devem ser atualizados conforme a versao do kubeadm que deseja instalar.

4. Atualizar o cluster:
```bash
kubeadm upgrade apply v1.35.9
```

#### Taints and tolerations

Taint ê composto de tres partes: _key_, _value_ e _effect_. O efeito pode ser NoSchedule, PreferNoSchedule ou NoExecute.
Não necessariamente precisa de um value mas a key e o effect são obrigatórios.

exemplo:
node-role.kubernetes.io/control-plane:NoSchedule. 

Nesse caso a _key_ é
**node-role.kubernetes.io**, e o _effect_ é **NoSchedule**


Interfaces:
- Container Runtime Interface (CRI):
  ->  Define como o kubelet fala com o container runtime (exemplo: start/stop containers, pull de imagens, etc). Essencialmente, provê uma forma padronizada do Kubernetes lançar, gerenciar e parar container sem ficar preso em apenas uma implementacão (era somente Docker)
  -> crictl é a ferramenta utilizada para interagir com o CRI
  -> crictl é uma ferramenta que roda dentro de cada node, entao para realizar troubleshooting de containers, é necessário acessar o node onde esta localizado o pod e executar o crictl.

- Container Network Interface (CNI):
  -> É especificacção e biblioteca para configurar _network interfaces_ nos containers Linux. 
  -> No Kubernetes define como a rede do pod é configurada e gerenciada.
- Container Storage Interface (CSI)
- Custom Resource Definitions (CRD)


## 3. Identity and access management

Tudo que tenta acessas o kubernetes precisa estar autenticado via certificado ou token pra se autenticar com.

Fase de autenticação:

Kubernetes suporta varios tipos de metodos de autenticao. O buil-in plugin que utilizamos com kubectl é chamado certificate-based authentication utilizando um certificado X.509.

Quando uma requisicao é feita para o Kubernetes API Server, ele tenta autenticar utilizando o mecanismo configurado (como client certificates, bearer token, OpenID Connect, WebHooks, etc). 

Fase de autorização:
Role based access control (RBAC) é o mecanismo mais popular (mas tem outros como Node Authorization plugin, AlwaysAllow, ABAC e WebHook autorization plugin).

O modo de autorizacao do kuberntes é configurado no API server com o parametro `--authorization-mode=RBAC`


Roles e role bindings:

Como o RBAC fuciona baseado no menor acesso possivel, ele sabe que as acoes permitidas pelo usuario ou grupo baseado em 4 tipo diferentes de objetos: Roles, ClusterRoles, RoleBindings e ClusterRoleBindings.

[roles-demo](./img/roles-cluster-role.png)


Uma role tem escopo de um namespace especifico q define permissoes para recursos especificos como pods. Cada role lista acoes de "allow" utilizando verbos (GET, LIST, CREATE, DELETE) que serao traduzidos em operacoes de API mas nao sao os memos que metodos HTTP porem sao conceitualmente similares.

Mapa de combinacoes suportadas entre role, role binding, cluster role e cluster role binding

Cluster role binding -> cluster role
role binding -> role e cluster role

System roles e grupos
Algumas roles bindings criadas embutidas no cluster atribui cluster role para alguns grupos internos como:
- system:authenticate
- system:unauthenticated
- system:masters ou kubeadm:cluster-admins


## Deploying applications in Kubernetes

### Node selector e affinity
Node selectors, sao utilizados para designar pods a certos nodes, por varios motivos, pode ser que um node tenha ssd ao inves de HDs, pode ser que alguns pods tenham arquitetura diferentes e isso faz com q os nodes tambem sejam.

Alem disso existem `affinitys` que são utilizadas em pods e nodes tambem mas de uma forma mais "soft", o affinity *prefere* que um pod seja schedulado em um node, porem o node selector é obrigatorio.

**required**DuringSchedulingIgnoredDuringExecution (hard rules) and 
**preferred**DuringSchedulingIgnoredDuringExecution (soft preferences).


### Requests/limits

No kubernetes CPU é medida em unidades de CPU, uma unidade de CPU é equivalente a 1 unidade fisica de um core de CPU ou um core virtual de CPU, é possivel escrever os valores em `milicores` entao 1000m = 1 CPU, 0.5 CPU = 500m.

E memoria é medida em `mebibytes`, q é uma medida baseada em potencias de 2 (1 mebibyte=2ˆ20 ou 1.048.576 bytes)

> emptyDir é um dos muitos tipos de volume que sao efemeros e que morrem junto com o pod.

### Priority Class

Define quanto um pod é mais importante que o outro em termos de "schedulling", especialmente quando o node fica sem recursos.

Existem 2 priority class default no kubernetes
```bash
system-cluster-critical   2000000000   false            22h   PreemptLowerPriority
system-node-critical      2000001000   false            22h   PreemptLowerPriority
```

Quanto maior o numero maior a prioridade

## Running applications in kubernetes

Deployments usam uma estrategia de deploy das aplicacoes, em caso de alteracao de imagem ou de outros parametros, a configuracao fica em spec.strategy e o default é rollingUpdate, que mantem alguns pods disponiveis ate que os novos com a alteracao fiquem up tambem. Outro tipo de estrategia `recreate strategy`, nesse casoele vai terminar as replicas antigas antes de criar as novas.


Cordon e drain nodes

Passo a passo

Primeiro marcamos o worker como "nao schedulavel"
```sh
kubectl cordon kind-worker 
```

No caso aqui so temos 1 worker e o controlplane, devemos remover o taint o controlplane para q os pods consigam ser agendados
```sh
kubectl taint node  kind-control-plane node-role.kubernetes.io/control-plane:NoSchedule-
```

Depois drenar o worker 
```sh
kubectl drain kind-worker --ignore-daemonsets 
```

E habilitar novamente o agendamento de pods no worker
```sh
kubectl uncordon kind-worker 
```


Adicionando nodes em um cluster

```sh
kubeadm token create --print-join-command

> kubeadm join kind-control-plane:6443 --token t5mckr.a6jg5okxqdse2tkl --discovery-token-ca-cert-hash sha256:0b38c6cbab32974ee7ac38a46343267093614dba5e41e673b24013d8e53227d6
```

Para executar isso vamos criar um novo cluster com apenas 1 nó e deletando o worker node desse novo cluster.
Depois disso vamos dar exec dentro desse worker para reseta-lo para limpar sua afiliacao com o cluster antigo

> docker exec -it cka-worker bash

```sh
kubeadm reset
```

Em seguida aplicamos o comando de join que foi output do primeiro passo, logando novamente no cluster principal é pra termos um node adicional em estado "Ready"


## Communication in a Kubernetes cluster
CoreDNS -  responsavel por traduzier hostname em enderecos de IP, existem varios ranges de enderecos de IP em um cluster e todos eles devem ser UNICOS

Ingress e ingress controllers

Para o ingress existem 3 tipos de pathtypes
Exact - nesse caso o path tem que combinar exatamente o path sendo case-sensitive para dar certo
Prefix - o path corresponde a URL dividido por / onde os caracteres entre as barras sao o elemento
ImplementationSpecific - deixa pra o `ingress class` decidir um tipo prefix ou exact path 


Services:

ClusterIP - feitos para fazer os pods internamente disponiveis apenas dentro do cluster

NodePort - LoadBalancer - feitos para expor portas e criar acesso de fora do cluster, NodePort expoe uma porta no endereco de IP do node mas o LoadBalancer permite que um loadbalancer externo controle o trafego em cada node port.

ExternalName - nao aloca nenhum endereco de clusterIP nele, apenas cria um registro CNAME dentro do cluster DNS

 