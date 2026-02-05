# De Objetos Puros ao Spring Framework

## 1. O Conceito: Controle de Dependências

Toda aplicação complexa é composta por classes que dependem de outras. Se a Classe A precisa da Classe B para funcionar, dizemos que **B é uma dependência de A**.

### O Jeito "Antigo" (Acoplamento Forte)

Neste modelo, a classe cria suas próprias dependências.

```java
public class RelatorioService {
    private final EmailService emailService = new EmailService(); // Instanciação manual

    public void enviar() {
        emailService.enviar("Relatório pronto");
    }
}
```

**Problema:** Você não consegue testar `RelatorioService` sem criar um `EmailService` real. Se quiser mudar para um `SmsService`, terá que alterar o código de todas as classes que instanciam o serviço de e-mail.

---

## 2. Injeção de Dependência (DI) Manual

Em Java puro (sem frameworks), resolvemos isso passando a dependência pelo **construtor**. Isso é Injeção de Dependência.

```java
public class RelatorioService {
    private final EmailService emailService;

    // A classe não cria mais o objeto, ela o RECEBE pronto
    public RelatorioService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

Agora, quem "manda" no sistema precisa criar as peças e montá-las:

```java
// No método main (O montador)
EmailService email = new EmailService();
RelatorioService relatorio = new RelatorioService(email);
```

---

## 3. O Spring e o Inversion of Control (IoC)

O Spring Framework facilita isso criando um **Container de IoC**. Ele funciona como uma "caixa" que guarda as instâncias das suas classes (chamadas de **Beans**).

Existem duas formas principais de colocar um objeto dentro dessa "caixa" do Spring:

### A) Forma Automática (Stereotypes)

Você anota a classe com `@Component`, `@Service` ou `@Repository`. O Spring varre o projeto, encontra essas anotações e cria o objeto sozinho.

- **Vantagem:** Rápido e prático.
- **Desvantagem:** Polui o código de negócio com anotações de framework.

### B) Forma Manual (@Configuration + @Bean)

Você cria uma classe de configuração para dizer ao Spring como instanciar objetos que **não possuem anotações**.

```java
@Configuration
public class MinhaConfiguracao {

    @Bean // Indica que o retorno deste método deve ser um objeto gerenciado pelo Spring
    public RelatorioService relatorioService(EmailService emailService) {
        return new RelatorioService(emailService);
    }
}
```

---

## 4. Por que usar `@Bean` em vez de `@Service`?

Existem três cenários principais onde o registro manual via `@Bean` é obrigatório ou preferível:

1. **Classes de Terceiros:** Se você usa uma biblioteca externa (como o SDK da AWS ou uma biblioteca de PDF), você não pode abrir o código deles para colocar um `@Service`. Você precisa instanciá-los via `@Bean`.
    
2. **Desacoplamento de Framework:** Em arquiteturas limpas, queremos que o núcleo do sistema (Domínio) seja Java puro ([[POJO]]). Se você colocar `@Service` no seu UseCase, seu domínio agora depende do Spring. Usando `@Configuration`, o domínio fica "limpo" e o framework fica restrito à camada de infraestrutura.
    
3. **Configuração Condicional:** Você pode usar `@Bean` para decidir, em tempo de execução, qual implementação será criada (ex: "Se for ambiente de teste, crie o Bean de Mock; se for produção, crie o Bean real").
    

---

## 5. Resumo da Sintaxe no Spring

|**Termo**|**Função**|
|---|---|
|**@Configuration**|Avisa ao Spring: "Esta classe contém receitas de como criar objetos".|
|**@Bean**|Colocado sobre um método. O objeto retornado será guardado pelo Spring para ser usado em outros lugares.|
|**Injeção via Construtor**|Quando um método `@Bean` ou classe `@Service` pede um parâmetro, o Spring olha na sua "caixa de Beans" e entrega a instância correta automaticamente.|

---

## Notas de Revisão

- O Spring só consegue injetar o que ele conhece. Se você tentar usar uma classe que não é `@Component` e não foi declarada como `@Bean`, o Spring lançará um erro de `NoSuchBeanDefinitionException`.
- O uso de `@Configuration` centraliza a fiação (wiring) do sistema, facilitando a visualização de como as peças se conectam.