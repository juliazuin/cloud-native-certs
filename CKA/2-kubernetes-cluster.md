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

Nesse caso a _key_ é **node-role.kubernetes.io**, e o _effect_ é **NoSchedule**.


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
