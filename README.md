# Kubernetes — Principais Componentes e Recursos

O Kubernetes é uma plataforma de orquestração de containers utilizada para
implantar, executar, escalar e gerenciar aplicações de forma automatizada.

A arquitetura pode ser dividida principalmente em:

- **Control Plane** → responsável por gerenciar o cluster.
- **Nodes** → máquinas onde as aplicações são executadas.
- **Objetos/Recursos Kubernetes** → definem como as aplicações devem funcionar.
- **Ferramentas de gerenciamento** → permitem interagir com o cluster.

---

## 1. Control Plane

O **Control Plane** é o cérebro do Kubernetes.

Ele recebe as configurações desejadas e coordena o cluster para que o estado
real dos recursos corresponda ao estado desejado.

### Principais componentes

| Componente | Função |
|---|---|
| **API Server** | Porta de entrada para comunicação com o Kubernetes |
| **etcd** | Banco de dados que armazena o estado do cluster |
| **Scheduler** | Decide em qual Node um Pod será executado |
| **Controller Manager** | Monitora os recursos e tenta manter o estado desejado |
| **Cloud Controller Manager** | Integra o Kubernetes com recursos de provedores de cloud |

### API Server

O **API Server** é o principal ponto de comunicação do Kubernetes.

Ferramentas como `kubectl`, aplicações e outros componentes do cluster
conversam com o Kubernetes através dele.

```text
kubectl
   │
   ▼
API Server
```
---

## 2. etcd

O **etcd** é um banco de dados distribuído utilizado pelo Kubernetes para
armazenar o estado do cluster.

Ele mantém informações como:

* Pods
* Services
* Deployments
* Nodes
* ConfigMaps
* Secrets
* Configurações do cluster

```text
Kubernetes
     │
     ▼
   etcd
     │
     └── Estado do cluster
```

---

## 3. Scheduler

O **Scheduler** decide **em qual Node um Pod deve ser executado**.

Ele analisa características como:

* CPU disponível
* Memória disponível
* Restrições
* Afinidade
* Taints e tolerations
* Regras definidas pelo administrador

```text
Pod criado
    │
    ▼
Scheduler
    │
    ├── Node 1
    ├── Node 2
    └── Node 3
```

---

## 4. Controller Manager

O **Controller Manager** executa diversos controllers responsáveis por
monitorar o estado do cluster.

A ideia principal é:

```text
Estado desejado
      │
      ▼
  Controller
      │
      ▼
Estado atual
```

Se houver uma diferença, o Kubernetes tenta corrigir automaticamente.

Por exemplo:

```text
Desejado:
3 Pods

Atual:
2 Pods

        ↓

Controller cria
mais 1 Pod
```

---

# Nodes

Um **Node** é uma máquina que executa as aplicações.

Pode ser:

* Máquina física
* Máquina virtual
* Instância em cloud

Um cluster normalmente possui vários Nodes.

```text
              Kubernetes Cluster
                     │
        ┌────────────┴────────────┐
        │                         │
     Node 1                    Node 2
        │                         │
     ┌──┴──┐                   ┌──┴──┐
     │ Pod │                   │ Pod │
     │ Pod │                   │ Pod │
     └─────┘                   └─────┘
```

---

## 5. kubelet

O **kubelet** é o agente que roda em cada Node.

Sua responsabilidade é garantir que os Pods determinados pelo Kubernetes
estejam realmente executando naquele Node.

```text
Control Plane
      │
      ▼
   kubelet
      │
      ▼
 Containers
```

---

## 6. Container Runtime

O **Container Runtime** é responsável por executar os containers.

O Kubernetes utiliza a interface **CRI (Container Runtime Interface)** para
se comunicar com o runtime.

Exemplos de runtimes:

* containerd
* CRI-O

---

# Workloads

Os recursos de **Workload** representam as aplicações que serão executadas
no cluster.

---

## 7. Pod

O **Pod** é a menor unidade de execução do Kubernetes.

Normalmente um Pod contém **um container**, mas pode conter mais de um
container quando eles precisam compartilhar recursos.

```text
Pod
┌─────────────────────┐
│                     │
│     Container       │
│                     │
└─────────────────────┘
```

Um Pod possui, entre outros:

* IP próprio
* Containers
* Volumes
* Configurações
* Recursos de CPU e memória

### Exemplo

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: minha-api
spec:
  containers:
    - name: api
      image: minha-api:1.0
```

---

## 8. ReplicaSet

O **ReplicaSet** garante que uma determinada quantidade de Pods esteja
executando.

Exemplo:

```text
replicas: 3

        ReplicaSet
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
     Pod 1 Pod 2 Pod 3
