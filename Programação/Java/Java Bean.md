## 1. O que é um Java Bean?

Um **Java Bean** é uma convenção de desenvolvimento (um padrão de design) para classes Java. Diferente do [[POJO]], que é apenas um objeto "puro", o Java Bean segue regras específicas para garantir que ferramentas, frameworks e IDEs consigam manipular o objeto de forma automática e previsível.

O termo surgiu originalmente para criar componentes visuais reutilizáveis, mas hoje é amplamente utilizado em frameworks como Spring e [[Hibernate]] para mapeamento de dados e injeção de dependências.

---

## 2. As 3 Regras Obrigatórias

Para que uma classe Java seja considerada um **Java Bean**, ela deve obrigatoriamente seguir estas normas:

1. **Construtor Público Sem Argumentos:** A classe deve ter um construtor `public` que não recebe parâmetros (construtor default). Isso permite que frameworks criem instâncias da classe dinamicamente via Reflexão (Reflection).
2. **Acesso aos Atributos via Accessors (Getters e Setters):** Os atributos (variáveis) devem ser privados (`private`). O acesso e a modificação desses valores devem ser feitos exclusivamente por métodos que seguem o padrão de nomenclatura `getAtributo()` e `setAtributo()`.
3. **Implementar Serializable:** A classe deve implementar a interface `java.io.Serializable`. Isso permite que o estado do objeto seja convertido em um fluxo de bytes (salvamento em disco, envio pela rede ou armazenamento em sessão).
    

---

## 3. Exemplo Prático: Um Java Bean Padrão

```java
package clientes.infrastructure.implementation.persistence.entity;

import java.io.Serializable;

public class ClienteBean implements Serializable {
    private static final long serialVersionUID = 1L; // Recomendado para Serializable

    private String nome;
    private String email;

    // 1. Construtor público sem argumentos
    public ClienteBean() {
    }

    // 2. Getters e Setters seguindo o padrão de nomenclatura
    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

---

## 4. Diferença Crucial: POJO vs. Java Bean

Embora todo **Java Bean** seja tecnicamente um **POJO**, nem todo POJO é um Java Bean.

|**Característica**|**POJO**|**Java Bean**|
|---|---|---|
|**Dependência**|Nenhuma.|Depende da interface `Serializable`.|
|**Construtor**|Pode ter qualquer construtor.|Deve ter um construtor sem argumentos.|
|**Acesso**|Pode usar atributos públicos.|Atributos obrigatoriamente privados.|
|**Regras**|Nenhuma regra de nomenclatura.|Regras rígidas (`get`/`set`).|
|**Objetivo**|Simplicidade e desacoplamento.|Interoperabilidade com frameworks.|

---

## 5. Por que usamos Java Beans?

- **Introspecção:** Frameworks como o Spring conseguem "olhar" para a classe e descobrir seus atributos apenas lendo o nome dos métodos `get` e `set`.
- **Persistência:** Bibliotecas de Banco de Dados (como [[JPA]]/[[Hibernate]]) precisam do construtor vazio para instanciar o objeto após buscar os dados no banco antes de preenchê-los.
- **Ferramentas de UI:** IDEs podem exibir propriedades de um objeto em painéis de configuração automaticamente se ele seguir o padrão Bean.
    

---

## Notas de Revisão para o seu Projeto

- **No [[Domínio]]:** Você deve preferir **POJOs** (especialmente objetos imutáveis com atributos `final` e sem setters), pois o domínio deve focar na integridade dos dados e não em agradar frameworks.
    
- **Na Infraestrutura (Entities):** Suas classes na pasta `entity` ([[JPA]]) são, na maioria das vezes, **Java Beans**. O [[Hibernate]] exige o construtor público sem argumentos para funcionar corretamente.
    
- **Lombok:** Quando você usa `@Data` ou `@NoArgsConstructor` do Lombok, você está transformando sua classe em um Java Bean de forma automatizada, gerando os métodos e o construtor exigidos pela convenção.