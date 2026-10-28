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


## 2. Kubernetes Cluster

### Como atualizar o cluster

Depois de subir o cluster com kind, utilizamos o kubectl de dentro do container, pois já existem todas as ferramentas necessárias para executar essa ação.

1. Verificar a versão atual do cluster:
   ```bash
   docker exec -it kind-control-plane bash
   ```

2. Verificar a disponibilidade de atualizações do cluster:
   ```bash
   kubeadm upgrade plan
   ```

3. Se for necessário atualizar o kubeadm conforme output do comando anterior, execute:

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

   > Note que a versão do kubeadm muda, e os comandos curl e printf devem ser atualizados conforme a versão do kubeadm que deseja instalar.

4. Atualizar o cluster:
   ```bash
   kubeadm upgrade apply v1.35.9
   ```

### Taints and tolerations

Taint é composto de três partes: _key_, _value_ e _effect_. O efeito pode ser NoSchedule, PreferNoSchedule ou NoExecute.
Não necessariamente precisa de um value, mas a key e o effect são obrigatórios.

Exemplo:

```text
node-role.kubernetes.io/control-plane:NoSchedule. 
```

Nesse caso a _key_ é
**node-role.kubernetes.io**, e o _effect_ é **NoSchedule**


### Interfaces

- Container Runtime Interface (CRI):
  - Define como o kubelet fala com o container runtime (exemplo: start/stop containers, pull de imagens, etc). Essencialmente, provê uma forma padronizada do Kubernetes lançar, gerenciar e parar container sem ficar preso em apenas uma implementação (era somente Docker)
  - crictl é a ferramenta utilizada para interagir com o CRI
  - crictl é uma ferramenta que roda dentro de cada node, então para realizar troubleshooting de containers, é necessário acessar o node onde está localizado o pod e executar o crictl.

- Container Network Interface (CNI):
  - É especificação e biblioteca para configurar _network interfaces_ nos containers Linux.
  - No Kubernetes define como a rede do pod é configurada e gerenciada.

- Container Storage Interface (CSI)
- Custom Resource Definitions (CRD)


## 3. Identity and access management

Tudo que tenta acessar o Kubernetes precisa se autenticar por meio de um certificado ou token.

### Fase de autenticação

Kubernetes suporta vários tipos de métodos de autenticação. O built-in plugin que utilizamos com kubectl é chamado certificate-based authentication e utiliza um certificado X.509.

Quando uma requisição é feita para o Kubernetes API Server, ele tenta autenticar utilizando o mecanismo configurado (como client certificates, bearer token, OpenID Connect, WebHooks, etc).

### Fase de autorização

Role based access control (RBAC) é o mecanismo mais popular (mas tem outros como Node Authorization plugin, AlwaysAllow, ABAC e WebHook authorization plugin).

O modo de autorização do Kubernetes é configurado no API server com o parâmetro `--authorization-mode=RBAC`.


### Roles e role bindings

Como o RBAC funciona com base no menor acesso possível, ele sabe quais são as ações permitidas para o usuário ou grupo, com base em 4 tipos diferentes de objetos: Roles, ClusterRoles, RoleBindings e ClusterRoleBindings.

[roles-demo](./img/roles-cluster-role.png)


Uma role tem escopo de um namespace específico que define permissões para recursos específicos, como pods. Cada role lista ações de "allow" utilizando verbos (GET, LIST, CREATE, DELETE), que serão traduzidos em operações de API. Eles não são os mesmos que métodos HTTP, embora sejam conceitualmente similares.

Mapa de combinações suportadas entre role, role binding, cluster role e cluster role binding.

| Binding | Role |
| --- | --- |
| Cluster role binding | cluster role |
| role binding | role e cluster role |

### System roles e grupos

Algumas roles bindings criadas embutidas no cluster atribuem cluster role para alguns grupos internos como:
- system:authenticated
- system:unauthenticated
- system:masters ou kubeadm:cluster-admins


## 4. Deploying applications in Kubernetes

### Node selector e affinity
Node selectors são utilizados para designar pods a certos nodes por vários motivos. Pode ser que um node tenha SSD em vez de HDs, ou que alguns pods tenham arquiteturas diferentes e, por isso, os nodes também sejam diferentes.

Além disso, existem `affinities` que são utilizadas em pods e nodes, mas de uma forma mais "soft": o affinity *prefere* que um pod seja schedulado em um node, porém o node selector é obrigatório.

- **required**DuringSchedulingIgnoredDuringExecution (hard rules)
- **preferred**DuringSchedulingIgnoredDuringExecution (soft preferences)


### Requests/limits

No Kubernetes, CPU é medida em unidades de CPU. Uma unidade de CPU é equivalente a 1 unidade física de um core de CPU ou um core virtual de CPU. É possível escrever os valores em `milicores`; então, 1000m = 1 CPU e 0.5 CPU = 500m.

E memória é medida em `mebibytes`, que é uma medida baseada em potências de 2 (1 mebibyte = 2ˆ20 ou 1.048.576 bytes).

> emptyDir é um dos muitos tipos de volume que são efêmeros e que morrem junto com o pod.

### Priority Class

Define quanto um pod é mais importante que outro em termos de "scheduling", especialmente quando o node fica sem recursos.

Existem 2 priority classes padrão no Kubernetes.
```bash
system-cluster-critical   2000000000   false            22h   PreemptLowerPriority
system-node-critical      2000001000   false            22h   PreemptLowerPriority
```

Quanto maior o número, maior a prioridade.

## 5. Running applications in Kubernetes

