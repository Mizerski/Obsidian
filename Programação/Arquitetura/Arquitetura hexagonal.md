## Sobre

A **Arquitetura Hexagonal** (também conhecida como **Ports and Adapters**) é um **estilo arquitetural**, não um framework nem uma simples organização de pastas.

Seu objetivo central é **controlar o fluxo de dependências**, garantindo que o **core da aplicação** não dependa de detalhes externos como frameworks, banco de dados, protocolos ou mecanismos de entrega.

O nome “hexagonal” vem da ideia de que o sistema pode ser acessado por **múltiplos lados (portas)**, sem que o núcleo precise saber _quem_ está chamando ou _como_ os dados entram ou saem.

## Princípio Fundamental
> **Dependências sempre apontam para dentro.**

- O core **não conhece** HTTP, JPA, Spring, JSON ou SQL
- Detalhes externos **conhecem e dependem** do core
- Integrações são feitas por **contratos (ports)**

Isso é o que diferencia Hexagonal de uma arquitetura apenas “em camadas”.
## Estrutura Conceitual
Uma representação comum em projetos Java/Spring:
```
clientes/
├── api/                    ← Adapters primários (entrada)
│   ├── controller/         ← Controllers REST/Web
│   ├── request/            ← DTOs de entrada
│   └── response/           ← DTOs de saída
│
├── application/            ← Casos de uso (orquestração)
│   └── usecase/
│
├── domain/                 ← Core da aplicação
│   ├── model/              ← Entidades e Value Objects
│   └── repository/         ← Portas (interfaces)
│
└── infrastructure/         ← Adapters secundários (saída)
    └── persistence/        ← DB, JPA, APIs externas

```
> **Importante:**  
> Essa estrutura é apenas uma _convenção_.  
> O que define a arquitetura é o **sentido das dependências**, não o nome das pastas.

## Componentes da Arquitetura
### 1. Core (Centro do Hexágono)

O core representa a **lógica da aplicação**, isolada de qualquer tecnologia externa.

Ele é composto por:

- [[Estudos/Programação/Gerais/Domínio]]
- Casos de uso (application layer)
- Portas (interfaces)

O core:

- Não conhece Spring
- Não conhece HTTP
- Não conhece JPA
- Não sabe como os dados entram ou saem
### 2. Portas (Ports)

Portas são **interfaces que o core define** para se comunicar com o mundo externo.

Existem dois tipos principais:

#### 2.1 Portas de Entrada (Inbound Ports)

- Representam **o que a aplicação pode fazer**
- Normalmente mapeadas como casos de uso
- Exemplo: `CriarClienteUseCase`

O controller chama uma porta de entrada, não o domínio diretamente.

#### 2.2 Portas de Saída (Outbound Ports)

- Representam **o que o core precisa do mundo externo**
- Geralmente interfaces de repositório ou gateways

Exemplo:
```java
ClienteRepository
```
O core sabe _o que_ precisa, mas não _como_ será feito.

### 3. Adapters Primários (Entrada)

Adapters primários são responsáveis por:

- Receber chamadas externas
- Traduzir formato/protocolo
- Acionar portas de entrada

Exemplos:

- Controllers REST
- Consumers de fila
- Interfaces CLI

Eles:

- Conhecem o core
- Dependem das portas de entrada
- Não contêm regra de negócio
### 4. Adapters Secundários (Saída)

Adapters secundários implementam as portas de saída.

Exemplos:

- Repositórios JPA
- Integração com APIs externas
- Mensageria

Eles:

- Dependem das interfaces definidas no core
- Contêm detalhes técnicos
- Podem ser trocados sem afetar o domínio

## Regra de Validação Mental
> Se eu remover o framework, o core continua válido?

Se a resposta for **sim**, a Arquitetura Hexagonal está bem aplicada.  
Se não, o core já foi contaminado por detalhes externos.