# DAY-6

# Volumes

### O que iremos ver hoje?<a name="oquevamosaprenderhoje"></a>

Durante o dia de hoje nós vamos aprender tudo sobre volumes, hoje é o dia de você finalmente descomplicar os volumes no Kubernetes!

Hojel nós iremos entender e configurar o que é um configmap, um persistente volume (PV) e um persistent volume claim (PVC)! E para isso iremos utilizar como exemplo diferentes tipos de clusters Kubernetes! Calma, eu explico melhor!

Para ajudar no nosso aprendizado sobre volumes, vamos utlizar diferentes clusters Kubernetes! Vamos ter exemplos utilizando EKS, kind e instâncias em cloud providers.

Então fique ciente de que hoje é o dia onde você irá descomplicar volumes no Kubernetes!


### O que são volumes?<a name="oquesaovolumes"></a>

Para simplificar o seu entendimento nesse momento, volumes nada mais são do que um diretório dentro do Pod que pode ser utilizado para armazenar dados. Eles podem ser utilizados para armazenar dados que precisam ser persistidos, como por exemplo, dados de um banco de dados, ou dados de um sistema de arquivos distribuído.

Quando estamos falando sobre volumes no Kubernetes, precisamos entender que temos basicamente dois tipos de volumes, os ephemeral volumes e os persistent volumes.

Os ephemeral volumes, que inclusive já vimos durante o treinamento o emptyDir, são volumes que são criados e destruídos junto com o Pod. Ele é um volume também, porém com uma diferença, ele não é persistente. Caso ocorra algum problema com o Pod e ele seja removido, o emptyDir também será removido.

Agora quando estamos falando sobre volumes do tipo persistent volumes, estamos falando sobre volumes que são criados e não são destruídos junto com o Pod, eles são persistidos, são volumes que seus dados são mantidos mesmo que o Pod seja removido.

Esse tipo de volume é super importante para aplicações que precisam armazenar dados que precisam ser mantidos mesmo que o Pod seja removido, como por exemplo, um banco de dados.


### Storage Class<a name="storageclass"></a>

Uma StorageClass no Kubernetes é um objeto que descreve e define diferentes classes de armazenamento disponíveis no cluster. Essas classes de armazenamento podem ser usadas para provisionar dinamicamente PersistentVolumes (PVs) de acordo com os requisitos dos PersistentVolumeClaims (PVCs) criados pelos usuários.

A StorageClass é útil para gerenciar e organizar diferentes tipos de armazenamento, como armazenamento em disco rápido e caro ou armazenamento em disco mais lento e barato. Além disso, a StorageClass pode ser usada para definir diferentes políticas de retenção, provisionamento e outras características de armazenamento específicas.

Os administradores do cluster podem criar e gerenciar várias StorageClasses para permitir que os usuários finais escolham a classe de armazenamento adequada para suas necessidades.

Cada StorageClass é definida com um provisionador, que é responsável por criar PersistentVolumes dinamicamente conforme necessário. Os provisionadores podem ser internos (fornecidos pelo próprio Kubernetes) ou externos (fornecidos por provedores de armazenamento específicos).

Inclusive os provisionadores podem ser diferentes para cada provedor de nuvem ou onde o Kubernetes está sendo executado. Vou listar alguns provisionadores que são usados e seus respectivos provedores:

- kubernetes.io/aws-ebs: **AWS Elastic Block Store (EBS)**
- kubernetes.io/azure-disk: **Azure Disk**
- kubernetes.io/gce-pd: **Google Compute Engine (GCE) Persistent Disk**
- kubernetes.io/cinder: **OpenStack Cinder**
- kubernetes.io/vsphere-volume: **vSphere**
- kubernetes.io/no-provisioner: **Volumes locais**
- kubernetes.io/host-path: **Volumes locais**

E se você estiver usando o Kubernetes em um ambiente local, como o Minikube, o provisionador padrão é o kubernetes.io/host-path, que cria volumes PersistentVolume no diretório do host. Já no Kind, o provisionador padrão é o rancher.io/local-path, que cria volumes PersistentVolume no diretório do host.

> [!NOTE] Para ver a lista completa de provisionadores, consulte a documentação do Kubernetes no link https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner.


Para você ver os Storage Classes disponíveis no seu cluster, basta executar o seguinte comando:  

