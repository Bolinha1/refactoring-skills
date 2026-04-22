# SMELL: Data Class — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/data-class

---

## 1. O que é?

Uma classe que contém apenas propriedades, getters e setters — mas nenhum comportamento real. Toda a lógica que opera sobre esses dados vive em outras classes. A classe é essencialmente um contêiner de dados passivo e age como um registro sem responsabilidade.

---

## 2. Sinais de alerta

- [ ] Uma classe possui apenas propriedades, getters e setters (e talvez um construtor)
- [ ] A lógica de negócios que deveria pertencer à classe está espalhada por classes de serviço ou gerenciador
- [ ] Outras classes manipulam as propriedades da classe de dados diretamente através de getters/setters
- [ ] A classe é usada apenas como um saco de parâmetros passado entre métodos
- [ ] Nenhum método existe além de acessores triviais

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Move Method** | Mova o comportamento das classes de serviço para a classe de dados, dando-lhe responsabilidade |
| **Extract Class** | Se a classe de dados cresceu muito, separe um subconjunto coesivo em sua própria classe com comportamento |
| **Hide Method** | Torne os setters privados se código externo não deveria modificar a propriedade diretamente |
| **Encapsulate Field** | Substitua o acesso direto à propriedade por getters/setters como primeiro passo para adicionar validação ou lógica |

---

## 4. Exemplo

**ANTES — não aceito:**
```php
// Contêiner de dados puro — sem comportamento
class Pedido {
    private array $itens = [];
    private string $status = '';

    public function getItens(): array { return $this->itens; }
    public function setItens(array $itens): void { $this->itens = $itens; }
    public function getStatus(): string { return $this->status; }
    public function setStatus(string $status): void { $this->status = $status; }
}

// Toda a lógica vive em um serviço externo
class ServicoPedido {
    public function calcularTotal(Pedido $pedido): float {
        return array_reduce(
            $pedido->getItens(),
            fn(float $soma, ItemPedido $item) => $soma + $item->getPreco() * $item->getQuantidade(),
            0.0
        );
    }

    public function podeCancelar(Pedido $pedido): bool {
        return $pedido->getStatus() === 'PENDENTE';
    }
}
```

**DEPOIS — esperado:**
```php
class Pedido {
    private string $status = 'PENDENTE';

    public function __construct(private readonly array $itens) {}

    public function total(): float {
        return array_reduce(
            $this->itens,
            fn(float $soma, ItemPedido $item) => $soma + $item->getPreco() * $item->getQuantidade(),
            0.0
        );
    }

    public function podeCancelar(): bool {
        return $this->status === 'PENDENTE';
    }

    public function cancelar(): void {
        if (!$this->podeCancelar()) {
            throw new \DomainException("Não é possível cancelar pedido com status: {$this->status}");
        }
        $this->status = 'CANCELADO';
    }
}
```

**Por que este padrão:**
- `Pedido` agora possui suas regras de negócio — chamadores perguntam ao próprio pedido
- A propriedade `status` é protegida: transições acontecem através de métodos de domínio

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover todos os métodos de serviço para a classe de dados sem discriminação**
```php
// Não aceito — a classe se torna uma Large Class; mova apenas comportamento coesivo
class Pedido {
    public function enviarEmailConfirmacao(): void { /* não relacionado ao domínio do Pedido */ }
    public function salvarNoBancoDados(): void { /* preocupação de infraestrutura */ }
}
```

**Erro 2: Manter setters públicos após adicionar lógica de domínio**
```php
// Não aceito — qualquer chamador pode contornar a regra de domínio chamando setStatus() diretamente
public function setStatus(string $status): void { $this->status = $status; }
```

---

## 6. Benefícios

- **Encapsulamento:** Regras de negócio vivem próximas às propriedades que governam
- **Redução de duplicação:** Lógica que estava copiada entre classes de serviço agora está em um único lugar
- **Diga, não pergunte:** Chamadores pedem ao objeto para fazer algo em vez de ler suas propriedades e decidir externamente
