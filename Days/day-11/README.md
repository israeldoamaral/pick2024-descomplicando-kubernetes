# DAY-11

## Ingress Crontoller.

### O que iremos ver hoje?<a name="oquevamosaprenderhoje"></a>

Se você está aqui, provavelmente já tem alguma noção do que o Kubernetes faz. Mas como expor seus serviços ao mundo externo de forma eficiente e segura? É aí que entra o nosso protagonista do dia: o Ingress. Nesta seção, vamos desvendar o que é o Ingress, para que serve e como ele se diferencia de outras formas de expor aplicações no Kubernetes.


### O que é o Ingress?  

O Ingress é um recurso do Kubernetes que gerencia o acesso externo aos serviços dentro de um cluster. Ele funciona como uma camada de roteamento HTTP/HTTPS, permitindo a definição de regras para direcionar o tráfego externo para diferentes serviços back-end. O Ingress é implementado através de um controlador de Ingress, que pode ser alimentado por várias soluções, como NGINX, Traefik ou Istio, para citar alguns.

Tecnicamente, o Ingress atua como uma abstração de regras de roteamento de alto nível que são interpretadas e aplicadas pelo controlador de Ingress. Ele permite recursos avançados como balanceamento de carga, SSL/TLS, redirecionamento, reescrita de URL, entre outros.  

### Principais Componentes e Funcionalidades:  

- Controlador de Ingress: É a implementação real que satisfaz um recurso Ingress. Ele pode ser implementado através de várias soluções de proxy reverso, como NGINX ou HAProxy.

- Regras de Roteamento: Definidas em um objeto YAML, essas regras determinam como as requisições externas devem ser encaminhadas aos serviços internos.

- Backend Padrão: Um serviço de fallback para onde as requisições são encaminhadas se nenhuma regra de roteamento for correspondida.

- Balanceamento de Carga: Distribuição automática de tráfego entre múltiplos pods de um serviço.

- Terminação SSL/TLS: O Ingress permite a configuração de certificados SSL/TLS para a terminação de criptografia no ponto de entrada do cluster.

- Anexos de Recurso: Possibilidade de anexar recursos adicionais como ConfigMaps ou Secrets, que podem ser utilizados para configurar comportamentos adicionais como autenticação básica, listas de controle de acesso etc.  

## Configurando o kind para suportar o Ingress  

Kind é uma ferramenta muito útil para testes e desenvolvimento com Kubernetes. Nesta seção atualizada, fornecemos detalhes específicos para garantir que o Ingress funcione como esperado em um cluster Kind.

Criando o Cluster com Configurações Especiais
Ao criar um cluster KinD, podemos especificar várias configurações que incluem mapeamentos de portas e rótulos para nós.

1. Crie um arquivo chamado kind-config.yaml com o conteúdo abaixo:  

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
- role: worker
```

2. Em seguida, crie o cluster usando este arquivo de configuração:  

```
kind create cluster --config kind-config.yaml
```