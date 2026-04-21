# SMELL: Inappropriate Intimacy — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/inappropriate-intimacy

---

## 1. O que é?

Uma classe acessa as propriedades privadas, arrays internos ou detalhes de implementação de outra classe de forma excessiva. As duas classes estão fortemente acopladas de um modo que dificulta a mudança de uma sem afetar a outra.

---

## 2. Sinais de alerta

- [ ] Uma classe chama múltiplos getters de outra classe para realizar um cálculo que deveria viver nessa outra classe
- [ ] Uma classe muta diretamente o array ou estado interno de outra classe
- [ ] Duas classes se referenciam mutuamente em uma dependência bidirecional
- [ ] Uma classe conhece a estrutura interna de outra
- [ ] "Enfileiramentos de trens" repetidos: `$a->getB()->getC()->fazerAlgo()`

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Move Method** | Mova o método que acessa internos alheios para a classe que possui esses internos |
| **Move Field** | Mova uma propriedade para a classe que mais a utiliza |
| **Hide Delegate** | Adicione um método de encaminhamento para que o cliente não precise acessar através de um intermediário |
| **Extract Class** | Se duas classes são muito íntimas porque compartilham responsabilidades, separe-as de forma limpa |

---

## 4. Exemplo

**ANTES — não aceito:**
```php
class Pedido {
    private array $itens = [];
    private string $emailCliente = '';

    public function getItens(): array { return $this->itens; } // expõe internos
    public function getEmailCliente(): string { return $this->emailCliente; }
}

// Relatorio vasculha os internos de Pedido
class RelatorioPedido {
    public function gerar(Pedido $pedido): string {
        $total = 0.0;
        foreach ($pedido->getItens() as $item) {  // acessando internos de Pedido
            $total += $item->getPreco() * $item->getQuantidade();
        }
        return 'Pedido para ' . $pedido->getEmailCliente() . ' total: ' . $total;
    }
}
```

**DEPOIS — esperado:**
```php
class Pedido {
    public function __construct(
        private readonly array $itens,
        private readonly string $emailCliente
    ) {}

    public function total(): float {
        return array_reduce(
            $this->itens,
            fn(float $soma, ItemPedido $item) => $soma + $item->getPreco() * $item->getQuantidade(),
            0.0
        );
    }

    public function getEmailCliente(): string { return $this->emailCliente; }
}

// Relatorio apenas pergunta ao Pedido o que precisa
class RelatorioPedido {
    public function gerar(Pedido $pedido): string {
        return 'Pedido para ' . $pedido->getEmailCliente() . ' total: ' . $pedido->total();
    }
}
```

**Por que este padrão:**
- `RelatorioPedido` não depende mais da estrutura interna de `Pedido`
- Mudar como `Pedido` armazena itens não afeta `RelatorioPedido`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Substituir o loop por array_reduce mas ainda puxando o array**
```php
// Não aceito — getItens() ainda expõe os internos
$total = array_reduce($pedido->getItens(), fn($soma, $item) => $soma + $item->getPreco() * $item->getQuantidade(), 0.0);
```

**Erro 2: Criar um DTO que copia todas as propriedades de Pedido**
```php
// Não aceito — um PedidoDTO com as mesmas propriedades é apenas indireção; o acoplamento permanece
```

---

## 6. Benefícios

- **Menor acoplamento:** Classes podem mudar seus internos sem afetar seus clientes
- **Melhor encapsulamento:** A classe proprietária controla como seus dados são calculados e acessados
- **Mais fácil de testar:** Classes com menos dependências são mais simples de testar em isolamento