```

Se um Pod morrer:

```text
Pod 1 ❌

ReplicaSet
    │
    ▼
Cria Pod 4
```

Assim, o número desejado de réplicas é mantido.

---

## 9. Deployment

O **Deployment** é normalmente utilizado para gerenciar aplicações
stateless.

Ele utiliza ReplicaSets para controlar os Pods.

Além disso, permite realizar:

* Deploy
* Atualizações
* Rollbacks
* Escalonamento
* Atualizações graduais

A relação normalmente é:

```text
Deployment
     │
     ▼
ReplicaSet
     │
     ├── Pod
     ├── Pod
     └── Pod
```

Por exemplo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minha-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: minha-api
  template:
    metadata:
      labels:
        app: minha-api
    spec:
      containers:
        - name: api
          image: minha-api:1.0
```

---

## 10. Service

O **Service** fornece uma forma estável de acessar um conjunto de Pods.

Como Pods podem ser criados e destruídos, seus IPs podem mudar.

O Service fornece um ponto de acesso estável.

```text
             Service
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
      Pod 1   Pod 2   Pod 3
```

O Service utiliza **Labels e Selectors** para identificar os Pods que
receberão o tráfego.

---

## Tipos principais de Service

| Tipo             | Função                                         |
| ---------------- | ---------------------------------------------- |
| **ClusterIP**    | Acesso interno ao cluster                      |
| **NodePort**     | Expõe a aplicação através de uma porta do Node |
| **LoadBalancer** | Cria/integra um Load Balancer externo          |
| **ExternalName** | Cria um alias para um serviço externo          |

---

## 11. ConfigMap

O **ConfigMap** armazena configurações que não são informações sensíveis.

Por exemplo:

```text
DATABASE_HOST=postgres
DATABASE_PORT=5432
APP_ENV=production
```

O Pod pode consumir essas configurações como:

* Variáveis de ambiente
* Arquivos montados em volumes

---

## 12. Secret

O **Secret** é utilizado para armazenar informações sensíveis.

Exemplos:

* Senhas
* Tokens
* Chaves
* Credenciais

```text
Secret
   │
   ├── DB_USER
   ├── DB_PASSWORD
   └── API_TOKEN
```

> Secrets são destinados a dados sensíveis, mas não devem ser considerados
> automaticamente uma solução completa de criptografia ou gerenciamento de
> segredos.

---

## 13. Namespace

O **Namespace** permite separar recursos dentro do mesmo cluster.

Por exemplo:

```text
Cluster
│
├── namespace: desenvolvimento
│      ├── Pods
│      └── Services
│
├── namespace: homologacao
│      ├── Pods
│      └── Services
│
└── namespace: producao
       ├── Pods
       └── Services
```

É muito utilizado para organizar ambientes e equipes.

---

## 14. Ingress

O **Ingress** permite definir regras para direcionar tráfego HTTP/HTTPS
para Services.

Por exemplo:

```text
Internet
   │
   ▼
Ingress
   │
   ├── api.exemplo.com
   │        ↓
   │     API Service
   │
   └── app.exemplo.com
            ↓
        Frontend Service
```

O Ingress normalmente trabalha junto com um **Ingress Controller**.

---

## 15. PersistentVolume (PV)

Um **PersistentVolume** representa armazenamento persistente disponível
para o cluster.

Ele permite que os dados sobrevivam à criação e destruição de Pods.

```text
Pod
 │
 ▼
PVC
 │
 ▼
PV
 │
 ▼
Storage
```

---

## 16. PersistentVolumeClaim (PVC)

O **PVC** é uma solicitação de armazenamento feita por uma aplicação.

Exemplo:

```text
Aplicação
    │
    ▼
   PVC
    │
    ▼
   PV
    │
    ▼
Disco/Storage
```

O Pod utiliza o PVC sem precisar conhecer diretamente os detalhes do
armazenamento físico.

---

## 17. Labels

**Labels** são identificadores associados aos recursos Kubernetes.

Exemplo:

```yaml
labels:
  app: api
  ambiente: producao
```

Eles são muito importantes para organizar e selecionar recursos.

---

## 18. Selector

O **Selector** permite selecionar recursos através de Labels.

Por exemplo:

```text
Service
   │
   │ selector:
   │ app: api
   ▼
┌───────────────┐
│ Pod 1         │ app=api
│ Pod 2         │ app=api
│ Pod 3         │ app=api
└───────────────┘
```