```
# kubectl get storageclass



NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  21m
```


Como você pode ver, no Kind, o provisionador padrão é o ***rancher.io/local-path***, que cria volumes PersistentVolume no diretório do host.



Já no EKS, o provisionador padrão é o kubernetes.io/aws-ebs, que cria volumes PersistentVolume no EBS da AWS.

```
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  6h5m
```


Vamos ver os detalhes do nosso Storage Class padrão:
```
# kubectl describe storageclass standard



Name:            standard
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"standard"},"provisioner":"rancher.io/local-path","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           rancher.io/local-path
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
```

Uma coisa que podemos ver é que o nosso Storage Class está com a opção IsDefaultClass como Yes, o que significa que ele é o Storage Class padrão do nosso cluster, com isso todos os Persistent Volume Claims que não tiverem um Storage Class definido, irão utilizar esse Storage Class como padrão.


Vamos criar um novo Storage Class para o nosso cluster Kubernetes no kind, com o nome "local-storage", e vamos definir o provisionador como "kubernetes.io/host-path", que cria volumes PersistentVolume no diretório do host.

[storageclass.yaml](/Days/day-6/files/storageclass.yaml) 
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: giropops
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```








#### Tipos de recursos:  
Você pode usar kubectl describe com vários tipos de recursos Kubernetes, incluindo pods, serviços, deployments, replicaset, nodes, namespaces, entre outros. Por exemplo:

kubectl describe pod \<nome-do-pod\>  
kubectl describe service \<nome-do-serviço\>  
kubectl describe node \<nome-do-nó\>  

#### Detalhes Importantes:

As informações fornecidas pelo kubectl describe podem ser cruciais para diagnosticar problemas ou entender o estado atual de um recurso. Ele mostra os eventos recentes relacionados ao recurso, o que pode ajudar na resolução de problemas.

Por exemplo:

```
kubectl describe pods giropops
```

### Delete Pods<a name="deletandoumpod"></a>

Agora vamos remover o Pod que criamos, usando o comando:

```
kubectl delete pods giropops
```  


## Criano Pods através de um arquivo YAML<a name="criandopodeatravesdoyaml"></a>

para nosso estudo, vamos criar um arquivo YAML chamado pod.yaml com o seguinte conteúdo:

```
apiVersion: v1              # versão da API do Kubernetes
kind: Pod                   # tipo do objeto que estamos criando
metadata:                   # metadados do Pod 
  labels:                   # labels do Pod
    run: girus              # label run com o valor giropops
    service: webserver      # label service com o valor webserver
  name: girus               # nome do Pod que estamos criando
spec:                       # especificação do Pod
  containers:               # containers que estão dentro do Pod
  - image: nginx            # imagem do container
    name: nginx             # nome do container
    ports:                  # portas que estão sendo expostas pelo container
    - containerPort: 80     # porta 80 exposta pelo container
  dnsPolicy: ClusterFirst   # define a política de resolução de DNS para o Pod
  restartPolicy: Always     # define a política de reinicialização do container dentro do Pod
