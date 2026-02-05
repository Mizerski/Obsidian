## O que é o Design Pattern Adapter?

O **Adapter** é um padrão estrutural que permite que objetos com interfaces incompatíveis trabalhem juntos. Em arquiteturas como a [[Arquitetura hexagonal]] ([[Portas]] & Adapters), ele atua como um "conversor" entre o mundo externo e o núcleo da aplicação.

**Analogia:** Pense no **Adaptador de Tomada Universal**.

- **A Tomada (Mundo Externo):** Pode ser o padrão Europeu, Americano ou Brasileiro (JSON, SQL, Mensageria).
- **O Aparelho (Seu [[Domínio]]):** Tem um plugue fixo que nunca muda (Interfaces/Ports).
- **O Adaptador:** É a peça que você encaixa entre os dois para que a energia flua sem precisar trocar o cabo do aparelho.
    

---

## 2. Direcionalidade: Driving vs. Driven

A ambiguidade geralmente desaparece quando você classifica o adaptador pela **direção do fluxo de controle**:

### A. [[Entry Adapters]] (Driving / Entrada)

Eles **iniciam** a ação. Eles "dirigem" a aplicação.

- **Exemplos:** REST Controllers, CLI (Linha de comando), Consumidores de Fila (RabbitMQ Listener).
- **O que faz:** Traduz o protocolo externo (HTTP/JSON) para uma chamada de método no seu **[[Estudos/Programação/Gerais/# Use Case]]**.

### B. Persistence/[[Infrastructure Adapters]] (Driven / Saída)

Eles **reagem** a uma ação. Eles são "dirigidos" pela aplicação.

- **Exemplos:** [[Repository]] ([[JPA]]/NoSQL), Adaptadores de E-mail (SendGrid), Clients de API Externa (Feign/Retrofit).
- **O que faz:** Traduz a necessidade do domínio ("Preciso salvar isso") para uma tecnologia específica (SQL, NoSQL, Chamada REST externa).
    

---

## 3. O Problema que ele resolve: O Acoplamento

Sem o Adapter, o seu código de negócio ficaria poluído com detalhes técnicos. Se o banco mudar ou se você precisar expor a mesma lógica via gRPC em vez de REST, você teria que reescrever o coração do sistema.

**O Adapter isola o impacto da mudança.**

---

## 4. Funcionamento Prático ([[POJO]] vs. Framework)

Para diferenciar os tipos de adaptadores, observe quem chama quem.

### Exemplo [[Entry Adapters]] (Entrada) - Spring Boot

O Framework recebe o sinal e o Adapter "empurra" para dentro do sistema.

```Java
@RestController // Framework (Spring) cuida da recepção
public class OrdemCompraController { // O ADAPTER

    private final ProcessarCompraUseCase useCase; // A PORTA (Interface)

    @PostMapping("/compras")
    public void criar(@RequestBody CompraDTO dto) {
        // Converte DTO para Objeto de Domínio e envia para dentro
        useCase.executar(dto.toDomain()); 
    }
}
```

### Exemplo [[Persistence Adapters]] (Saída) - POJO / JDBC

O sistema chama a porta e o Adapter "puxa" a execução para fora.

```Java
public class MySqlEstoqueAdapter implements EstoqueRepository { // O ADAPTER implementa a PORTA

    @Override
    public void atualizarSaldo(Long id, Integer qtd) {
        // Traduz comando de domínio para tecnologia específica (JDBC/SQL)
        String sql = "UPDATE estoque SET saldo = ? WHERE id = ?";
        // ... execução técnica do SQL
    }
}
```

---

## 5. Quando usar e Por quê?

|**Situação**|**Usar Adapter?**|**Por quê?**|
|---|---|---|
|**Integração com API Externa**|Sim|Para que o domínio não dependa do formato do JSON de terceiros.|
|**Persistência de Dados**|Sim|Para poder trocar ou atualizar o banco de dados sem tocar na lógica de negócio.|
|**Exposição de Funcionalidades**|Sim|Para permitir múltiplos pontos de entrada (Web, Mobile, Jobs agendados).|
|**Lógica de Cálculo Interna**|**Não**|Isso deve estar no Domínio/Entidade, sem adaptadores.|

---

## 6. Resumo de Identificação Rápida

Para acabar com a ambiguidade na sua cabeça, faça estas perguntas ao ler um código:

1. **Quem está chamando quem?** * Mundo externo chamando meu código? -> **Entry Adapter**.
    
    - Meu código chamando algo externo? -> **Persistence/Output Adapter**.
2. **O que está sendo traduzido?**
    
    - JSON/Request para Domínio? -> **Entry**.
    - Domínio para Tabela/Linha ou Log/E-mail? -> **Output**.

---

## Notas de Revisão

- **Localização Sugerida:** No pacote `infrastructure/adapters/inbound` (para API) e `infrastructure/adapters/outbound` (para Persistência).
- **O Contrato:** Todo adaptador deve, idealmente, estar escondido atrás de uma **Interface** (Porta) que reside na camada de aplicação ou domínio.