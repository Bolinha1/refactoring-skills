# SKILL: Detecção e Refatoração de Long Method — PHP

## Fonte
Baseado em: https://refactoring.guru/smells/long-method

---

## 1. O que é Long Method

Um método que contém linhas de código em excesso.
Como regra geral: qualquer método com mais de 10 linhas já merece atenção.

**Por que isso acontece:**
- Sempre se adiciona lógica ao método, nunca se remove
- É mentalmente mais fácil adicionar duas linhas do que criar um novo método
- O problema cresce silenciosamente até virar código espaguete

---

## 2. Sinais de alerta (gatilhos para acionar este SKILL)

- [ ] Método com mais de 10 linhas
- [ ] Bloco de código que você sentiu vontade de comentar
- [ ] Condicional complexa (if/else aninhado)
- [ ] Loop com lógica não trivial dentro
- [ ] Variáveis temporárias espalhadas pelo método
- [ ] Método que faz mais de uma coisa claramente distinta

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                        | Técnica indicada                  |
|--------------------------------------------|-----------------------------------|
| Trecho de código comentado ou comentável   | Extract Method                    |
| Variáveis locais impedem a extração        | Replace Temp with Query           |
| Muitos parâmetros no método extraído       | Introduce Parameter Object        |
| Objeto inteiro sendo desmembrado           | Preserve Whole Object             |
| Nenhuma das anteriores resolve             | Replace Method with Method Object |
| Condicional complexa                       | Decompose Conditional             |
| Loop com lógica interna complexa           | Extract Method                    |

**Regra de ouro:** Se você sentiu vontade de escrever um comentário dentro do método,
esse trecho deve virar um método com nome descritivo. O nome substitui o comentário.

---

## 4. Exemplo

**ANTES — não aceito:**
```php
public function processarPedido(Pedido $pedido): void
{
    // validar pedido
    if (empty($pedido->getItens())) {
        throw new PedidoInvalidoException("Pedido sem itens");
    }
    if ($pedido->getCliente() === null) {
        throw new PedidoInvalidoException("Cliente não informado");
    }

    // calcular total
    $total = 0;
    foreach ($pedido->getItens() as $item) {
        $total += $item->getPreco() * $item->getQuantidade();
    }
    if ($pedido->temDesconto()) {
        $total = $total * (1 - $pedido->getDesconto());
    }

    // persistir
    $this->pedidoRepository->save($pedido);
    $this->emailService->enviarConfirmacao($pedido->getCliente()->getEmail());
}
```

**DEPOIS — esperado:**
```php
public function processarPedido(Pedido $pedido): void
{
    $this->validarPedido($pedido);
    $total = $this->calcularTotal($pedido);
    $pedido->setTotal($total);
    $this->confirmarPedido($pedido);
}

private function validarPedido(Pedido $pedido): void
{
    if (empty($pedido->getItens())) {
        throw new PedidoInvalidoException("Pedido sem itens");
    }
    if ($pedido->getCliente() === null) {
        throw new PedidoInvalidoException("Cliente não informado");
    }
}

private function calcularTotal(Pedido $pedido): float
{
    $total = array_reduce($pedido->getItens(), function (float $carry, Item $item) {
        return $carry + $item->getPreco() * $item->getQuantidade();
    }, 0.0);

    return $pedido->temDesconto() ? $total * (1 - $pedido->getDesconto()) : $total;
}

private function confirmarPedido(Pedido $pedido): void
{
    $this->pedidoRepository->save($pedido);
    $this->emailService->enviarConfirmacao($pedido->getCliente()->getEmail());
}
```

**Por que esse padrão:**
- Cada método privado tem nome que expressa intenção de negócio
- O método principal virou uma narrativa legível
- Nenhum comentário necessário — o nome substitui

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair sem nomear com intenção**
```php
// Não aceito — nome não diz o que faz
private function fazAlgumaCoisa(Pedido $pedido): void { ... }
private function metodo1(Pedido $pedido): void { ... }
private function processarParte2(Pedido $pedido): void { ... }
```

**Erro 2: Comentar em vez de extrair**
```php
// Não aceito — comentário indica que devia ser um método
// valida os dados do cliente
if ($cliente === null || $cliente->getCpf() === null) { ... }
```

**Erro 3: Extrair e deixar método ainda longo**
```php
// Não aceito — extraiu um trecho mas o método original
// ainda tem 60 linhas com múltiplas responsabilidades
```

---

## 6. Benefícios

- **Longevidade:** Classes com métodos curtos vivem mais
- **Manutenção:** Reduz a dificuldade de entender e manter o código
- **Qualidade:** Previne código duplicado escondido em métodos longos
- **Performance:** Impacto negativo desprezível; habilita otimizações melhores
- **Clareza:** Código claro e compreensível facilita identificar melhorias reais de performance