```

Agora, vamos criar o Pod usando o arquivo YAML que acabamos de criar.

```
kubectl apply -f pod.yaml
```
O comando acima irá criar o Pod usando o arquivo YAML que criamos.

> [!NOTE]
O comando **apply** é um comando que faz o que o nome diz, ele aplica o arquivo YAML no cluster, ou seja, ele cria o objeto que está descrito no arquivo YAML no cluster. Caso o objeto já exista, ele irá atualizar o objeto com as informações que estão no arquivo YAML.


Para ver o Pod criado, podemos usar o comando:

```
kubectl get pods
```

## Criando um Pod com mais de um container<a name="criandopodecommaisdeumcontainer"></a>

Vamos criar um arquivo YAML chamado pod-multi-container.yaml com o seguinte conteúdo:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: giropops
  name: giropops
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    resources: {}
  - image: alpine
    name: strigus
    args:
    - sleep
    - "1800"
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

Com o manifesto acima, estamos criando um Pod com dois containers, um container chamado girus com a imagem nginx e outro container chamado strigus com a imagem alpine. Um coisa importante de lembrar é que o container do Alpine está sendo criado com o comando sleep 1800 para que o container não pare de rodar, diferente do container do Nginx que possui um processo principal que fica sendo executado em primeiro plano, fazendo com que o container não pare de rodar.

Agora, vamos criar o Pod usando o arquivo YAML que acabamos de criar.

```
kubectl apply -f pod-multi-container.yaml
```

Para ver o Pod criado, podemos usar o comando:

```
kubectl get pods
```

Agora, vamos ver os detalhes do Pod que acabamos de criar.

```
kubectl describe pods giropops
```

## Attach e Exec<a name="attacheexec"></a>

O comando **kubectl attach** é usado no Kubernetes para conectar-se a uma sessão interativa de um contêiner em execução dentro de um pod. Ele permite interagir diretamente com o processo principal do contêiner, como se estivesse conectado diretamente a ele. Aqui está uma descrição detalhada do comando:

#### Uso Básico:
```
kubectl attach POD [-c CONTAINER] [options]
```

- POD: O nome do pod ao qual você deseja se conectar.  
- -c CONTAINER: Opcional. O nome do contêiner dentro do pod ao qual você deseja se conectar. Se o pod tiver apenas um contêiner, esta opção não é necessária.

#### Funcionamento

1. Conexão Interativa: O kubectl attach estabelece uma conexão direta com o processo principal (PID 1) do contêiner especificado no pod.

2. Entrada e Saída: Depois de conectar-se, você pode enviar entradas para o processo do contêiner e ver sua saída diretamente no seu terminal.

3. Terminal Ativo: A interação ocorre em tempo real. Você pode enviar comandos e receber saída do processo, útil para depurar ou interagir com aplicativos em execução.

#### Exemplo de uso

```
kubectl attach giropops -c strigus
```
> [!NOTE]
Usando o attach é como se estivéssemos conectando diretamente em uma console de uma máquina virtual, não estamos criando nenhum processo dentro do container, apenas nos conectando a ele.  
Por esse motivo se tentarmos utilizar o attach para conectar no container que está rodando o Nginx, nós iremos conectar ao container e ficaremos presos ao processo do Nginx que está em execução em primeiro plano, e não conseguiremos executar nenhum outro comando.

Para sair do container, basta apertar a tecla Ctrl + C.


> [!NOTE]Entendeu? Só vamos usar o attach para se conectar a um container que está rodando dentro de um Pod, e não para executar comandos dentro do container.

Agora, se você está afim de executar comandos dentro do container, você pode usar o comando exec.

O comando kubectl exec é usado no Kubernetes para executar comandos dentro de um contêiner em execução dentro de um pod. Ele permite interagir com um contêiner sem a necessidade de uma conexão interativa. Aqui está uma descrição detalhada do comando:

#### Uso Básico

```
kubectl exec POD [-c CONTAINER] -- COMMAND [args...]
```

- POD: O nome do pod onde o contêiner está em execução.
- -c CONTAINER: Opcional. O nome do contêiner dentro do pod onde você deseja executar o comando. Se o pod tiver apenas um contêiner, esta opção não é necessária.
- -- COMMAND [args...]: O comando que você deseja executar dentro do contêiner.


#### Funcionamento

1. Execução de Comandos: O kubectl exec permite que você execute comandos arbitrários dentro de um contêiner em execução no pod especificado.

2. Comunicação com o Contêiner: Você pode enviar comandos diretamente para o contêiner e receber a saída de volta no seu terminal.

3. Utilidade: Este comando é amplamente utilizado para fins de depuração, inspeção e execução de tarefas administrativas dentro de um contêiner.

#### Exemplos de Uso:

```
kubectl exec giropops -c strigus -- ls
```

Nós também podemos utilizar o exec para conectar em uma container que está rodando dentro de um Pod, porém, para isso, precisamos passar o parâmetro -it para o comando exec.

```
kubectl exec giropops -c strigus -it -- sh
```

O parametro -it é usado para que o comando exec crie um processo dentro do container com interatividade e com um terminal, fazendo com que o comando exec se comporte como o comando attach porém, com a diferença que o comando exec cria um processo dentro do container, no caso o processo sh. 

Nesse caso, podemos até mesmo conectar no container do Nginx, pois ele vai conectar no container criando um processo que é o nosso interpretador de comandos sh, sendo possível executar qualquer comando dentro do container pois temos um shell para interagir com o container.

```
kubectl exec giropops -c girus -it -- sh
```

Para sair do container, basta apertar a tecla Ctrl + D.


## Criando um container com limites de memória e CPU<a name="criandoumcontainercomlimitesdememoriaecpu"></a>

Vamos criar um arquivo YAML chamado pod-limitado.yaml com o seguinte conteúdo:

```
apiVersion: v1              # versão da API do Kubernetes
kind: Pod                   # tipo do objeto que estamos 
metadata:                   # metadados do Pod
  labels:                   # labels do Pod
    run: giropops           # label run com o valor giropops
  name: giropops            # nome do Pod que estamos criando
