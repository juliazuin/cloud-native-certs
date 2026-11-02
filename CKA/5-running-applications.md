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
