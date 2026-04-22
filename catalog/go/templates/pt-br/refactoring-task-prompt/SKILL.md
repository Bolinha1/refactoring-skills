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
Ex: "Função de processamento de pedidos na camada de serviço de um e-commerce."]

### Código atual
[Cole aqui o código que precisa ser refatorado]

### Smell identificado
[Informe o code smell detectado. Ex: Long Method, Large Class, Primitive Obsession]

### Técnica recomendada
[Informe a técnica de refatoração indicada. Ex: Extract Method, Replace Type Code with Enum]

### Restrições
[Liste o que NÃO pode mudar. Ex:]
- A assinatura pública da função não pode mudar (é usada por N chamadores)
- Não há testes automatizados — a refatoração deve ser conservadora
- Linguagem: Go [versão]
- [Outras restrições relevantes]

### Resultado esperado
[Descreva como o código deve ficar após a refatoração. Ex:]
- A função principal deve ter no máximo 10 linhas
- Cada responsabilidade deve estar em uma função privada com nome que expresse intenção
- Nenhum comentário explicativo deve ser necessário
- Os testes existentes devem continuar passando
```

---

## Exemplos preenchidos

### Exemplo 1 — Long Method com Extract Method

```
Preciso refatorar o seguinte trecho de código.

### Contexto
Função `ProcessarPedido` na struct `PedidoService` de um sistema de e-commerce em Go.
Responsável por validar, calcular total, reservar estoque, persistir e notificar o cliente.

### Código atual
func (s *PedidoService) ProcessarPedido(pedido *Pedido) error {
    if len(pedido.Itens) == 0 {
        return fmt.Errorf("pedido sem itens")
    }
    if pedido.Cliente == nil {
        return fmt.Errorf("pedido sem cliente")
    }

    total := 0.0
    for _, item := range pedido.Itens {
        total += item.Preco * float64(item.Quantidade)
    }
    pedido.Total = total

    if err := s.estoque.Reservar(pedido); err != nil {
        return fmt.Errorf("reservando estoque: %w", err)
    }
    if err := s.repo.Salvar(pedido); err != nil {
        return fmt.Errorf("salvando pedido: %w", err)
    }
    return s.email.EnviarConfirmacao(pedido.Cliente.Email)
}

### Smell identificado
Long Method — a função tem mais de 20 linhas e faz 4 coisas distintas (validar, calcular, persistir, notificar).

### Técnica recomendada
Extract Method

### Restrições
- A assinatura `ProcessarPedido(pedido *Pedido) error` não pode mudar
- Linguagem: Go 1.22
- Os testes de integração existentes devem continuar passando

### Resultado esperado
- A função principal deve ter no máximo 5 linhas
- Cada responsabilidade deve virar um método privado com nome de negócio
- Sem comentários — o nome do método substitui
- Todos os erros devem continuar sendo encapsulados com contexto
```

---

### Exemplo 2 — Primitive Obsession com Replace Data Value with Object

```
Preciso refatorar o seguinte trecho de código.

### Contexto
Struct `Cliente` em um sistema de CRM em Go. O CPF é usado como `string` em todo o sistema
e sua validação está duplicada em 6 lugares diferentes.

### Código atual
type Cliente struct {
    Nome  string
    CPF   string
    Email string
}

### Smell identificado
Primitive Obsession — `CPF` e `Email` são strings cruas sem validação centralizada.

### Técnica recomendada
Replace Data Value with Object

### Restrições
- Go 1.22
- Sem quebrar a API pública da struct Cliente
- Novos tipos de valor devem ser imutáveis (sem setters exportados)

### Resultado esperado
- `CPF` como tipo imutável com validação em uma função construtora `NovoCPF`
- `Email` como tipo imutável com validação em uma função construtora `NovoEmail`
- Campos de `Cliente` usam os novos tipos
- Validação não se repete em nenhum outro lugar
```

---

## Dicas de uso

- Quanto mais específico o contexto, melhor a refatoração sugerida
- Sempre informe a versão do Go — a stdlib e os idioms evoluem entre versões
- Listar restrições evita sugestões que quebram contratos existentes
- Se não souber a técnica exata, descreva o problema e peça diagnóstico antes da refatoração
- Mencione se o código é concorrente — goroutines e channels mudam quais refatorações são seguras
