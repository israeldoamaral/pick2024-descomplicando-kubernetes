# DAY-1

# Kubernetes

### Container Engine

> [!NOTE]  
Antes de começar a falar um pouco mais sobre o Kubernetes, nós primeiro precisamos entender alguns componentes que são importantes no ecossistema do Kubernetes, um desses componentes é o Container Engine.  

O **Container Engine** é o responsável por gerenciar as imagens e volumes, é ele o responsável por garantir que os os recursos que os containers estão utilizando está devidamente isolados, a vida do container, storage, rede, etc.

Hoje temos diversas opções para se utilizar como Container Engine, que até pouco tempo atrás tinhamos somente o Docker para esse papel.

Opções como o Docker, o CRI-O e o Podman são bem conhecidas e preparadas para o ambiente produtivo. O Docker, como todos sabem, é o Container Engine mais popular e ele utiliza como Container Runtime o containerd.

### Container Runtime

Para que seja possível executar os containers nos nós é necessário ter um Container Runtime instalado em cada um dos nós.

O Container Runtime é o responsável por executar os containers nos nós. Quando você está utilizando Docker ou Podman para executar containers em sua máquina, por exemplo, você está fazendo uso de algum Container Runtime, ou melhor, o seu Container Engine está fazendo uso de algum Container Runtime.  

Temos três tipos de Container Runtime:  

1. Low-level: são os Container Runtime que são executados diretamente pelo Kernel, como o runc, o crun e o runsc.
2. High-level: são os Container Runtime que são executados por um Container Engine, como o containerd, o CRI-O e o Podman.
3. Sandbox: são os Container Runtime que são executados por um Container Engine e que são responsáveis por executar containers de forma segura em unikernels ou utilizando algum proxy para fazer a comunicação com o Kernel. O gVisor é um exemplo de Container Runtime do tipo Sandbox.
4. Virtualized: são os Container Runtime que são executados por um Container Engine e que são responsáveis por executar containers de forma segura em máquinas virtuais. A performance aqui é um pouco menor do que quando temos um sendo executado nativamente. O Kata Containers é um exemplo de Container Runtime do tipo Virtualized.  

### O que é o Kubernetes?<a name="oquekubernetes"></a>

Kubernetes é uma plataforma de código aberto projetada para automatizar a implantação, escalonamento e gerenciamento de aplicativos em contêineres. Desenvolvido pelo Google e agora mantido pela Cloud Native Computing Foundation (CNCF), o Kubernetes oferece um ambiente robusto para orquestração de contêineres, permitindo que os desenvolvedores implantem, dimensionem e gerenciem aplicativos de forma eficiente e escalável em uma infraestrutura de nuvem ou local.  

Como Kubernetes é uma palavra difícil de se pronunciar - e de se escrever - a comunidade simplesmente o apelidou de k8s, seguindo o padrão i18n (a letra "k" seguida por oito letras e o "s" no final), pronunciando-se simplesmente "kates".

### Arquitetura do k8s

1. NÓS  
    - sdsdsOs nós são as máquinas físicas ou virtuais que compõem o cluster Kubernetes.
    - Cada nó executa contêineres encapsulados em pods (unidades lógicas que compartilham recursos, como armazenamento e rede).  

2. Comunicação entre Nó e Control Plane:
    - O Control Plane é o cérebro do Kubernetes, onde os principais componentes são executados.
    - Os nós se comunicam com o Control Plane para obter instruções sobre como gerenciar os contêineres.  

3. Componentes Principais do Control Plane:
    - etcd: Armazena o estado do cluster, incluindo informações sobre os nós, pods e serviços.
    - Master: É o nó onde os principais componentes do K8s são executados, como o API Server, o Controller Manager e o Scheduler.
    - API Server: Expõe a API do Kubernetes para gerenciar recursos.
    - Controller Manager: Mantém o estado desejado do sistema, como escalonamento e replicação.
    - Scheduler: Decide em qual nó um novo pod deve ser executado com base nos recursos disponíveis.  

4. Contêineres:
    - Os contêineres são unidades de implantação no Kubernetes.
    - Eles encapsulam as aplicações e suas dependências, permitindo a portabilidade e o gerenciamento eficiente.  

## Portas que devemos nos preocupar

### Control Plane

[portas_controlplane](/day-1/files/prints/portas_controlplane.png)