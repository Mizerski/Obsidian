## 1. O que é um Use Case?

Um **Use Case** (ou Caso de Uso) representa uma ação específica que um usuário ou sistema externo pode realizar na sua aplicação. Se o seu sistema fosse um restaurante, o Use Case seria a **receita**: ele descreve o passo a passo para transformar ingredientes (dados) em um prato (resultado).

Ele é frequentemente chamado de **Application Service** (Serviço de Aplicação) porque sua função é orquestrar a lógica necessária para realizar uma tarefa de negócio.

---

## 2. O Papel do Orquestrador

O Use Case não executa cálculos matemáticos complexos (isso fica no [[Domínio]]/Model) nem lida com protocolos HTTP (isso fica no Controller). Ele **orquestra**:

1. Recebe os dados de entrada.
2. Busca o que for necessário (no banco de dados ou outras fontes).
3. Chama as regras de negócio nos objetos de domínio.
4. Manda salvar o resultado.
5. Retorna uma resposta ou confirmação.

---

## 3. Funcionamento em Java Puro ([[POJO]])

Em Java puro, um Use Case é apenas uma classe comum com um método público (geralmente chamado de `execute`, `handle` ou o próprio nome da ação).

```Java
public class CancelarPedidoUseCase {
    private final PedidoRepository repository;
    private final EmailService emailService;

    public CancelarPedidoUseCase(PedidoRepository repository, EmailService emailService) {
        this.repository = repository;
        this.emailService = emailService;
    }

    public void executar(Long pedidoId) {
        // 1. Busca os dados
        Pedido pedido = repository.buscarPorId(pedidoId);
        
        // 2. Orquestra a regra (o pedido sabe se pode ser cancelado)
        pedido.cancelar();
        
        // 3. Persiste a mudança
        repository.salvar(pedido);
        
        // 4. Aciona efeitos colaterais
        emailService.notificarCancelamento(pedido.getClienteEmail());
    }
}
```

---

## 4. Funcionamento no Spring Framework

No Spring, o Use Case é transformado em um **[[Bean]]** para que possa ser injetado onde for necessário (como em um Controller).

- **Anotação:** Geralmente usa-se `@Service`.
- **Injeção:** O Spring entrega as dependências (Repositories, outros Services) automaticamente no construtor.

```Java
@Service
public class CancelarPedidoUseCase {
    private final PedidoRepository repository;

    public CancelarPedidoUseCase(PedidoRepository repository) {
        this.repository = repository;
    }

    public void executar(Long id) {
        // Lógica de orquestração aqui...
    }
}
```

---

## 5. Comparação de Responsabilidades

| **Camada**            | **O que faz?**                  | **O que NÃO deve fazer?**                           |
| --------------------- | ------------------------------- | --------------------------------------------------- |
| **Controller**        | Traduz HTTP e valida JSON.      | Não contém regras de negócio ou cálculos.           |
| **Use Case**          | Orquestra o fluxo da tarefa.    | Não sabe se os dados vieram de um JSON ou do banco. |
| **[[Domínio]]/Model** | Contém a regra de negócio pura. | Não chama repositórios ou envia e-mails.            |
| **Repository**        | Salva e busca dados.            | Não decide se um pedido pode ser cancelado.         |

---

## 6. Por que não colocar tudo no Controller?

Se você colocar a lógica no Controller (**Fat Controller**), você terá problemas:

- **Código Duplicado:** Se precisar da mesma ação em uma tarefa agendada (Cron) ou via Mensageria, terá que copiar o código do Controller.
- **Dificuldade de Teste:** Testar um Controller exige simular requisições HTTP, o que é lento e complexo. Testar um Use Case é testar uma classe Java simples.

---

## Notas de Revisão

- **Um método por Use Case:** Uma boa prática é que cada classe de Use Case tenha apenas **um método público**. Isso respeita o Princípio da Responsabilidade Única (SRP).
- **Independência:** O Use Case deve ser escrito de forma que, se você trocar o Spring pelo Quarkus ou o REST por GraphQL, a lógica dentro do `executar()` não mude.
- **Input/Output:** Use classes simples (DTOs) para enviar dados para o Use Case e receber resultados dele. Isso evita que o Controller tenha que lidar com entidades complexas.