spec:                       # especificação do Pod
  containers:               # containers que estão dentro do Pod
  - image: nginx            # imagem do container
    name: girus             # nome do container
    ports:                  # portas que estão sendo expostas pelo container
    - containerPort: 80     # porta 80 exposta pelo container
    resources:              # recursos que estão sendo utilizados pelo container
      limits:               # limites máximo de recursos que o container pode utilizar
        cpu: "0.5"          # limite máxima de CPU que o container pode utilizar, (50% de uma CPU)
        memory: "128Mi"     # limite de memória que está sendo utilizado pelo container (128 megabytes)
      requests:             # recursos garantidos ao container
        cpu: "0.3"          # CPU garantida ao container, no caso 30% de uma CPU
        memory: "64Mi"      # memória garantida ao container, no caso 64 megabytes
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

O campo resources é usado para definir os recursos que estão sendo utilizados pelo container, e dentro dele temos os campos limits e requests.

O campo limits é usado para definir os limites máximos de recursos que o container pode utilizar, e o campo requests é usado para definir os recursos garantidos ao container.

Agora vamos criar o Pod com os limites de memória e CPU.

```
kubectl create -f pod-limitado.yaml
```

Agora vamos verificar se o Pod foi criado.

```
kubectl get pods
```

Vamos verificar os detalhes do Pod.

```
kubectl describe pod giropops
```

Veja que o Pod foi criado com sucesso, e que os limites de memória e CPU foram definidos conforme o arquivo YAML.

> [!NOTE]
Veja a parte da saída do comando describe que mostra os limites de memória e CPU.


## Adicionando um volume EmptyDir no Pod<a name="adicionandovolumeemptydir"></a>

Um volume do tipo EmptyDir é um volume que é criado no momento em que o Pod é criado, e ele é destruído quando o Pod é destruído. Ou seja, ele é um volume temporário.

No dia-a-dia, nós não vamos usar muito esse tipo de volume, mas é importante que saibamos que ele existe. Um dos casos de uso mais comuns é quando você precisa compartilhar dados entre os containers de um Pod. Imagina que você tem dois containers em um Pod e um deles possui um diretório com dados, e você quer que o outro container tenha acesso a esses dados. Nesse caso, você pode criar um volume do tipo EmptyDir e compartilhar esse volume entre os dois containers.

Vamos criar um arquivo YAML chamado pod-emptydir.yaml com o swguinte conteúdo.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: giropops
  name: giropops
spec:
  containers:
  - image: nginx
    name: webserver
    volumeMounts:                   # lista de volumes que serão montados no container
    - mountPath: /giropops          # diretório onde o volume será montado
      name: primeiro-emptydir       # nome do volume
    resources:
      limits:
        cpu: "1"
        memory: "128Mi"
      requests:
        cpu: "0.5"
        memory: "64Mi"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:                          # lista de volumes
  - name: primeiro-emptydir         # nome do volume
    emptyDir:                       # tipo do volume
      sizeLimit: 256Mi              # tamanho máximo do volume
```

Agora vamos criar o Pod.

```
kubectl create -f pod-emptydir.yaml
```

Agora vamos verificar se o Pod foi criado.

```
kubectl get pods
```

Você pode ver a saída do comando kubectl describe pod giropops para ver o volume que foi criado.

```
kubectl describe pod giropops
```

Agora vamos entrar do container.

```
kubectl exec -it ubuntu -- bash
```

Agora vamos criar um arquivo dentro do diretório /giropops.

```
touch /giropops/FUNCIONA_MESMO
```

Pronto, o nosso arquivo foi criado dentro do diretório /giropops, que é um diretório dentro do volume do tipo EmptyDir.


Se você digitar mount, vai ver que o diretório /giropops está montado certinho dentro de nosso container.

```
mount
```

Por hoje é só!!!!



