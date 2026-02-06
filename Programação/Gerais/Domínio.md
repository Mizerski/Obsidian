## 1. Definição 

Domínio é o **núcleo do problema que o sistema resolve**. É onde ficam as **regras de negócio que fazem sentido independentemente de HTTP, banco, framework ou tecnologia**.

Uma forma útil de pensar:

- **Controller**: orquestra entrada/saída (quem chama, formato, protocolo)
- **Domínio**: decide _o que pode_ e _o que não pode_ acontecer
- **Infraestrutura**: decide _como_ isso é persistido, enviado ou integrado

Quando a regra depende de _contexto técnico_ (request, status HTTP, JPA, transação), ela **não é domínio**.

---

## 2. Erro comum e por que ele acontece

> "Regra de negócio fica na controller"

Isso acontece porque, em projetos simples, a controller vira o único ponto visível de execução.

O problema:

- Controller conhece HTTP → domínio passa a conhecer HTTP indiretamente
- Regra fica acoplada a framework
- Reuso e teste ficam caros

Boa prática: **controller só delega**. Se ela decide algo relevante, esse código provavelmente deveria estar no domínio.

---

## 3. O que são regras de negócio no domínio

Regras de negócio são **invariantes e comportamentos do modelo**, por exemplo:
- Um cliente não pode ser criado sem CPF válido
- Um pedido não pode ser pago duas vezes
- Um usuário só pode alterar dados próprios

Note que nenhuma dessas regras fala _como salvar_, _qual endpoint_ ou _qual banco_.

Comparação útil:

- `if (dto.getIdade() < 18) return 400` → controller-centric
- `cliente.validarMaioridade()` → domínio

---

## 4. O que normalmente pertence ao domínio

Depende do nível de maturidade do projeto, mas em geral:

### 4.1 Model / Entidade de Domínio

Objeto que **representa algo real do negócio** e **se protege sozinho**.

Características:
- Contém estado + comportamento
- Garante suas próprias regras
- Não depende de framework

Exemplo conceitual:
- Cliente
- Pedido
- Conta

> Entidade de domínio ≠ entidade JPA (podem coincidir, mas não são a mesma coisa conceitualmente)

---

### 4.2 Value Objects

Objetos imutáveis que representam conceitos do domínio:

- CPF
- Email
- Dinheiro

Por quê usar:

- Centraliza validação
- Evita `String` sem significado
- Aumenta legibilidade e segurança

---

### 4.3 Serviços de Domínio

Usados quando:

- A regra não pertence claramente a uma única entidade
- A lógica envolve múltiplos agregados

Exemplo:

- Cálculo de preço
- Regras de aprovação

Importante:

- Serviço de domínio **não é service do Spring**
- Ele expressa regra, não orquestra infraestrutura

---

### 4.4 Interfaces de Repositório (contratos)

O domínio **define o que precisa**, não como é feito.

Exemplo:

- `ClienteRepository`

O domínio sabe que precisa buscar ou salvar, mas **não sabe se é JPA, JDBC ou API externa**.

Implementação concreta fica na infraestrutura.

---

## 5. O que NÃO é domínio (na maioria dos casos)

### 5.1 Controllers

- Lidam com HTTP
- Conhecem request/response
- Traduzem exceções em status

Nunca deveriam conter regras essenciais do negócio.

---

### 5.2 DTOs de Request/Response

DTOs são **modelos de transporte**, não de negócio.

Eles:

- Existem por causa da API
- Mudam conforme contrato externo
- Não representam conceitos reais do domínio

Boa prática:

- DTO → Mapper → Domínio
- Domínio → Mapper → DTO

Em projetos simples, podem ficar próximos, mas **conceitualmente não são domínio**.

---

### 5.3 Anotações e dependências técnicas

- `@Entity`
- `@Table`
- `@JsonProperty`

Se o domínio depende disso, ele deixa de ser puro.

---

