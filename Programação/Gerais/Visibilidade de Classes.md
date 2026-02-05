# Sobre
Os modificadores de acesso definem **quem pode ver e usar classes, métodos e variáveis**.  
Eles são uma ferramenta de **controle de dependência**, não só de “segurança”.

Em aplicações reais (ex: Spring Boot), eles ajudam a:
- reforçar fronteiras arquiteturais
- evitar acoplamento acidental
- permitir refatorações sem quebrar meio sistema

## 1) `public`

### O que significa
- Pode ser acessado por **qualquer classe**, em **qualquer pacote**.
- É o nível máximo de exposição.

### Quando usar

- Pontos de entrada do sistema (Controllers REST)
- Contratos explícitos (APIs, Services usados por outras camadas)
- Tudo que você **assume como compromisso estável**

```java
public class PublicPark {  
    public String openGate = "The gate is open to everyone.";  
}

PublicPark park = new PublicPark();
System.out.println(park.openGate); // acessível por qualquer um

```

### Observação importante

Em código real, **raramente** você deixaria um campo `public`.  
O exemplo é didático, mas na prática isso vira risco de inconsistência de estado.

Regra prática:

> Classe pode ser `public`. Campo quase nunca.

## 2) `private`

### O que significa
- Só é acessível **dentro da própria classe**.
- Nem subclasses, nem classes do mesmo pacote acessam diretamente.

### Quando usar
- Estado interno
- Métodos auxiliares
- Regras internas que não fazem parte do contrato público

```java
public class Diary {
    private String secret = "top secret";

    public String getSecret() {
        return secret;
    }
}

```

> Implicação: não dá pra fazer `diary.secret` fora da classe.

### Por quê isso é importante?
- Evita dependência em detalhes de implementação
- Permite mudar o funcionamento interno sem quebrar consumidores
- Garante invariantes da classe

Comparação:
- `public` → promessa
- `private` → detalhe descartável

## 3) _Default_ (package-private, sem modificador)

### O que significa

- Visível **apenas dentro do mesmo pacote**.
- Não é acessível de fora, mesmo com `import`.

### Quando usar

- Classes auxiliares de um módulo
- Regras internas que só fazem sentido naquele pacote
- Organização arquitetural por pacote

```java
package com.example.order.service;

class Helper {
    int value;
}

```

> `Helper.value` só é acessível por outras classes **no mesmo pacote**.

### Por que isso é útil?

Porque pacotes podem (e devem) representar **unidades de coesão**.

Em Spring Boot, isso ajuda a:
- impedir que outro módulo “descubra” e use algo indevidamente
- manter a camada bem delimitada

Regra prática:

> Se algo não deveria ser usado fora do pacote, não marque como `public`.

## 4) `protected`

### O que significa
- Acessível:
    - no mesmo pacote
    - em subclasses, mesmo em outros pacotes

### Quando usar

- Hierarquias de herança
- Classes base abstratas
- Template Method

```java
public abstract class BaseService {

    protected void audit(String action) {
        System.out.println("Auditando: " + action);
    }
}

public class OrderService extends BaseService {

    public void create() {
        audit("CREATE_ORDER");
    }
}


```

### Diferença real para o _default_

- _default_: só pacote
- `protected`: pacote **+ herança externa**

### Observação 

Em APIs REST modernas:

- `protected` é **bem menos usado**
- composição costuma ser preferida a herança

Use quando a herança for **intencional**, não por conveniência.

## Classes de topo (regra importante)

Classes de topo **não podem ser `private` nem `protected`**.

Opções válidas:
- `public`
- sem modificador (package-private)

```java
public class OrderService { }   // ok
class OrderService { }          // ok
private class OrderService { }  // NÃO compila

```

`private` só faz sentido para:
- atributos
- métodos
- construtores
- classes internas
## Resumo 

| Visibilidade | Use quando                        |
| ------------ | --------------------------------- |
| `public`     | é contrato, ponto de entrada, API |
| `private`    | é detalhe interno                 |
| default      | colaboração interna de módulo     |
| `protected`  | extensão via herança              |
