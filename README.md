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