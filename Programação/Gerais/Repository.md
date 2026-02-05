## 1. O que é um Repository?

O **Repository** (Repositório) é um padrão que faz a mediação entre a camada de domínio e a camada de persistência de dados. Se o seu sistema fosse um restaurante, o Repository seria o **estoquista**: você pede um ingrediente (objeto) para ele, e ele sabe exatamente em qual prateleira (tabela do banco) buscar ou guardar, sem que você precise saber como o estoque está organizado internamente.

Ele abstrai a complexidade do acesso a dados, tratando a fonte de dados (banco, API externa, arquivo) como se fosse uma coleção de objetos em memória.

---

## 2. O Que Resolve e Para Que Serve?

O padrão Repository resolve o acoplamento direto entre a lógica de negócio e as consultas SQL ou tecnologias de banco de dados.

- **Abstração:** Esconde os detalhes da infraestrutura (se é MySQL, MongoDB ou uma lista em memória).
- **Centralização:** Centraliza a lógica de seleção de dados (filtros, queries complexas) em um só lugar.
- **Testabilidade:** Permite substituir o banco de dados real por um "Fake" ou "Mock" em testes unitários, sem alterar a lógica de negócio.

---

## 3. Funcionamento em Java Puro ([[POJO]])

Em Java puro, definimos o Repository como uma **Interface**. Isso garante que o domínio dependa de uma abstração e não de uma implementação específica.

```Java
// A interface reside no Domínio
public interface PedidoRepository {
    void salvar(Pedido pedido);
    Pedido buscarPorId(Long id);
    List<Pedido> listarTodos();
}

// A implementação reside na Infraestrutura
public class PedidoRepositoryMemoria implements PedidoRepository {
    private List<Pedido> db = new ArrayList<>();

    @Override
    public void salvar(Pedido pedido) {
        db.add(pedido);
    }

    @Override
    public Pedido buscarPorId(Long id) {
        return db.stream().filter(p -> p.getId().equals(id)).findFirst().orElse(null);
    }

    @Override
    public List<Pedido> listarTodos() {
        return new ArrayList<>(db);
    }
}
```

---

## 4. Funcionamento no Spring Framework

No Spring Boot, o padrão é simplificado pelo **Spring Data [[JPA]]**. Você define apenas a interface e o framework gera a implementação em tempo de execução.

- **Anotação:** Usa-se `@Repository` na implementação (opcional quando se estende interfaces do Spring).
- **Abstração:** Ao estender `JpaRepository`, você ganha métodos como `save()`, `findById()` e `findAll()` automaticamente.

```Java
@Repository
public interface PedidoRepository extends JpaRepository<Pedido, Long> {
    // O Spring cria a query automaticamente pelo nome do método
    List<Pedido> findByStatus(String status);
    
    // Ou via query customizada
    @Query("SELECT p FROM Pedido p WHERE p.valor total > :valor")
    List<Pedido> buscarPedidosCaros(@Param("valor") BigDecimal valor);
}
```

---

## 5. Quando Usar e Por Que?

O uso é recomendado em praticamente qualquer aplicação profissional que lide com persistência.

- **Quando usar:** Sempre que houver necessidade de persistir e recuperar entidades de domínio.
- **Por que usar:** 1. **Troca de Tecnologia:** Se você decidir trocar o [[Hibernate]] por JDBC puro, o seu `UseCase` não sofrerá nenhuma alteração.
    2. **Clean Code:** Evita que comandos SQL sujem a clareza da sua regra de negócio.
    3. **Consistência:** Garante que a forma de acessar um dado seja a mesma em qualquer parte do sistema.

---

## 6. Comparação de Responsabilidades

|**Camada**|**O que faz?**|**O que NÃO deve fazer?**|
|---|---|---|
|**[[Estudos/Programação/Gerais/# Use Case]]**|Pede dados ao Repository para processar.|Não escreve SQL nem abre conexões com banco.|
|**Repository**|Traduz objetos para registros e vice-versa.|Não contém regras de negócio ou validações.|
|**Infraestrutura**|Implementa a conexão (JPA, JDBC, MongoDB).|Não decide o fluxo da aplicação.|

---

## Notas de Revisão

- **Interface vs Implementação:** Sempre injete a Interface no seu Use Case, nunca a classe concreta. Isso respeita o **D** do SOLID (Inversão de Dependência).
- **Mapeamento:** O Repository frequentemente trabalha em conjunto com [[Mappers]] para converter Entidades de Banco (DAO) para Entidades de Domínio (POJO).
- **Escopo:** Um Repository deve ser responsável por uma única **Agregação** (um conjunto de objetos relacionados que são tratados como uma unidade).