## 1. O que é a Camada de Infraestrutura?

A camada de **Infraestrutura** é a periferia do seu sistema. É nela que residem os detalhes técnicos e as ferramentas externas. O **Persistence Adapter** (Adaptador de Persistência) é a parte da infraestrutura responsável por falar com o banco de dados.

**Analogia:** Pense na infraestrutura como o **driver de uma impressora**.

- O computador (seu sistema) envia um comando genérico: "Imprimir documento".
- O driver (adaptador) sabe exatamente como traduzir esse comando para os pulsos elétricos específicos que a impressora da marca X ou Y entende.

---

## 2. Por que usar um "Adaptador"? 

O banco de dados entende **Tabelas e Linhas**. O seu [[Domínio]] entende **Objetos e Regras de Negócio**.

O Adaptador de Persistência serve para fazer a **tradução** entre esses dois mundos.

**Responsabilidades do Adaptador:**

1. **Mapeamento:** Converter um Objeto de Domínio em uma Entidade de Banco (e vice-versa).
2. **Comunicação:** Executar comandos SQL ou usar um ORM (como [[JPA]]/[[Hibernate]]).
3. **Tratamento de Erros Técnicos:** Capturar exceções de banco (ex: "Conexão perdida") e decidir o que fazer.

---

## 3. Funcionamento Fora de Frameworks ([[POJO]] / JDBC)

Sem frameworks modernos, o adaptador usaria JDBC puro para traduzir os dados.

```Java
// Implementação manual da Porta (Interface)
public class MySqlClienteAdapter implements ClienteRepository {
    
    @Override
    public void salvar(Cliente cliente) {
        // Tradução manual: Objeto -> SQL
        String sql = "INSERT INTO clientes (nome, email) VALUES (?, ?)";
        
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setString(1, cliente.getNome());
            stmt.setString(2, cliente.getEmail());
            stmt.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException("Erro técnico no banco de dados", e);
        }
    }
}
```

---

## 4. Funcionamento Dentro do Framework (Spring + JPA)

No Spring, usamos o **EntityManager** ou **Spring Data JPA** para facilitar. O adaptador agora foca em converter os objetos.

```Java
@Repository // Avisa ao Spring que este é um componente de infraestrutura
public class JpaClienteAdapter implements ClienteRepository {

    private final EntityManager entityManager;

    public JpaClienteAdapter(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    @Override
    public void salvar(Cliente cliente) {
        // 1. Tradução: Domínio -> Entidade JPA
        ClienteEntity entity = new ClienteEntity();
        entity.setNome(cliente.getNome());
        entity.setEmail(cliente.getEmail());

        // 2. Persistência usando a ferramenta (JPA)
        entityManager.persist(entity);
    }
}
```

---

## 5. Por que separar Entidade de Domínio e Entidade de Banco?

Muitas pessoas cometem o erro de usar a mesma classe para os dois, mas separar traz benefícios críticos:

| **Objeto de Domínio (Cliente)**             | **Entidade de Banco (ClienteEntity)**                |
| ------------------------------------------- | ---------------------------------------------------- |
| Vive na camada de **Domínio**.              | Vive na camada de **Infraestrutura**.                |
| Foca em **Regras de Negócio** e validações. | Foca em **Mapeamento de Tabelas** (@Table, @Column). |
| É um **[[POJO]]** (sem dependências).       | É um **Java [[Bean]]** (com anotações de framework). |
| Se a regra de negócio mudar, ele muda.      | Se o nome da coluna no banco mudar, ele muda.        |

---

## 6. O Fluxo de Dependência

Uma regra fundamental da arquitetura moderna é: **A Infraestrutura depende do Domínio, mas o Domínio nunca depende da Infraestrutura.**

Isso significa que, se você decidir trocar o MySQL pelo MongoDB, você cria um novo `MongoClienteAdapter` na pasta de infraestrutura, implementa a mesma interface (`ClienteRepository`) e o resto do seu sistema ([[Domínio]] e API) **não sofre nenhuma alteração**.

---

## Notas de Revisão

- **Localização:** No seu projeto, isso fica em `infrastructure/implementation/persistence`.
- **Substituibilidade:** O objetivo de um adaptador é ser substituível. Se for difícil trocar a forma de salvar os dados, seu adaptador está muito "misturado" com a lógica de negócio.
- **Mapeadores:** Em projetos grandes, é comum usar ferramentas como **MapStruct** ou criar classes `Mapper` para automatizar a conversão entre `Cliente` e `ClienteEntity`.