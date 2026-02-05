
## 0. Conceito Fundamental: Inversão de Dependência

Na [[Arquitetura Hexagonal]] (ou Ports and Adapters), o **[[Domínio]] é o centro**.

- **Regra de Ouro:** O núcleo (Domain) não conhece detalhes de infraestrutura (Banco de Dados, Frameworks, APIs externas).
- **Fluxo de Dependência:** A Infraestrutura e a API dependem do Domínio. O Domínio depende apenas de interfaces (Portas).

---

## Estrutura de Pastas Respeitada (Que uso normalmente)

```
clientes
├── api (Adapter de Entrada)
│   ├── controller
│   ├── request
│   └── response
├── domain (Núcleo/Core)
│   ├── model
│   └── repository (Porta de Saída)
└── infrastructure (Adapter de Saída)
    └── implementation
        └── persistence
            └── entity
```

---

## 1. Camada de [[Domínio]] (Core)

### Model: O Objeto de Negócio

`domain/model/Cliente.java` O domínio contém a lógica e as validações. Ele deve ser POJO (Plain Old Java Object).

```java
package clientes.domain.model;

import lombok.Getter;

@Getter
public class Cliente {
    private final String nome;
    private final String email;

    public Cliente(String nome, String email) {
        if (nome == null || nome.isBlank()) {
            throw new IllegalArgumentException("Nome é obrigatório");
        }
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Email inválido");
        }
        this.nome = nome;
        this.email = email;
    }
}
```

### Repository: A Porta de Saída (Port)

`domain/repository/ClienteRepository.java` Uma interface que define **o que** o domínio precisa, sem dizer **como** será feito.


```java
package clientes.domain.repository;

import clientes.domain.model.Cliente;

public interface ClienteRepository {
    void salvar(Cliente cliente);
}
```

### Use Case: O Orquestrador

`domain/model/CriarClienteUseCase.java` Embora esteja na pasta `model` por sua estrutura, ele atua como a **Camada de Aplicação**. Ele coordena a execução.


```java
package clientes.domain.model;

import clientes.domain.repository.ClienteRepository;

public class CriarClienteUseCase {
    private final ClienteRepository repository;

    public CriarClienteUseCase(ClienteRepository repository) {
        this.repository = repository;
    }

    public void executar(String nome, String email) {
        // 1. Cria o objeto de domínio (validações ocorrem no construtor)
        Cliente cliente = new Cliente(nome, email);
        
        // 2. Persiste através da interface (porta)
        repository.salvar(cliente);
    }
}
```

---

## 2. Camada de Infraestrutura (Persistence Adapter)

### Entity: O Detalhe do Banco de Dados

`infrastructure/implementation/persistence/entity/ClienteEntity.java` Esta classe pertence ao Hibernate/JPA, não ao domínio.


```java
package clientes.infrastructure.implementation.persistence.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Entity
@Table(name = "clientes")
@Getter
@NoArgsConstructor
public class ClienteEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String nome;
    private String email;

    public ClienteEntity(String nome, String email) {
        this.nome = nome;
        this.email = email;
    }
}
```

### Implementation: O Adapter de Saída

`infrastructure/implementation/persistence/ClienteRepositoryImpl.java` Aqui o mundo externo (JPA) se adapta à necessidade do domínio.


```java
package clientes.infrastructure.implementation.persistence;

import clientes.domain.model.Cliente;
import clientes.domain.repository.ClienteRepository;
import clientes.infrastructure.implementation.persistence.entity.ClienteEntity;
import org.springframework.stereotype.Repository;
import jakarta.persistence.EntityManager;

@Repository
public class ClienteRepositoryImpl implements ClienteRepository {
    private final EntityManager em;

    public ClienteRepositoryImpl(EntityManager em) {
        this.em = em;
    }

    @Override
    public void salvar(Cliente cliente) {
        // Converte o Objeto de Domínio para a Entidade do Banco
        ClienteEntity entity = new ClienteEntity(cliente.getNome(), cliente.getEmail());
        em.persist(entity);
    }
}
```

---

## 3. Camada de API (Entry Adapter)

### Request: DTO de Entrada

`api/request/CriarClienteRequest.java`

```java
package clientes.api.request;

import lombok.Getter;

@Getter
public class CriarClienteRequest {
    private String nome;
    private String email;
}
```

### Controller: O Adapter de Entrada

`api/controller/ClienteController.java`

```java
package clientes.api.controller;

import clientes.api.request.CriarClienteRequest;
import clientes.domain.model.CriarClienteUseCase;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/clientes")
public class ClienteController {
    private final CriarClienteUseCase useCase;

    public ClienteController(CriarClienteUseCase useCase) {
        this.useCase = useCase;
    }

    @PostMapping
    public void criar(@RequestBody CriarClienteRequest request) {
        useCase.executar(request.getNome(), request.getEmail());
    }
}
```

---

## 4. Configuração de Injeção de Dependência

Como o `CriarClienteUseCase` é uma classe de domínio pura (sem anotações do Spring), precisamos registrá-lo manualmente.

`infrastructure/config/BeanConfig.java` (Sugerido criar esta pasta de config na infra)

```java
@Configuration
public class BeanConfig {
    @Bean
    public CriarClienteUseCase criarClienteUseCase(ClienteRepository repository) {
        return new CriarClienteUseCase(repository);
    }
}
```

---

## Notas de Aprendizado (Review)

- **Separação de Modelos:** Note que temos `Cliente` (Domínio) e `ClienteEntity` (Banco). Isso permite que seu banco de dados mude (ex: mudar nomes de colunas) sem que sua lógica de negócio sofra qualquer alteração.
- **A Validação:** A validação acontece no construtor do `Cliente` (Domínio). Isso garante que é impossível existir um "Cliente" inválido dentro do seu sistema.
- **O Papel do Use Case:** Ele é o ponto de entrada para qualquer operação. Se amanhã você quiser criar clientes via CLI ou Mensageria (RabbitMQ), você usará o mesmo `CriarClienteUseCase`.
    
- **Inversão de Dependência na Prática:** O `ClienteRepositoryImpl` está na camada de infraestrutura, mas ele implementa uma interface que está no domínio. Isso significa que o domínio dita as regras.