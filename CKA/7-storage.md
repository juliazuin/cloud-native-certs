## 7. Storage in Kubernetes

### Persistent Volumes (PV) e Persistent Volume Claim (PVC)

PV - Recurso Kubernetes que não tem escopo de namespace. O volume pode estar nos seguintes estados:

- Available
- Bound
- Released - quando a reclaim policy=retain, esse status significa que pode ser utilizado por outro PVC
- Failed

PVC - Recurso Kubernetes que tem escopo de namespace, é utilizado para "reivindicar" um PV, e então um workload (deployment, daemonset, etc) vai referenciá-lo para acessar os dados. O PVC vai escolher o PV que tem o storage o mais próximo possível. O papel dele é reservar esse storage em nome do container, quando entra em status `Bound`, o PVC não vai liberar o volume.

### hostPath

É um tipo de volume persistente que se monta em um diretório local, significando que montaria o volume em um dos nodes, quando o pod morre os dados ainda estão no diretório local do node, porém nem sempre um pod cai no mesmo node, então o dado ficaria inconsistente pois não existe sincronização de dados entre os nodes.

Para hostPath somente access mode `ReadWriteOnce` é suportado!

### NFS volume

Permite guardar os dados num `nfs share` na rede, ele também permite múltiplos `writers` significando que vários pods podem utilizá-lo simultaneamente.

Exemplo:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 1Mi
  accessModes:
  - ReadWriteMany
  nfs:
    server: nfs-server.default.svc.cluster.local
    path: "/"
  mountOptions:
  - nfsvers=4.2
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Mi
  volumeName: nfs-pv
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nfs-pod
  name: nfs-pod
spec:
  containers:
  - image: nginx
    name: nfs-pod
    volumeMounts:
    - name: nfs-vol
      mountPath: "/usr/share/nginx/html"
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: nfs-vol
    persistentVolumeClaim:
      claimName: nfs-pvc
```


### Volume modes

- Filesystem: é o modo default, no fim fica montado no pod como um diretório.
- Block: apoiado por um device `block` e esse device está vazio (não formatado), exemplos de block system: AWS EBS, Azure Disk


### Access modes

Determina por quais permissões e de quais aplicações o volume será acessado

por exemplo: no arquivo [pv.yml](./manifests/pv.yml) access mode é `ReadWriteOnce`, significando que, pensando na perspectiva do node, sendo o node que está acessando o volume, somente um node pode montar um volume pra leitura e escrita nele.

Existem também `ReadOnlyMany`, que significa que múltiplos nodes podem montar volumes de leitura somente; `ReadWriteMany`, múltiplos nodes podem montar o volume pra escrita e leitura e `ReadWriteOncePod` que somente um pode de um node pode montar volume pra leitura e escrita.


Um PV pode ter múltiplos access modes, mas o PVC só pode ter um. Então ainda que estejam listam vários access modes no pv, o node só vai utilizar um por vez.

### Resumo

| Access mode      | Short form | Description                                                                 |
| ---------------- | ---------- | --------------------------------------------------------------------------- |
| ReadWriteOnce    | RWO        | Only one node can mount the volume and read and write to it                 |
| ReadOnlyMany     | ROX        | Multiple nodes can mount the volume for reading only                        |
| ReadWriteMany    | RWX        | Multiple nodes can mount the volume for writing and reading                 |
| ReadWriteOncePod | RWOP       | Only one pod on that one node can mount that volume for reading and writing |


### Reclaim policy

Decisão do que acontece com o PV quando o PVC for deletado, default para `Retain`, também tem valores de `Recycle` e `Delete`.

- Retain > o volume pode ser usado de novo e ele sempre retém o dado no volume, então pode ser usado por outro claim (PVC);
- Recycle(depreciado) > O volume é formatado (rm -f)
- Delete > o storage "backend" (ex, EBS, Azure disk) foi deletado, então o PV também será.


### Storage Class

- manual
- standard: nesse modelo o PVC quando criado fica em status de pending se NÃO houver nenhum pod o consumindo (um consumer, no caso um pod)

### EmptyDir - volumes não persistentes

Nesse modelo o volume toma um pedaço do próprio container, que dura somente o tempo de vida do pod.