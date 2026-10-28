As configurações de todos os manifestos do Kubernetes são armazenadas em:
/etc/kubernetes/manifests - dentro do control-plane node.


Durante o exame, é preciso acessar o controlplane utilizando SSH. As chaves estão compartilhadas, então dá para digitar somente:
```bash
ssh control-plane-node 
```


Dica:
Utilize o comando kubectl auth can-i para verificar se você concluiu a tarefa com sucesso.

Adicionalmente, o comando "--as" permite a impersonação de outros usuários, e "--list" lista todas as permissões de um usuário.

É possível associar uma cluster role a múltiplos usuários, groups ou Service Accounts via role ou cluster role binding. Durante o exame, pode-se economizar tempo criando somente uma role binding e atribuindo-a a uma role existente (view).