Assim, o Service sabe para quais Pods deve encaminhar o tráfego.

---

## 19. kubectl

O **kubectl** é a principal ferramenta de linha de comando para interagir
com um cluster Kubernetes.

Exemplos:

```bash
kubectl get pods
```

Lista os Pods.

```bash
kubectl get nodes
```

Lista os Nodes.

```bash
kubectl get services
```

Lista os Services.

```bash
kubectl get deployments
```

Lista os Deployments.

```bash
kubectl describe pod minha-api
```

Mostra detalhes de um Pod.

```bash
kubectl logs minha-api
```

Mostra os logs de um Pod.

```bash
kubectl apply -f deployment.yaml
```

Aplica uma configuração ao cluster.

```bash
kubectl delete pod minha-api
```

Remove um Pod.

---

## 20. Manifestos YAML

Os recursos Kubernetes normalmente são declarados através de arquivos YAML.

Exemplo:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: minha-api

spec:
  replicas: 3

  selector:
    matchLabels:
      app: minha-api

  template:
    metadata:
      labels:
        app: minha-api

    spec:
      containers:
        - name: api
          image: minha-api:1.0
          ports:
            - containerPort: 8080
```

O YAML representa o **estado desejado**.

O Kubernetes então trabalha para fazer o cluster chegar a esse estado.

---

# Visão geral da arquitetura

Uma forma simplificada de visualizar tudo é:

```text
                         ┌───────────────────────┐
                         │       kubectl         │
                         │   ou API externa      │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │      API Server       │
                         └───────────┬───────────┘
                                     │
              ┌──────────────────────┼──────────────────────┐
              │                      │                      │
              ▼                      ▼                      ▼
          Scheduler              Controllers              etcd
              │                      │                      │
              └──────────────────────┼──────────────────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │        Nodes          │
                         └───────────┬───────────┘
                                     │
                     ┌───────────────┼───────────────┐
                     ▼               ▼               ▼
                  Node 1          Node 2          Node 3
                     │               │               │
                  kubelet         kubelet         kubelet
                     │               │               │
                  ┌─────┐         ┌─────┐         ┌─────┐
                  │ Pod │         │ Pod │         │ Pod │
                  └─────┘         └─────┘         └─────┘
```

---

# Fluxo de uma aplicação

Uma aplicação normalmente pode ser organizada assim:

```text
                   Usuário
                      │
                      ▼
                  Ingress
                      │
                      ▼
                   Service
                      │
            ┌─────────┼─────────┐
            ▼         ▼         ▼
          Pod 1     Pod 2     Pod 3
            │         │         │
            └─────────┼─────────┘
                      │
                  Containers
```

E o gerenciamento desses Pods ocorre através de:

```text
Deployment
     │
     ▼
ReplicaSet
     │
     ▼
   Pods
```

Enquanto o Control Plane mantém todo o cluster funcionando:

```text
                    Control Plane
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   API Server        Scheduler       Controllers
        │
        ▼
       etcd
        │
        ▼
      Nodes
        │
      kubelet
        │
      Pods
```

---

# Resumo dos principais componentes

| Componente             | Responsabilidade                          |
| ---------------------- | ----------------------------------------- |
| **Cluster**            | Conjunto completo do ambiente Kubernetes  |
| **Control Plane**      | Gerencia o cluster                        |
| **API Server**         | Porta de entrada para a API do Kubernetes |
| **etcd**               | Armazena o estado do cluster              |
| **Scheduler**          | Escolhe onde os Pods serão executados     |
| **Controller Manager** | Mantém o estado desejado                  |
| **Node**               | Máquina que executa as aplicações         |
| **kubelet**            | Agente responsável pelos Pods no Node     |
| **Container Runtime**  | Executa os containers                     |
| **Pod**                | Menor unidade de execução                 |
| **ReplicaSet**         | Mantém determinada quantidade de Pods     |
| **Deployment**         | Gerencia versões e ReplicaSets            |
| **Service**            | Fornece acesso estável aos Pods           |
| **Ingress**            | Roteia tráfego HTTP/HTTPS                 |
| **ConfigMap**          | Armazena configurações                    |
| **Secret**             | Armazena dados sensíveis                  |
| **Namespace**          | Organiza e isola recursos                 |
| **PV**                 | Representa armazenamento persistente      |
| **PVC**                | Solicita armazenamento                    |
| **Label**              | Identifica recursos                       |
| **Selector**           | Seleciona recursos através de Labels      |
| **kubectl**            | Ferramenta CLI para administrar o cluster |
| **YAML**               | Define o estado desejado dos recursos     |

```
```