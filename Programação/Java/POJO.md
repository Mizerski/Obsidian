## 1. O que é um POJO?

O termo **POJO** é um acrônimo para **Plain Old Java Object** (em português: "Simples Objeto Java Antigo").

Criado por Martin Fowler, Rebecca Parsons e Josh MacKenzie em 2000, o termo foi cunhado para dar um nome "atraente" a objetos simples que não dependiam de frameworks complexos. Na época, era comum o uso de tecnologias pesadas (como EJB 2.x) que forçavam as classes a herdar de diversas interfaces de sistema.

**Definição técnica:** Um POJO é uma classe Java que não possui restrições além daquelas forçadas pela especificação da linguagem Java.

---

## 2. Características de um POJO

Para que uma classe seja considerada um POJO puro, ela:

1. **Não pode estender classes pré-definidas de frameworks** (ex: `extends ActionForm` do Struts ou `extends HibernateDaoSupport`).
2. **Não pode implementar interfaces obrigatórias de frameworks** (ex: `implements javax.ejb.EntityBean`).
3. **Não deve possuir anotações de frameworks de terceiros** que ditem seu comportamento (embora na prática moderna, anotações de conveniência como Lombok sejam aceitas).
4. **Independência de ambiente:** Ele deve ser capaz de rodar em qualquer máquina virtual Java (JVM) sem precisar de um servidor de aplicação ou biblioteca externa.

---

## 3. Exemplo Prático: POJO vs. Não-POJO

### Exemplo de um POJO (Simples e Puro)

Este objeto representa apenas o conceito de negócio. Ele é portátil e fácil de testar.

```java
public class Produto {
    private String nome;
    private Double preco;

    public Produto(String nome, Double preco) {
        this.nome = nome;
        this.preco = preco;
    }

    public void aplicarDesconto(Double percentual) {
        this.preco -= this.preco * (percentual / 100);
    }

    // Getters e Setters
}
```

### Exemplo de um Não-POJO (Acoplado)

Note como esta classe está "presa" a tecnologias externas.

```java
import jakarta.persistence.*; // Dependência de JPA
import org.springframework.data.annotation.Id; // Dependência de Spring

@Entity
@Table(name = "produtos")
public class ProdutoEntity extends BaseFrameworkClass { // Herança forçada
    @Id
    @GeneratedValue
    private Long id;
    
    // Esta classe só faz sentido dentro de um contexto com Banco de Dados e Spring
}
```

---

## 4. Importância na Arquitetura Hexagonal

Na arquitetura que você está estudando, o POJO é a peça central da camada de **Domínio**.

- **Testabilidade:** Como o POJO não depende de banco de dados ou web, você pode criar testes unitários extremamente rápidos.
- **Desacoplamento:** O seu domínio (regra de negócio) fica protegido. Se você decidir trocar o Spring por outro framework no futuro, os seus POJOs de domínio permanecerão intactos, pois não conhecem nenhuma tecnologia externa.
- **Simplicidade:** O desenvolvedor foca na lógica do problema, não na configuração do framework.

---

## 5. POJO vs. Java Bean

É comum confundir os dois, mas existem diferenças sutis:

- **POJO:** Qualquer objeto Java que não tenha dependências externas forçadas.
- **Java Bean:** É um tipo específico de POJO que segue convenções rígidas:
    1. Deve ter um construtor público sem argumentos (default).
    2. Atributos devem ser privados e acessados via Getters e Setters.
    3. Deve ser serializável (`implements Serializable`).

---

## Notas de Revisão

- **O Domínio deve ser POJO:** Sempre que criar uma classe em `domain/model`, pergunte-se: "Se eu remover o Spring do projeto, esta classe continua compilando?". Se a resposta for sim, ela é um POJO.
    
- **Entity não é POJO:** Classes anotadas com `@Entity` são tecnicamente "Managed Beans" ou objetos de infraestrutura, pois possuem alto acoplamento com a especificação JPA.
    
- **Lombok:** Embora o Lombok seja uma biblioteca externa, o uso de `@Getter` ou `@Setter` é aceito na maioria das arquiteturas modernas de domínio, pois ele é apenas um gerador de código em tempo de compilação e não interfere na lógica em tempo de execução.