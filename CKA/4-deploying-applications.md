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
