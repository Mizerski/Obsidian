Perfeito! Vou explicar de forma bem simples para você. 😊

## O Novo Sistema de Input da Unity (Input System)

Você está usando o **Input System**, que é o sistema moderno da Unity. Pense em **input** como qualquer ação do jogador (apertar tecla, clicar, controle do joystick, etc).

### Os Conceitos Principais:

**1. `PlayerInputActions` - O "Mapa de Controles"**
```
Pense como se fosse um MANUAL de instruções que você criou.
Diz: "Quando o jogador apertar WASD, vou chamar a ação Move"
```

**2. `OnEnable()` e `OnDisable()` - Ligando/Desligando o Controle**
```
OnEnable()  = "Ei, vamos ATIVAR o mapa de controles!"
OnDisable() = "Desligamos o mapa de controles"

É como plugar/despluguar um controle da TV
```

**3. `OnMove()` - O Que Fazer Quando o Jogador Mexe**
```
Cada vez que o jogador aperta uma tecla de movimento, 
essa função é chamada e GUARDA a direção que ele quer ir
```

### Como Funciona na Prática:

```
Jogador aperta WASD
        ↓
"Opa! Isso é movimento!" (sistema reconhece)
        ↓
OnMove() é chamado
        ↓
input_mover recebe a direção (ex: cima = [0, 1])
        ↓
Update() pega esse valor e move o personagem
```

### No `Update()`:**

```csharp
transform.position += (Vector3)(velocidade * Time.deltaTime * input_mover);
```

- `transform.position` = posição do objeto no mapa
- `velocidade` = quão rápido anda
- `Time.deltaTime` = tempo desde o último frame (faz ficar smooth)
- `input_mover` = para onde o jogador quer ir

# Codigo referenciado:

```c#
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerMovement2D : MonoBehaviour
{
    public float velocidade = 1f;

    private PlayerInputActions acoes_de_input;
    private Vector2 input_mover;

    public Vector2 MoverInput => input_mover;

    private void Awake()
    {
        acoes_de_input = new PlayerInputActions();
    }

    private void OnEnable()
    {
        acoes_de_input.Player.Enable();
        acoes_de_input.Player.Move.performed += OnMove;
        acoes_de_input.Player.Move.canceled += OnMove;
    }

    private void OnDisable()
    {
        acoes_de_input.Player.Move.performed -= OnMove;
        acoes_de_input.Player.Move.canceled -= OnMove;
        acoes_de_input.Player.Disable();
    }

    private void OnMove(InputAction.CallbackContext context)
    {
        input_mover = context.ReadValue<Vector2>();
    }

    private void Update()
    {
        transform.position += (Vector3)(velocidade * Time.deltaTime * input_mover);
    }
}

```