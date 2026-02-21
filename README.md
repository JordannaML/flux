# Teste FluxCD

Bom, vamos lá...

Eu AMMMEI a ferramenta, nao conhecia e nao fazia ideia do quanto ela e legal. Vou usar com certeza no meu dia a dia.

---

## Primeiro Ponto

criei a app do nginx via
```bash
kubectl apply -k .
```

e tambem criei um kustomization.yaml dentro da mesma pasta, anteriormente eu usei um spec de label que estava defazado, e acabeio recebendo um warning por conta disso, mas pesquizando na internet ddescobri um comando que faz uma correcao do arquivo de forma altomatica, o comando foi
```bash
kustomize edit fix
```
```
Warning: 'commonLabels' is deprecated. Please use 'labels' instead.
Run 'kustomize edit fix' to update your Kustomization automatically.

Fixed fields:
patchesJson6902 -> patches
patchesStrategicMerge -> patches
commonLabels -> labels
```

Eu AMEI

---

## Segundo Ponto

fiquei em duvida sobre o arquivo podinfo.yaml, vai dar para ver que mudei ele de lugar algumas vezes pq quando procurava por algum kustomization ele nao me voltava um lugar, mas me falava que existia
```bash
kubectl get kustomization podinfo
```
```
NAME      AGE   READY   STATUS
podinfo   81m   True    Applied revision: main@sha1:ed283a5e05a4f2921e2ce67ca1b171a608cb1cfe
```
```bash
kubectl get kustomization podinfo -A
```
```
error: a resource cannot be retrieved by name across all namespaces
```

e nesse momento eu vi a magica acontecendo de fato, alterei o kustomization.yaml no path do flux-system, adicionando o podinfo.yaml nele, e coloquei o yaml dentro desse mesmo path (flux-system), tambem vi que esse kind deve ficar dentro desse mesmo fluxo, fiz o commit apenas e assim que subiu as alteracoes vi dentro do cluster ele sendo criado dentro do fluxo que defini no yaml
```bash
kubectl get kustomizations -n flux-system
```
```
NAME          AGE    READY   STATUS
flux-system   106m   True    Applied revision: main@sha1:7788a72bc0ddb1aef210208f3d14f0ce96a86e11
podinfo       2s     True    Applied revision: main@sha1:7788a72bc0ddb1aef210208f3d14f0ce96a86e11
```

Outra alteracao que fiz foi vendo essa doc aqui sobre estrutura de pasta https://fluxcd.io/flux/guides/repository-structure/ E com ela eu alterei tambem o meu path de onde o kustomize iria ficar
```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: podinfo
  namespace: flux-system
spec:
  interval: 10m
  targetNamespace: default
  sourceRef:
    kind: GitRepository
    name: podinfo
  path: "./apps/kustomize"
  prune: true
  timeout: 1m
```

Entao agora esta tudo rodando conforme o esperado:
```bash
kubectl get pods -A
```
```
NAMESPACE     NAME                                           READY   STATUS    RESTARTS   AGE
flux-system   helm-controller-74988df57b-d8qpk              1/1     Running   0          123m
flux-system   image-automation-controller-86497776cd-cc26n  1/1     Running   0          147m
flux-system   image-reflector-controller-684c57479b-47gwt   1/1     Running   0          147m
flux-system   kustomize-controller-6b646448f6-zd4kf         1/1     Running   0          123m
flux-system   notification-controller-6c9f6f77d8-bt59l      1/1     Running   0          123m
flux-system   source-controller-56c7f45479-hh9lp            1/1     Running   0          123m
kube-system   coredns-7d764666f9-8cb2m                      1/1     Running   0          3h52m
kube-system   coredns-7d764666f9-mmq98                      1/1     Running   0          3h52m
kube-system   etcd-teste-cluster-control-plane              1/1     Running   0          3h52m
kube-system   kindnet-qnqj7                                 1/1     Running   0          3h52m
kube-system   kube-apiserver-teste-cluster-control-plane    1/1     Running   0          3h52m
kube-system   kube-controller-manager-teste-cluster-control-plane  1/1  Running  0       3h52m
kube-system   kube-proxy-db98v                              1/1     Running   0          3h52m
kube-system   kube-scheduler-teste-cluster-control-plane    1/1     Running   0          3h52m
kube-system   metrics-server-5f54fb74d9-ljlfd               1/1     Running   0          3h36m
nginx         web-dc9f554b7-9zzgv                           1/1     Running   0          57m
nginx         web-dc9f554b7-jp4c2                           1/1     Running   0          57m
```

Bom por enquanto e isso, se quiser algum outro teste estou a disposicao. Thanks =)
