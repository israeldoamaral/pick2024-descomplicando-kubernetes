# Desafio Day-1

## Descomplicando Kubernetes


### Instalar o kind e criar o nosso cluster kubernetes

    - É hora de instalar o incrível Kind, para que possamos ter o nosso primeiro cluster Kubernetes criado com sucesso!

    - Para a criação do seu cluster Kubernetes, nós vamos utilizar o Kind! O nosso cluster será composto por:  
      - 1 node Control-Plane
      - 3 nodes Workers.  


    - Você deve criar um arquivo chamado meu-primeiro-cluster.yaml no diretório /root com todas as definições necessárias para o Kind criar o seu cluster.  

### Deploy do nosso Primeiro Pod

    - Temos um arquivo já criado esperando apenas você realizar o deploy desse nosso primeiro pod!

    - O manifesto com todas as definições para a criação desse pod está em meu-primeiro-pod.yaml.

    - Lembre-se, se houver algum erro no momento do deploy, você precisa ajustar o que estiver errado e tentar novamente!

Precisamos ter esse pod em execução!

**meu-primeiro-pod.yaml**
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

## Passos

#### Passo 1:

Vamos instalar o **kubectl**
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version --client
```  

#### Passo 2

Instalar o kind
```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.14.0/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind
```  

#### Passo 3

Criando o arquivo meu-primeiro-cluster.yaml
```
cat << EOF > meu-primeiro-cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
EOF
```  

#### Passo 4

Criando o cluster
```
kind create cluster --config meu-primeiro-cluster.yaml --name meu-primeiro-cluster
```

#### Passo 5  

Correção do manifesto "meu-primeiro-pod.yaml"

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
        memory: "4800Mi"
        cpu: "0.5"
      requests:
        memory: "4400Mi"
        cpu: "0.3"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  ```  

##### Passo 5

Executando o meu-primeiro-pod.yaml

```
kubectl apply -f meu-primeiro-pos.yaml
```

[Voltar para o início do repositório](https://github.com/israeldoamaral/pick2024-descomplicando-kubernetes)