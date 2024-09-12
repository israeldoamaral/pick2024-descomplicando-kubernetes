# DAY-13

# Horizontal Pod Autoscaler.

### O que iremos ver hoje?<a name="oquevamosaprenderhoje"></a>  

Hoje é um dia particularmente fascinante! Vamos desbravar os territórios do Kubernetes, explorando a magia do Horizontal Pod Autoscaler (HPA), uma ferramenta indispensável para quem almeja uma operação eficiente e resiliente. Portanto, afivelem os cintos e preparem-se para uma jornada de descobertas. 

### Introdução ao Horizontal Pod Autoscaler (HPA)  

O Horizontal Pod Autoscaler, carinhosamente conhecido como HPA, é uma das joias brilhantes incrustadas no coração do Kubernetes. Com o HPA, podemos ajustar automaticamente o número de réplicas de um conjunto de pods, assegurando que nosso aplicativo tenha sempre os recursos necessários para performar eficientemente, sem desperdiçar recursos. O HPA é como um maestro que, com a batuta das métricas, rege a orquestra de pods, assegurando que a harmonia seja mantida mesmo quando a sinfonia do tráfego de rede atinge seu crescendo.  

### Como o HPA Funciona?  

O HPA é o olheiro vigilante que monitora as métricas dos nossos pods. A cada batida do seu coração métrico, que ocorre em intervalos regulares, ele avalia se os pods estão suando a camisa para atender às demandas ou se estão relaxando mais do que deveriam. Com base nessa avaliação, ele toma a decisão sábia de convocar mais soldados para o campo de batalha ou de dispensar alguns para um merecido descanso.

Certamente! O Metrics Server é uma componente crucial para o funcionamento do Horizontal Pod Autoscaler (HPA), pois fornece as métricas necessárias para que o HPA tome decisões de escalonamento. Vamos entender um pouco mais sobre o Metrics Server e como instalá-lo em diferentes ambientes Kubernetes, incluindo Minikube e KinD.  

# Introdução ao Metrics Server  

Antes de começarmos a explorar o Horizontal Pod Autoscaler (HPA), é essencial termos o Metrics Server instalado em nosso cluster Kubernetes. O Metrics Server é um agregador de métricas de recursos de sistema, que coleta métricas como uso de CPU e memória dos nós e pods no cluster. Essas métricas são vitais para o funcionamento do HPA, pois são usadas para determinar quando e como escalar os recursos.  

## Por que o Metrics Server é importante para o HPA?  

O HPA utiliza métricas de uso de recursos para tomar decisões inteligentes sobre o escalonamento dos pods. Por exemplo, se a utilização da CPU de um pod exceder um determinado limite, o HPA pode decidir aumentar o número de réplicas desse pod. Da mesma forma, se a utilização da CPU for muito baixa, o HPA pode decidir reduzir o número de réplicas. Para fazer isso de forma eficaz, o HPA precisa ter acesso a métricas precisas e atualizadas, que são fornecidas pelo Metrics Server. Portanto, precisamos antes conhecer essa peça fundamental para o dia de hoje! :D  

## Instalando o Metrics Server  

Durante meus estudos, estou com um cluster kubeadm, e para instalar o Metrics Server, podemos usar o seguinte comando:

fonte: https://github.com/kubernetes-sigs/metrics-server

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```  

Esse comando aplica o manifesto do Metrics Server ao seu cluster, instalando todos os componentes necessários.

```
kubectl get pods -n kube-system


NAME                                        READY   STATUS    RESTARTS       AGE
metrics-server-69678d4786-js6fk             1/1     Running   0              3m50s
```

OBS: Caso não fique na coluna READY 1/1 execute o logs para verificar o problema.  

```
kubectl logs -f metrics-server-69678d4786-js6fk -n kube-system
```

Se no log você vericar que não subiu devido ao problema de certificado TLS execute os passos abaixo:

fonte: https://datacenterdope.wordpress.com/2020/01/20/installing-kubernetes-metrics-server-with-kubeadm/

1. Baixe o arqui .yaml do metrics.

```
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```   
2. Edite o arquivo yaml e adicione as linha abaixo em .containers.args  
```
- --kubelet-preferred-address-types=InternalDNS,InternalIP,ExternalDNS,ExternalIP,Hostname
- --kubelet-insecure-tls
```  
3. Salve e aplique novamente.  
```
kubectl apply -f components.yaml
```



#### No Minikube:

```
minikube addons enable metrics-server
```  

Após a execução deste comando, o Metrics Server será instalado e ativado em seu cluster Minikube.  


#### No KinD (Kubernetes in Docker):  

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```  

