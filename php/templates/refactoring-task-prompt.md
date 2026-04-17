# TEMPLATE: Prompt de Tarefa de Refatoração

## Como usar
Preencha os campos marcados com `[...]` e use o resultado como prompt
ao solicitar uma refatoração a um LLM ou descrever uma tarefa para um colega.

---

## Template

```
Preciso refatorar o seguinte trecho de código.

### Contexto
[Descreva brevemente o que esse código faz e onde ele se encontra no sistema.
Ex: "Método de processamento de pedidos na camada de serviço de um e-commerce."]

### Código atual
[Cole aqui o código que precisa ser refatorado]

### Smell identificado
[Informe o code smell detectado. Ex: Long Method, Large Class, Primitive Obsession]

### Técnica recomendada
[Informe a técnica de refatoração indicada. Ex: Extract Method, Replace Type Code with Enum]

### Restrições
[Liste o que NÃO pode mudar. Ex:]
- A assinatura pública do método não pode mudar (é usada por N chamadores)
- Não há testes automatizados — a refatoração deve ser conservadora
- A linguagem é [Java / Python / PHP] versão [X]
- [Outras restrições relevantes]

### Resultado esperado
[Descreva como o código deve ficar após a refatoração. Ex:]
- O método principal deve ter no máximo 10 linhas
- Cada responsabilidade deve estar em um método privado com nome que expresse intenção
- Nenhum comentário explicativo deve ser necessário
- Os testes existentes devem continuar passando
```

---

## Exemplos preenchidos

### Exemplo 1 — Long Method com Extract Method

```
Preciso refatorar o seguinte trecho de código.

### Contexto
Método `processarPedido` na classe `PedidoService` de um sistema de e-commerce em Java.
Responsável por validar, calcular total, reservar estoque, persistir e notificar o cliente.

### Código atual
public void processarPedido(Pedido pedido) {
    if (pedido.getItens().isEmpty()) throw new RuntimeException("Sem itens");
    if (pedido.getCliente() == null) throw new RuntimeException("Sem cliente");

    double total = 0;
    for (Item item : pedido.getItens()) {
        total += item.getPreco() * item.getQuantidade();
    }
    pedido.setTotal(total);

    estoqueService.reservar(pedido);
    repository.save(pedido);
    emailService.enviarConfirmacao(pedido.getCliente().getEmail());
}

### Smell identificado
Long Method — o método tem 14 linhas e faz 4 coisas distintas (validar, calcular, persistir, notificar).

### Técnica recomendada
Extract Method

### Restrições
- A assinatura `processarPedido(Pedido pedido)` não pode mudar
- Linguagem: Java 17
- Os testes de integração existentes devem continuar passando

### Resultado esperado
- O método principal deve ter no máximo 5 linhas
- Cada responsabilidade deve virar um método privado com nome de negócio
- Sem comentários — o nome do método substitui
```

---

### Exemplo 2 — Primitive Obsession com Replace Data Value with Object

```
Preciso refatorar o seguinte trecho de código.

### Contexto
Classe `Cliente` em um sistema de CRM em Python. O CPF é usado como `str` em todo o sistema
e sua validação está duplicada em 6 lugares diferentes.

### Código atual
class Cliente:
    def __init__(self, nome: str, cpf: str, email: str):
        self.nome = nome
        self.cpf = cpf
        self.email = email

### Smell identificado
Primitive Obsession — `cpf` e `email` são strings cruas sem validação centralizada.

### Técnica recomendada
Replace Data Value with Object

### Restrições
- Python 3.11
- Sem quebrar a API pública da classe Cliente
- Novos value objects devem ser imutáveis

### Resultado esperado
- `Cpf` como classe imutável com validação no construtor
- `Email` como classe imutável com validação no construtor
- Instâncias de `Cliente` usam os novos tipos
- Validação não se repete em nenhum outro lugar
```

---

## Dicas de uso

- Quanto mais específico o contexto, melhor a refatoração sugerida
- Sempre informe a linguagem e versão — idioms mudam entre versões
- Listar restrições evita sugestões que quebram contratos existentes
- Se não souber a técnica exata, descreva o problema e peça diagnóstico antes da refatoração
