# DAY-12

## Cert-Manager e Annotations.

### Instalando o Cert-Manager<a name="oquevamosaprenderhoje"></a>  

- Fonte: https://cert-manager.io/docs/installation/

```
$ kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.3/cert-manager.yaml


namespace/cert-manager created
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
serviceaccount/cert-manager-cainjector created
serviceaccount/cert-manager created
serviceaccount/cert-manager-webhook created
clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificates created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-challenges created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim created
clusterrole.rbac.authorization.k8s.io/cert-manager-cluster-view created
clusterrole.rbac.authorization.k8s.io/cert-manager-view created
clusterrole.rbac.authorization.k8s.io/cert-manager-edit created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests created
clusterrole.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificates created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-challenges created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews created
role.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection created
role.rbac.authorization.k8s.io/cert-manager:leaderelection created
role.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
rolebinding.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection created
rolebinding.rbac.authorization.k8s.io/cert-manager:leaderelection created
rolebinding.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
service/cert-manager created
service/cert-manager-webhook created
deployment.apps/cert-manager-cainjector created
deployment.apps/cert-manager created
deployment.apps/cert-manager-webhook created
mutatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created
validatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created
```  

-  Verificando os pods instalados

```
$ kubecetl get pods -n cert-manager


cert-manager-756d54fb98-n2k97              1/1     Running   0          52s
cert-manager-cainjector-7d96c69dbf-dfxnd   1/1     Running   0          52s
cert-manager-webhook-778c78f68c-mpttk      1/1     Running   0          52s
```

- Verificando tudo que foi instalado do cert-manager

```
$ kubectl get all -n cert-manager



NAME                                           READY   STATUS    RESTARTS   AGE
pod/cert-manager-756d54fb98-n2k97              1/1     Running   0          2m58s
pod/cert-manager-cainjector-7d96c69dbf-dfxnd   1/1     Running   0          2m58s
pod/cert-manager-webhook-778c78f68c-mpttk      1/1     Running   0          2m58s

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/cert-manager           ClusterIP   10.103.158.183   <none>        9402/TCP   2m58s
service/cert-manager-webhook   ClusterIP   10.98.217.35     <none>        443/TCP    2m58s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cert-manager              1/1     1            1           2m58s
deployment.apps/cert-manager-cainjector   1/1     1            1           2m58s
deployment.apps/cert-manager-webhook      1/1     1            1           2m58s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/cert-manager-756d54fb98              1         1         1       2m58s
replicaset.apps/cert-manager-cainjector-7d96c69dbf   1         1         1       2m58s
replicaset.apps/cert-manager-webhook-778c78f68c      1         1         1       2m58s
```  

## Configuração do Emissor

A primeira coisa que você precisa configurar depois de instalar o cert-manager é um **Issuer** ou um **ClusterIssuer**. Esses são recursos que representam autoridades de certificação (CAs) capazes de assinar certificados em resposta a solicitações de assinatura de certificado.

- Vamos aplicar o Issuer-staging e Issuer-Cluster

```
$ kubectl apply -f letsencrypt-issuers/staging_issuer.yaml
```

```
$ kubectl apply -f letsencrypt-issuers/production_issuer.yaml
```

- verificando

```
$ kubectl get issuers.cert-manager.io


NAME                  READY   AGE
letsencrypt-staging   True    70s
```  

```
$ kubectl get clusterissuers.cert-manager.io


NAME               READY   AGE
letsencrypt-prod   True    2m42s
```  

## Configurando o arquivo do ingress para aceitar o certificado

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: giropops-senhas
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    # cert-manager.io/cluster-issuer: letsencrypt-prod      <-- Adicionado para o letsencrypt
    cert-manager.io/issuer: letsencrypt-staging             <-- Adicionado para o letsencrypt
spec:
  ingressClassName: nginx
  tls:                                                      <-- Adicionado para o letsencrypt
  - hosts:                                                  <-- Adicionado para o letsencrypt 
    - senhas.israeldoamaral.com.br                          <-- Adicionado para o letsencrypt
    secretName: israeldoamaral.com.br-tls                   <-- Adicionado para o letsencrypt
  rules:
  - host: senhas.israeldoamaral.com.br
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: giropops-senhas
              port:
                number: 5000
```

- Vamos aplicar  

```
$ kubectl apply -f ingress.yaml
```

- Verificando o certificado

```
$ kubectl get certificaterequests.cert-manager.io
```

- Verificando o Secret  
```
$ kubectl get secret
```

## Adicionando Autenticação no Ingress

## Configurando Affinity Cookie no Ingress

## Configurando Upstream Hashing no Ingress

## Limitando requisições as nossas aplicações com o Ingress