## Verificando a Instalação do Metrics Server  

Após a instalação do Metrics Server, é uma boa prática verificar se ele foi instalado corretamente e está funcionando como esperado. Execute o seguinte comando para obter a lista de pods no namespace kube-system e verificar se o pod do Metrics Server está em execução:  

```
kubectl get pods -n kube-system | grep metrics-server
```  

## Obtendo Métricas  

Com o Metrics Server em execução, agora você pode começar a coletar métricas de seu cluster. Aqui está um exemplo de como você pode obter métricas de uso de CPU e memória para todos os seus nodes:  

```
kubectl top nodes
```  

E para obter métricas de uso de CPU e memória para todos os seus pods:  

```
kubectl top pods
```  

Esses comandos fornecem uma visão rápida da utilização de recursos em seu cluster, o que é crucial para entender e otimizar o desempenho de seus aplicativos.  

## Criando um HPA  

Antes de nos aprofundarmos no HPA, vamos recapitular criando um deployment simples para o nosso confiável servidor Nginx.  

nginx-deployment-hpa.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m        # Limite de CPU
            memory: 256Mi    # Limite de memória
          requests:
            cpu: 250m        # Requisição de CPU
            memory: 128Mi    # Requisição de memória
```  

```
kubectl apply -f nginx-deployment-hpa.yaml
```

Agora, com nosso deployment pronto, vamos dar o próximo passo na criação do nosso HPA.

nginx-deployment-hpa.yaml

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-deployment-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```  

Neste exemplo, criamos um HPA que monitora a utilização da CPU do nosso nginx-deployment. O HPA se esforçará para manter a utilização da CPU em torno de 50%, ajustando o número de réplicas entre 3 e 10 conforme necessário.  

Para aplicar esta configuração ao seu cluster Kubernetes, salve o conteúdo acima em um arquivo chamado  

```
kubectl apply -f nginx-deployment-hpa.yaml
```  

Agora, você tem um HPA monitorando e ajustando a escala do seu nginx-deployment baseado na utilização da CPU. Fantástico, não é?  

## Exemplos Práticos com HPA  

Agora que você já entende o básico sobre o HPA, é hora de rolar as mangas e entrar na prática. Vamos explorar como o HPA responde a diferentes métricas e cenários.  

#### Autoscaling com base na utilização de CPU  

Vamos começar com um exemplo clássico de escalonamento baseado na utilização da CPU, que já discutimos anteriormente. Para tornar a aprendizagem mais interativa, vamos simular um aumento de tráfego e observar como o HPA responde a essa mudança.  

```
kubectl run -i --tty load-generator --image=busybox /bin/sh


while true; do wget -q -O- http://nginx-deployment.default.svc.cluster.local; done

```  
Este script simples cria uma carga constante no nosso deployment, fazendo requisições contínuas ao servidor Nginx. Você poderá observar como o HPA ajusta o número de réplicas para manter a utilização da CPU em torno do limite definido.  

#### Autoscaling com base na utilização de Memória  

O HPA não é apenas um mestre em lidar com a CPU, ele também tem um olho afiado para a memória. Vamos explorar como configurar o HPA para escalar baseado na utilização de memória.  

nginx-deployment-hpa-memory.yaml

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-deployment-hpa-memory
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
```  

Neste exemplo, o HPA vai ajustar o número de réplicas para manter a utilização de memória em cerca de 70%. Assim, nosso deployment pode respirar livremente mesmo quando a demanda aumenta.

