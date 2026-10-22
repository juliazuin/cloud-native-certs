Configuracoes de todos manifestos kubernetes são armazenados em:
/etc/kubernetes/manifests - dentro do control-plane node.


Durante o exame tem q acessar o controlplane utilizando SSH. As chaves estao compartilhadas entao da pra digitar somete
```bash
ssh control-plane-node 
```


Dica:
utilizar o comando kubectl auth can-i -> vai permitir que voce verifique que completou a tarefa com sucesso

adicionais, comandos "--as" permite impersonificacao de outros usuarios e "--list" lista todas permissoes de um usuario

É possivel atachar uma cluster role a multilpos usuarios/groups/service Accounts via role ou cluster rolebinding. Durante o exame pode salvar tempo somente criando uma role binding e atribuindo a uma role existente (view)

