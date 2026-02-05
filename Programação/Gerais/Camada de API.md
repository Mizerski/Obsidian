## 1. O que é a Camada de API?

A camada de API é o **Adapter de Entrada** (ou _Driving Adapter_). Ela é a interface de comunicação entre o mundo externo (usuários, front-end, outros microserviços) e a sua lógica de negócio.

**Analogia:** Pense no **Garçom** de um restaurante.

- O cliente (mundo externo) olha o menu e faz um pedido.
- O garçom (API) anota o pedido em um formato que a cozinha entende (comanda).
- O garçom não cozinha; ele apenas traduz o desejo do cliente em uma instrução para o cozinheiro ([[Use Case]]).
    

---

## 2. Por que "Entry Adapter"?

Na arquitetura moderna, o sistema é "dirigido" (_driven_) por estímulos externos. A camada de API "adapta" esses estímulos:

- **Entrada:** Um JSON via HTTP, uma linha de comando via terminal, ou uma mensagem de um broker (RabbitMQ).
- **Adaptação:** O Adapter converte esse formato específico para uma chamada de método no seu [[**Use Case**]].

---

## 3. Responsabilidades da Camada de API

|**O que ela FAZ**|**O que ela NÃO FAZ**|
|---|---|
|Define os caminhos (Endpoints/URLs).|Não contém lógica de decisão de negócio.|
|Valida o formato dos dados (ex: "O campo email é um texto?").|Não valida regras de negócio (ex: "Este email já existe?").|
|Converte JSON para Objetos Java (DTOs).|Não acessa o Banco de Dados.|
|Define os códigos de status (200 OK, 404 Not Found, 201 Created).|Não decide como um objeto deve ser salvo.|

---

## 4. Funcionamento Fora de Frameworks ([[POJO]])

Sem Spring, você usaria **Servlets** ou um simples método `main` para ler entradas do teclado. O princípio é o mesmo: capturar a entrada e chamar o [[Use Case]].

```Java
// Exemplo via Console (CLI) atuando como Entry Adapter
public class ConsoleEntryAdapter {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.println("Digite o nome:");
        String nome = scanner.nextLine();
        
        // O Adapter de entrada prepara os dados e chama o Use Case
        CriarClienteUseCase useCase = new CriarClienteUseCase(repository);
        useCase.executar(nome, "email@teste.com");
    }
}
```

---

## 5. Funcionamento Dentro do Framework (Spring Boot)

No Spring, o Controller assume o papel de Adapter. Ele usa anotações para mapear o protocolo HTTP.

```Java
@RestController // Define que esta classe é um Entry Adapter Web
@RequestMapping("/clientes")
public class ClienteController {
    
    private final CriarClienteUseCase useCase;

    public ClienteController(CriarClienteUseCase useCase) {
        this.useCase = useCase;
    }

    @PostMapping
    public ResponseEntity<Void> criar(@RequestBody CriarClienteRequest request) {
        // 1. O Adapter recebe o DTO (Request)
        // 2. O Adapter chama o Use Case (Orquestrador)
        useCase.executar(request.getNome(), request.getEmail());
        
        // 3. O Adapter define a resposta do protocolo (HTTP 201)
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

---

## 6. DTOs: Os Mensageiros da API

Na pasta `api/request` e `api/response`, usamos **DTOs (Data Transfer Objects)**.

Eles são classes simples que definem exatamente o que entra e o que sai da API.

- **Request DTO:** O que o cliente envia (pode ter anotações de validação como `@NotBlank`).
- **Response DTO:** O que o sistema devolve (evita expor dados sensíveis como senhas ou IDs internos).

---

## 7. Por que separar a API do [[Domínio]]?

1. **Versatilidade:** Você pode ter um `WebController` e um `CliController` chamando o **mesmo** [[Use Case]].
2. **Segurança:** Se você usar sua classe de [[Domínio]] ou de Banco diretamente na API, um usuário mal-intencionado pode tentar alterar campos que não deveria (ex: mudar o `id` via JSON).
3. **Contrato Estável:** Você pode mudar o nome de um campo no Banco de Dados ou no Domínio sem quebrar a API que o seu Front-end consome, pois o DTO da API continua o mesmo.

---

## Notas de Revisão

- **Localização:** No seu projeto, isso fica em `api/controller`, `api/request` e `api/response`.
- **Fluxo:** HTTP Request -> Controller -> DTO -> Use Case.
- **Isolamento:** O Controller nunca deve injetar um Repository. Ele sempre deve falar com um **Use Case**.