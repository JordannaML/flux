Bom, vamos lá...

Eu AMMMEI  a ferramenta, nao conhecia e nao fazia ideia do quanto ela e legal. Vou usar com certeza no meu dia a dia.


- Primeiro Ponto
 criei a app do nginx via 

❯ kubectl apply -k .

 e tambem criei um kustomization.yaml dentro da mesma pasta, anteriormente eu usei um spec de label que estava defazado, e acabeio recebendo um warning por conta disso, mas pesquizando na internet ddescobri um comando que faz uma correcao do arquivo de forma altomatica, o comando foi

❯ kustomize edit fix    
 Warning: 'commonLabels' is deprecated. Please use 'labels' instead. Run 'kustomize edit fix' to update your Kustomization automatically.

Fixed fields:
  patchesJson6902 -> patches
  patchesStrategicMerge -> patches
  commonLabels -> labels

Eu AMEI

- Segundo ponto, fiquei em duvida sobre o arquivo podinfo.yaml, vai dar para ver que mudei ele de lugar algumas vezes pq quando procurava por algum kustomization ele nao me voltava um lugar, mas me falava que existia

❯ kubectl get kustomization podinfo            
NAME      AGE   READY   STATUS
podinfo   81m   True    Applied revision: main@sha1:ed283a5e05a4f2921e2ce67ca1b171a608cb1cfe

 ~/teste-cluster/flux/apps/kustomize │ on main ▓▒░                                ░▒▓ at kind-teste-cluster ⎈ │ at 20:31:30 
❯ kubectl get kustomization podinfo -A
error: a resource cannot be retrieved by name across all namespaces

e nesse momento eu vi a magica acontecendo de fato, alterei o kustomization.yaml no path do flux-system, adicionando o podinfo.yaml nele, e coloquei o yaml dentro desse mesmo path (flux-system), tambem vi que esse kind deve ficar dentro desse mesmo fluxo, fiz o commit apenas e assim que subiu as alteracoes vi dentro do cluster ele sendo criado dentro do fluxo que defini no yaml


❯ kubectl get kustomizations -n flux-system
NAME          AGE    READY   STATUS
flux-system   106m   True    Applied revision: main@sha1:7788a72bc0ddb1aef210208f3d14f0ce96a86e11
podinfo       2s     True    Applied revision: main@sha1:7788a72bc0ddb1aef210208f3d14f0ce96a86e11

Outra alteracao que fiz foi vendo essa doc aqui sobre estrutura de pasta 
https://fluxcd.io/flux/guides/repository-structure/
E com ela eu alterei tambem o meu path de onde o kustomize iria ficar

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

  Bom por enquanto e isso, se quiser algum outro teste estou a disposicao. Thanks =)