Deployments usam uma estratégia de deploy das aplicações. Em caso de alteração de imagem ou de outros parâmetros, a configuração fica em spec.strategy e o padrão é rollingUpdate, que mantém alguns pods disponíveis até que os novos com a alteração também fiquem prontos. Outro tipo de estratégia é `recreate strategy`; nesse caso, ele vai terminar as réplicas antigas antes de criar as novas.


### Cordon e drain nodes

Passo a passo:

1. Primeiro marcamos o worker como "não schedulável":
   ```sh
   kubectl cordon kind-worker
   ```

2. No caso, aqui só temos 1 worker e o controlplane. Devemos remover o taint do controlplane para que os pods consigam ser agendados:
   ```sh
   kubectl taint node  kind-control-plane node-role.kubernetes.io/control-plane:NoSchedule-
   ```

3. Depois drenar o worker:
   ```sh
   kubectl drain kind-worker --ignore-daemonsets
   ```

4. E habilitar novamente o agendamento de pods no worker:
   ```sh
   kubectl uncordon kind-worker
   ```


### Adicionando nodes em um cluster

```sh
kubeadm token create --print-join-command
```

O comando de join gerado será semelhante a:

```sh
kubeadm join kind-control-plane:6443 --token t5mckr.a6jg5okxqdse2tkl --discovery-token-ca-cert-hash sha256:0b38c6cbab32974ee7ac38a46343267093614dba5e41e673b24013d8e53227d6
```

Para executar isso vamos criar um novo cluster com apenas 1 nó e deletando o worker node desse novo cluster.
Depois disso vamos dar exec dentro desse worker para resetá-lo para limpar sua afiliação com o cluster antigo

```sh
docker exec -it cka-worker bash
```

```sh
kubeadm reset
```

Em seguida, aplicamos o comando de join que foi output do primeiro passo. Ao entrar novamente no cluster principal, devemos ter um node adicional em estado "Ready".


## 6. Communication in a Kubernetes cluster

### CoreDNS

Responsável por traduzir hostnames em endereços IP. Existem vários ranges de endereços IP em um cluster, e todos eles devem ser únicos.

### Ingress e ingress controllers

Para o ingress existem 3 tipos de pathtypes

- **Exact** - nesse caso o path tem que combinar exatamente o path sendo case-sensitive para dar certo
- **Prefix** - o path corresponde à URL dividido por / onde os caracteres entre as barras são o elemento
- **ImplementationSpecific** - deixa pra o `ingress class` decidir um tipo prefix ou exact path

### Services

#### ClusterIP

Feitos para deixar os pods disponíveis apenas dentro do cluster, para comunicação interna.

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: internal-svc
  name: internal-svc
spec:
  clusterIP: None # headless service
  ports:
  - name: 8080-80
    port: 8080 # porta que vai estar exposto o próprio service
    protocol: TCP
    targetPort: 80 # a porta interna do container
  - name: search
    port: 9200
    protocol: TCP
    targetPort: 9200
  selector:
    app: internal-svc # selector é como o svc encontra e se "liga" ao pod
  type: ClusterIP
status:
  loadBalancer: {}
```


`headless service` -> É possível dar bypass no service indo diretamente para o pod, pra isso setamos spec.clusterIP: None, isso vai remover o endereço de IP do service e então vai retornar todos os IPs dos pods que estão atrelados. É um caso de uso útil pra databases clusters, que uma cópia do banco é responsável pela escrita e as demais só tem permissão de leitura -> statefulsets normalmente usam headless services


#### NodePort e LoadBalancer

Feitos para expor portas e criar acesso de fora do cluster, NodePort expõe uma porta no endereço de IP do node mas o LoadBalancer permite que um loadbalancer externo controle o tráfego em cada node port.

Nesse modelo, o service não apenas se comunica internamente com outros componentes do cluster, mas também consegue expor uma porta no node que pode ser utilizada para tráfego externo.

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: "no"
  name: "no"
spec:
  ports:
  - name: 443-80
    nodePort: 30000 # porta exposta em cada node do cluster
    port: 443
    protocol: TCP
    targetPort: 80
  selector:
    app: "no"
  type: NodePort
status:
  loadBalancer: {}
```


#### LoadBalancer

Um service tipo load balancer, podendo ser um recurso provisionado na cloud ou on-premises.

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: lb-svc
  name: lb-svc
spec:
  ports:
  - name: 8080-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: lb-svc
  type: LoadBalancer
status:
  loadBalancer: {}
```


#### ExternalName

Não aloca nenhum endereço de clusterIP nele, apenas cria um registro CNAME dentro do cluster DNS

### Cluster node networking configuration

`Network Overlay` - essencialmente, são abstrações de rede "em cima" da rede que já temos no cluster. Isso existe porque cada pod dentro do cluster precisa de um IP único próprio e, muitas vezes, isso não é possível. Esse termo também pode ser encontrado como *VXLAN*.

Uma VXLAN fica no topo de uma rede física e usa encapsulação do tunnel layer-2 (OSI model) para layer-3. Durante a comunicação entre nodes, quando tentamos comunicação com um pod, esse encapsulamento ajuda a rotear para o destino correto, incluindo um header no pacote.

No cluster, quem faz isso é o CNI (container network Interface), que roda como um daemonset, criando uma camada virtual de rede no namespace, que faz um "meio de campo" entre pacotes indo de um node para outro.


### Network Policies 
Mecanismo para previnir que os pods acessem redes externas, 
reforcando isolamento a nivel de namespace, 
filtrando ingress em portas TCP especificas, 
restringindo comunicacao pod-to-pod 