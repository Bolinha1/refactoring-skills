# TECHNIQUE: Inline Method — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/inline-method

---

## 1. Problema

O corpo de um método é tão óbvio quanto seu nome, ou o método é usado apenas uma vez e a indireção não agrega valor. Manter um wrapper trivial em torno de uma única expressão torna o código mais difícil de seguir, não mais fácil.

---

## 2. Solução

Substitua cada chamada ao método pelo corpo do método. Delete o método.

---

## 3. Quando aplicar

- O corpo do método é uma única linha e o nome não acrescenta nada além de reformular o código
- O método é chamado em apenas um lugar e a indireção prejudica a legibilidade
- Uma sequência de métodos curtos foi criada durante a refatoração e a estrutura ficou super fragmentada
- O método era um shim de delegação cujo delegado foi incorporado ou removido

---

## 4. Passos de refatoração

1. Verifique que o método não é sobrescrito em uma subclasse
2. Encontre todos os pontos de chamada
3. Para cada ponto de chamada, substitua a chamada pelo corpo do método (substituindo parâmetros por argumentos)
4. Delete o método
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
class ServicoPedido {
    public function getDesconto(Pedido $pedido): float {
        return $this->maisDeCinquentaItens($pedido) ? 0.1 : 0.0;
    }

    private function maisDeCinquentaItens(Pedido $pedido): bool {
        return $pedido->getQuantidadeItens() > 50;
    }
}
```

**DEPOIS — esperado:**
```php
class ServicoPedido {
    public function getDesconto(Pedido $pedido): float {
        return $pedido->getQuantidadeItens() > 50 ? 0.1 : 0.0;
    }
}
```

**Por que este padrão:**
- `maisDeCinquentaItens` não adicionava clareza — a condição é legível por si só
- Um método a menos para pesquisar, testar e manter

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Fazer inline de um método chamado em múltiplos lugares**
```php
// Não aceito — fazer inline duplica a condição em cada ponto de chamada
if ($this->maisDeCinquentaItens($pedido)) { ... } // chamado aqui e em outros três lugares
```

**Erro 2: Fazer inline de um método sobrescrito em uma subclasse**
```php
// Não aceito — fazer inline remove o hook polimórfico usado por subclasses
protected function maisDeCinquentaItens(Pedido $pedido): bool { ... }
```

**Erro 3: Fazer inline de um método cujo nome explica um invariante não óbvio**
```php
// Não aceito — o nome carrega significado de domínio que a expressão bruta não transmite
private function elegivelParaFreteGratis(Pedido $pedido): bool {
    return $pedido->getTotal() > 100;
}
```

---

## 7. Benefícios

- **Indireção reduzida:** Não é necessário pular para outro método para entender uma linha
- **Código mais simples:** Remove métodos que existem puramente como andaimes remanescentes
- **API pública mais limpa:** Menos métodos públicos expõem menos superfície
