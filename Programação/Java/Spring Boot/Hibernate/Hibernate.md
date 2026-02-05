
## Sobre

**Hibernate** é um **ORM (Object–Relational Mapping)**.

Ele é um **framework concreto** que:
- executa SQL
- gerencia conexões
- controla o ciclo de vida das entidades
- mantém sincronização entre objetos Java e tabelas do banco

Quando você usa **[[JPA]]**, você está usando **uma especificação**.  
Quando você usa **Hibernate**, você está usando **uma implementação real** dessa especificação.

> JPA define _o que_ deve existir. Hibernate define _como_ isso acontece.

---

## JPA vs Hibernate 

### [[JPA]]

- É uma **API / contrato**
- Define:
    - anotações (`@Entity`, `@Id`, etc.)
    - interfaces (`EntityManager`)
    - regras de comportamento
- Não executa nada sozinha
### Hibernate

- É um **engine de persistência**
- Implementa o contrato JPA
- Traduz operações Java em SQL real

Comparação direta:

- JPA → interface
- Hibernate → motor

Usar JPA sem Hibernate é como:

- ter uma interface Java sem nenhuma classe que a implemente

---

## O que o Hibernate faz na prática

Quando você escreve:

```java
userRepository.save(user);
```

O Hibernate executa uma cadeia de responsabilidades:

1. Analisa o estado do objeto
2. Decide se é `INSERT` ou `UPDATE`
3. Gera SQL específico do banco
4. Executa SQL via JDBC
5. Atualiza o estado do objeto em memória

Sem Hibernate:

- você escreveria SQL
- abriria conexão
- executaria `PreparedStatement`
- mapearia colunas manualmente

---

## ORM: o problema que o Hibernate resolve

### Problema original

Java trabalha com:

- objetos
- herança
- identidade

Banco relacional trabalha com:

- tabelas
- linhas
- chaves primárias

Esses modelos **não são equivalentes**.

Hibernate atua como uma **camada de tradução** entre esses dois mundos.

---

## Entity Lifecycle (ciclo de vida de uma entidade)

Hibernate controla estados da entidade:

- **Transient**
    - objeto novo
    - não associado ao banco
- **Managed (Persistent)**
    - associado ao contexto de persistência
    - alterações são rastreadas automaticamente
- **Detached**
    - objeto existe, mas não está sendo rastreado
- **Removed**
    - marcado para exclusão

> Esse controle é uma das principais vantagens do Hibernate.

---

## Persistence Context (conceito central)

O **Persistence Context** é um cache de primeiro nível.

Ele garante que:
- uma entidade com o mesmo ID seja única em memória
- mudanças sejam acumuladas e sincronizadas depois

Exemplo mental:
- Você carrega `User id=1`
- Hibernate garante **uma única instância** desse User

Sem isso:
- você teria múltiplos objetos representando a mesma linha

---

## Dirty Checking (por que setters funcionam sem save)

Hibernate monitora mudanças em entidades gerenciadas.

```java
user.changeName("Novo Nome");
```

Mesmo sem chamar `save()`:

- Hibernate detecta a mudança
- gera `UPDATE`
- sincroniza no `commit`

Isso é chamado de **dirty checking**.

Sem Hibernate:
- você teria que controlar manualmente o que mudou

---

## Lazy Loading (carregamento sob demanda)

Hibernate não carrega tudo imediatamente.

```java
@ManyToOne(fetch = FetchType.LAZY)
private Role role;
```

- `role` começa como um proxy
- SQL só é executado quando o campo é acessado

Por que isso importa:
- evita consultas desnecessárias
- reduz acoplamento
- melhora performance

---

## Hibernate e SQL

Importante:
- Hibernate **não elimina SQL**
- Ele **gera SQL**
- Você ainda precisa:
    - entender índices
    - entender joins
    - analisar planos de execução
Hibernate automatiza o repetitivo, não o pensamento.

---

## Hibernate no Spring Boot

No Spring Boot:
- Hibernate vem como default
- configurado automaticamente
- integrado com transações

Você normalmente **não instancia Hibernate diretamente**.

Fluxo típico:
- Controller → Service → Repository
- Repository usa JPA
- JPA delega ao Hibernate
    

---

## Quando Hibernate vira problema

Hibernate pode atrapalhar quando:
- você não entende o SQL gerado
- usa `EAGER` indiscriminadamente
- ignora o ciclo de vida da entidade

Na maioria dos casos, o problema não é o Hibernate, mas o uso ingênuo dele.



