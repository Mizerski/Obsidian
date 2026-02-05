
## Sobre

JPA (**Java Persistence API**) é uma **abstração padrão** para mapeamento objeto–relacional (ORM).  
Ela define _contratos_ e _anotações_, mas **não executa nada sozinha**.
No Spring Boot, quem normalmente executa esses contratos é o **[[Hibernate]]**.
Em termos práticos:

- Sem JPA → você escreve SQL, mapeia `ResultSet`, gerencia transações e identidade
- Com JPA → você descreve o modelo em Java e o provider cuida da persistência

> JPA não elimina SQL. Ele **adianta decisões de mapeamento** e centraliza regras no domínio.

---

## O que uma Entity realmente é

Uma entidade JPA é apenas uma **classe Java comum**, com algumas regras:

- precisa ser instanciável por reflexão
- possui identidade estável (ID)
- representa um estado persistente

Ela **não é**:

- DTO
- modelo de API
- classe utilitária

Entity representa **estado de [[Estudos/Programação/Gerais/Domínio]] + identidade**, não transporte de dados.

---

## Estrutura mínima de uma Entity

```java
@Entity
@Table(name = "users")
public class User {
}
```

### `@Entity`

- Marca a classe como gerenciada pelo JPA
- Sem isso, o Hibernate ignora completamente a classe

### `@Table`

- Opcional
- Recomendado quando:
    - o nome da tabela não é trivial
    - você quer evitar dependência de naming strategy implícita
> Boa prática: ser explícito quando o nome importa.

---

## Identidade da entidade (ID)

### Padrão mais usado hoje

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

### Por que esse padrão é comum

- `Long` (wrapper, não `long`)
    - permite estado **transiente** (`id == null`)
- `IDENTITY`
    - simples
    - funciona bem com PostgreSQL, MySQL, H2
    - não exige configuração de `sequence`

### Alternativa relevante

```java
@GeneratedValue(strategy = GenerationType.SEQUENCE)
```

- Melhor performance em inserts em lote
- Requer sequence no banco
- Escolha consciente em sistemas de alto volume

> Regra prática: comece com `IDENTITY`. Mude apenas se houver motivo técnico claro.

---

## Mapeamento de colunas

```java
@Column(nullable = false, length = 100)
private String name;
```

### O que isso faz de verdade
- Define restrições no schema
- Permite validação **antes** do SQL ser executado

### Comparação direta

**Sem JPA**:
- regra só no banco
- erro tardio (`SQLException`)

**Com JPA**:
- parte da regra sobe para o domínio
- falha acontece mais cedo

> Use `@Column` quando existir regra real. Evite anotar tudo por hábito.

---

## Construtores e [[JPA]]

### Requisito obrigatório

```java
protected User() {
}
```

- JPA instancia entidades via reflexão
- construtor sem argumentos é obrigatório
- `protected` é preferível a `public`

### Construtor de domínio

```java
public User(String name) {
    this.name = name;
}
```

> Construtor do JPA ≠ construtor do domínio.

Misturar os dois geralmente resulta em entidades inválidas.

---

## [[Lombok]] em Entities 

### O que é aceito

```java
@Getter
```

- reduz ruído
- não altera semântica

### O que evitar

```java
@Setter // ❌ em nível de classe
@Data  // ❌ quase sempre
```

### Por quê evitar

- Entidade não é DTO
- `@Data` gera:
    - `equals`
    - `hashCode`
    - `toString`
- Isso interfere em:
    - identidade JPA
    - proxies
    - lazy loading

### Padrão saudável

- campos privados
- setters apenas quando fizerem sentido
- métodos com intenção explícita

```java
public void changeName(String name) {
    this.name = name;
}
```

---

## Relacionamentos

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "role_id")
private Role role;
```

### Padrões modernos

- `LAZY` por padrão
- `@JoinColumn` explícito
- evitar `EAGER` como default

> `EAGER` costuma carregar mais dados do que o necessário e cria acoplamento implícito.

---

## equals e hashCode (nota crítica)

- Nunca basear em campos mutáveis
- Normalmente basear apenas no `id`

Implementação incorreta causa bugs difíceis em:

- `Set`
- cache
- dirty checking

---

## O que NÃO usar em Entities
- `record`
- campos `final`
- lógica de infraestrutura (HTTP, IO, chamadas externas)

Java 17 entra mais forte em:
- DTOs
- serviços
- testes
- código fora do modelo persistente

---

## Template final recomendado

```java
@Entity
@Table(name = "users")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    public User(String name) {
        this.name = name;
    }

    public void changeName(String name) {
        this.name = name;
    }
}
```

---

