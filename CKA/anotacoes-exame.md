## Anotações para o exame

As configurações de todos os manifestos do Kubernetes são armazenadas em `/etc/kubernetes/manifests`, dentro do control-plane node.


Durante o exame, é preciso acessar o control-plane utilizando SSH. As chaves estão compartilhadas, então dá para digitar somente:
```bash
ssh control-plane-node
```


### Dica

Utilize o comando `kubectl auth can-i` para verificar se você concluiu a tarefa com sucesso.

Adicionalmente, o comando `--as` permite a impersonação de outros usuários, e `--list` lista todas as permissões de um usuário.

É possível associar uma ClusterRole a múltiplos usuários, grupos ou ServiceAccounts via RoleBinding ou ClusterRoleBinding. Durante o exame, pode-se economizar tempo criando somente uma RoleBinding e atribuindo-a a uma role existente (`view`).


### Network Policies

Pode ser que seja pedido para criar uma NetworkPolicy que afete todo o namespace.

### Gateway API

A Gateway API é sucessora do Ingress; o exame pode pedir para transformar um Ingress em uma Gateway API.