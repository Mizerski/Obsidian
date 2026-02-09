Esta nota de estudo técnica detalha o funcionamento e a gestão de memória no ecossistema Java, focando no Garbage Collector (GC).

---

## 1. O que é o Garbage Collector

O **Garbage Collector** é um processo automático de gerenciamento de memória da Java Virtual Machine (JVM). Sua função é identificar e descartar objetos que não são mais acessíveis por nenhum ponto da aplicação, liberando o espaço correspondente na memória **Heap**.

**Analogia:** Imagine um restaurante onde os clientes (threads da aplicação) solicitam pratos (objetos). Conforme os clientes terminam de comer e saem da mesa (o método termina e a referência é perdida), os pratos sujos permanecem ocupando espaço. O Garbage Collector é o **garçom** que circula pelo salão, identifica quais mesas não têm mais clientes e limpa os pratos para que novos clientes possam sentar.

---

## 2. O que resolve

A existência do GC soluciona problemas críticos de gerenciamento manual de memória (comum em linguagens como C/C++):

- **Memory Leaks (Vazamento de Memória):** Evita que objetos inúteis continuem ocupando RAM até o esgotamento do sistema.
- **Dangling Pointers (Ponteiros Soltos):** Impede que o sistema tente acessar um endereço de memória que já foi liberado.
- **Complexidade de Desenvolvimento:** O desenvolvedor não precisa escrever código para desalocação (`free()` ou `delete`), focando apenas na lógica de negócio.
- **Double Free:** Evita o erro de tentar liberar a mesma região de memória duas vezes, o que causaria instabilidade na JVM.

---

## 3. Diferenciação: Stack vs. Heap

Para entender o GC, é preciso diferenciar onde ele atua:

- **Stack (Pilha):** Armazena tipos primitivos e **referências** para objetos. Cada thread tem sua própria Stack. A limpeza é automática conforme o escopo do método termina. O GC **não** atua aqui.
- **Heap:** Espaço compartilhado onde os objetos reais residem. É o território de atuação do **Garbage Collector**.

---

## 4. Exemplo Prático: Gerenciamento de Ciclo de Vida

Embora o GC seja automático, a forma como estruturamos o código define quando um objeto se torna elegível para coleta.

### Implementação Pura (POJO/Java Simples)

Aqui, o objeto torna-se elegível para o GC assim que a referência sai do escopo ou é anulada.

```java
public class RelatorioService {
    public void gerarRelatorio() {
        // Objeto alocado na Heap
        byte[] dadosPesados = new byte[1024 * 1024 * 10]; // 10MB
        
        processar(dadosPesados);
        
        // Ao sair deste método, a referência 'dadosPesados' na Stack morre.
        // O objeto de 10MB na Heap agora é "lixo" para o GC.
    }
}
```

### Implementação com Framework (Spring Boot)

O Spring gerencia o ciclo de vida através de **Beans**. Objetos com escopo `Singleton` (padrão) permanecem na Heap durante toda a vida da aplicação e raramente são coletados.

```java
@Service
public class RelatorioService {
    // Este objeto reside na Heap e NÃO é coletado enquanto o app rodar,
    // pois o Spring Context mantém uma referência constante para ele.
    private final Repository repository;

    public RelatorioService(Repository repository) {
        this.repository = repository;
    }
}
```

---

## 5. Tabela Comparativa de Algoritmos GC

|**Algoritmo**|**Mecanismo Principal**|**Ponto Forte**|**Ponto Fraco**|
|---|---|---|---|
|**Serial**|Single-thread (Stop-the-world)|Baixo overhead|Pausas longas; não escala.|
|**Parallel**|Multi-thread para throughput|Processamento em lote|Latência alta em coletas.|
|**G1**|Divisão por regiões|Equilíbrio (Padrão Java 9+)|Uso maior de CPU.|
|**ZGC**|Coleta concorrente (sem pausas)|Latência < 1ms|Apenas heaps grandes (64-bit).|
|**Shenandoah**|Compactação concorrente|Baixíssima latência|Complexidade e uso de CPU.|

---

## 6. Notas de Revisão e Boas Práticas

- **Evite Finalizers:** Nunca utilize o método `finalize()`. Ele atrasa a coleta do objeto e pode causar gargalos severos. Use `try-with-resources` para fechar arquivos/conexões.
- **Ajuste de Memória:** Use as flags `-Xms` (tamanho inicial) e `-Xmx` (tamanho máximo) para evitar que a Heap cresça e diminua constantemente, o que aciona o GC desnecessariamente.
- **Perfilamento (Profiling):** Utilize ferramentas como **VisualVM** ou **JProfiler** para monitorar a frequência de "Major GC" (coleta completa). Muitas coletas indicam que o tamanho da Heap está subestimado.
- **Dependências:** No Spring, evite criar objetos grandes dentro de métodos que são chamados em loops de alta frequência, pois isso sobrecarrega a "Young Generation" (área de objetos novos) do GC.