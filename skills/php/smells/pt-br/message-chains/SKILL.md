# SMELL: Message Chains — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/message-chains

---

## 1. O que é?

Um cliente pede a um objeto outro objeto, que pede a outro objeto ainda, formando uma cadeia: `$a->getB()->getC()->getD()->fazerAlgo()`. O cliente está acoplado a toda a cadeia de intermediários. Se qualquer passo mudar sua estrutura, o cliente quebra.

---

## 2. Sinais de alerta

- [ ] Chamadas de método são encadeadas três ou mais níveis de profundidade: `$pedido->getCliente()->getEndereco()->getCidade()`
- [ ] A mesma cadeia aparece em múltiplos lugares no código
- [ ] É necessário navegar por uma cadeia de objetos apenas para obter um único valor
- [ ] Uma cadeia aparece em um loop ou condicional, amplificando o acoplamento
- [ ] Adicionar uma camada entre dois objetos na cadeia requer atualizar cada ponto de chamada

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Hide Delegate** | Adicione um método de atalho no primeiro objeto da cadeia para que clientes não precisem navegar pelos intermediários |
| **Extract Method** | Extraia a cadeia para um método nomeado que oculte a navegação |

---

## 4. Exemplo

**ANTES — não aceito:**
```php
// Cliente navega por todo o grafo de objetos apenas para obter um nome de cidade
class ServicoFatura {
    public function getCidadeEntrega(Pedido $pedido): string {
        return $pedido->getCliente()->getEndereco()->getCidade();
    }

    public function isEntregaLocal(Pedido $pedido): bool {
        return $pedido->getCliente()->getEndereco()->getCidade() === 'São Paulo';
    }
}
```

**DEPOIS — esperado:**
```php
// Adicionar um atalho em Pedido (Hide Delegate)
class Pedido {
    public function getCidadeEntrega(): string {
        return $this->cliente->getEndereco()->getCidade();
    }
}

// Cliente não conhece mais Cliente nem Endereco
class ServicoFatura {
    public function getCidadeEntrega(Pedido $pedido): string {
        return $pedido->getCidadeEntrega();
    }

    public function isEntregaLocal(Pedido $pedido): bool {
        return $pedido->getCidadeEntrega() === 'São Paulo';
    }
}
```

**Por que este padrão:**
- `ServicoFatura` está desacoplado de `Cliente` e `Endereco`
- Renomear `Endereco::getCidade()` requer apenas uma mudança em vez de muitas

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Atribuir a cadeia a uma variável local sem quebrar a dependência**
```php
// Não aceito — o acoplamento ainda está lá; apenas a sintaxe mudou
$endereco = $pedido->getCliente()->getEndereco();
$cidade = $endereco->getCidade();
```

**Erro 2: Adicionar um método que retorna o objeto intermediário**
```php
// Não aceito — agora os chamadores encadeiam no objeto retornado; mesmo problema
public function getCliente(): Cliente { return $this->cliente; }
```

---

## 6. Benefícios

- **Menor acoplamento:** Clientes dependem apenas de seu colaborador direto
- **Refatoração mais fácil:** A navegação interna pode ser alterada sem atualizar cada ponto de chamada
- **Mais legível:** `$pedido->getCidadeEntrega()` é mais claro que `$pedido->getCliente()->getEndereco()->getCidade()`
