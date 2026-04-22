# TÉCNICA: Internalizar Método — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/inline-method

---

## 1. Problema

O corpo de uma função é tão óbvio quanto seu próprio nome. A função não agrega legibilidade — ela apenas adiciona uma camada de indireção sem valor, tornando o código mais difícil de navegar.

---

## 2. Solução

Substitua as chamadas à função pelo corpo dela. Delete a função original.

---

## 3. Quando aplicar

- O corpo da função é igualmente claro ao seu nome
- A função foi criada apenas para intermediar outra função e não adiciona lógica
- Há excesso de delegações triviais entre funções (Middle Man excessivo)
- Você está consolidando lógica antes de uma refatoração maior

---

## 4. Passos de refatoração

1. Verifique que a função não é sobrescrita (em Go: não implementa uma interface de forma polimórfica)
2. Localize todas as chamadas à função
3. Substitua cada chamada pelo corpo da função
4. Execute os testes
5. Delete a função original

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func (p *Pedido) ObterNivelRating() int {
    return p.maisQueCincoPedidosAtrasados()
}

func (p *Pedido) maisQueCincoPedidosAtrasados() int {
    if p.PedidosAtrasados > 5 {
        return 2
    }
    return 1
}
```

**DEPOIS — esperado:**
```go
func (p *Pedido) ObterNivelRating() int {
    if p.PedidosAtrasados > 5 {
        return 2
    }
    return 1
}
```

**Por que esse padrão:**
- `maisQueCincoPedidosAtrasados` era uma indireção sem valor semântico adicional
- O leitor não precisa navegar até outra função para entender a lógica

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Internalizar uma função que é reutilizada em vários lugares**
```go
// Não aceito — se calcularDesconto é chamado em 5 lugares,
// internalizá-lo duplica a lógica e dificulta manutenção futura
func calcularDesconto(pedido *Pedido) float64 { ... }
```

**Erro 2: Internalizar uma função que implementa uma interface**
```go
// Não aceito — se a função é parte de uma interface Go,
// removê-la quebra o contrato de interface
type Processador interface {
    Processar() error
}
```

---

## 7. Benefícios

- **Simplicidade:** Elimina camadas de indireção desnecessárias
- **Navegabilidade:** O leitor não precisa pular entre funções triviais
- **Clareza:** O código expressa diretamente o que faz, sem abstrações vazias
