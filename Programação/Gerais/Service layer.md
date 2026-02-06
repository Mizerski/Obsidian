## 1. O Que É

A **Service Layer** (Camada de Serviço) é um padrão de projeto arquitetural que estabelece um limite (boundary) entre as camadas de interface (Controller/API) e a persistência de dados (Repository). Ela atua como um orquestrador, onde a [[Lógica de Negócio]] é centralizada e as transações são gerenciadas.

### Analogia do Mundo Real

Pense em um **Garçom em um Restaurante**:

- O **Cliente (Controller)** faz o pedido.
- O **Garçom (Service)** recebe o pedido, verifica se os ingredientes estão disponíveis, coordena com a cozinha e garante que o prato seja entregue conforme as regras da casa.
- A **Cozinha/Despensa (Repository)** apenas fornece ou armazena os insumos.
- O cliente não vai até a cozinha, e o cozinheiro não atende a mesa. O Garçom é a camada que orquestra a "operação de negócio" de servir uma refeição.

---

## 2. O Que Resolve

A ausência de uma Service Layer geralmente leva ao [[Anti-pattern]] conhecido como _Fat Controller_ ou _Smart UI_. A implementação resolve:

- **Acoplamento Excessivo:** Impede que a lógica de negócio fique presa ao protocolo de entrada (ex: HTTP/REST) ou à tecnologia de banco de dados.
- **Duplicação de Código:** Evita que a mesma regra de validação ou cálculo precise ser escrita em múltiplos Controllers.
- **Complexidade de Transação:** Facilita o controle de atomicidade (garantir que ou tudo é salvo, ou nada é salvo).
- **Dificuldade em Testes:** Permite testar processos de negócio sem a necessidade de simular requisições HTTP complexas ou conexões reais com BD.

---

## 3. Diferenciação e Variações

Para evitar ambiguidades, é necessário distinguir como a Service Layer é estruturada:

- **Application Service:** Focado na orquestração de infraestrutura (enviar e-mail, buscar no repositório, logging). Não contém regras de negócio complexas, apenas coordena o fluxo.
- **Domain Service:** Utilizado quando uma regra de negócio não pertence naturalmente a uma única entidade (ex: uma transferência bancária que envolve duas contas diferentes).
- **Service Interface vs. Implementation:** Em sistemas complexos, utiliza-se uma `interface` (ex: `OrderService`) e uma implementação (ex: `OrderServiceImpl`) para permitir diferentes comportamentos e facilitar o uso de [[Design Patterns]] como Strategy.
---

## 4. Exemplo Prático

### Implementação Pura (POJO/Java Simples)

Focada na lógica manual e controle de fluxo sem auxílio de containers.

#### Java

```
public class OrderService {
    private final OrderRepository repository;

    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }

    public void processOrder(Order order) {
        if (order.getTotal() > 0) {
            repository.save(order);
        }
    }
}
```

### Implementação Spring Boot

Utiliza Inversão de Controle (IoC) e Gerenciamento de Transações via [[AOP]].
#### Java

```
@Service
public class OrderService {
    
    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Transactional
    public OrderResponseDTO createOrder(OrderRequestDTO dto) {
        // Orquestração: Converte DTO -> Entidade -> Salva -> Converte DTO
        Order order = OrderMapper.toEntity(dto);
        validateStock(order);
        Order savedOrder = orderRepository.save(order);
        return OrderMapper.toResponse(savedOrder);
    }
}
```

---

## 5. Tabela Comparativa

| **Conceito**      | **Responsabilidade Principal**             | **O que NÃO deve fazer**                      |
| ----------------- | ------------------------------------------ | --------------------------------------------- |
| **Controller**    | Lidar com HTTP, status codes e rotas.      | Executar cálculos ou persistir dados.         |
| **Service**       | Orquestrar fluxo, transações e regras.     | Manipular diretamente o `HttpServletRequest`. |
| **Repository**    | Abstração do acesso a dados (SQL/NoSQL).   | Decidir se um cliente tem crédito ou não.     |
| **Domain/Entity** | Representar os dados e regras intrínsecas. | Chamar outros serviços ou repositórios.       |

---

## 6. Notas de Revisão

### Localização de Pastas (Padrão de Pacotes)

- `com.empresa.projeto.service`: Contém as classes de serviço.
- `com.empresa.projeto.service.impl`: (Opcional) Contém implementações caso use interfaces.
### Regras de Dependência e Boas Práticas

1. **Direção única:** Controllers chamam Services. Services chamam Repositories ou outros Services. **Nunca** um Repository deve chamar um Service.
2. **Injeção por Construtor:** Prefira sempre injeção via construtor em vez de `@Autowired` em campos para facilitar a imutabilidade e testes unitários.
3. **Tratamento de Exceções:** Lance exceções de negócio customizadas na Service Layer e capture-as em um Global Exception Handler no Controller.
4. **Stateless:** Services devem ser "sem estado". Eles não devem armazenar dados de um usuário específico em variáveis de instância (fields).