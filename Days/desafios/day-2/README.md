# Desafio Day-2

## Descomplicando Kubernetes


### Corrija todos os erros encontrados e realize o deploy do Nginx utilizando o arquivo pod.yaml.

#### YAML com erros
```
apiVersion: v1beta1
kind: pods
metadata:
  labels:
    run: nginx-giropops
    app: giropops-strigus
  name: nginx_giropops
spec:
  containers:
  - image: nginx
    name: nginx_giropops
    ports:
    - containerPort: 80
    resources:
      limits:
        memory:
        cpu: "0.5"
    requests:
        memory: "4400MB"
        cpu: "0,3"
  dnsPolicy: ClusterSecond
  restartPolicy: Always
```

##### YAML corrigido

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-giropops
    app: giropops-strigus
  name: nginx-giropops
spec:
  containers:
  - image: nginx
    name: nginx-giropops
    ports:
    - containerPort: 80
    resources:
      limits:
        memory: "128Mi"
        cpu: "0.5"
      requests:
        memory: "44Mi"
        cpu: "0.3"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

#### Executando o Pod
```
kubectl apply -f pod.yaml
```