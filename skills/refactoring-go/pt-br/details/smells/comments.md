# SKILL: Detectando e Refatorando Comentários Excessivos — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/comments

---

## 1. O que é o smell de Comentários

Comentários que explicam *o que* o código faz em vez de *por que* uma decisão foi tomada. Quando o código precisa de um comentário para ser entendido, geralmente significa que ele precisa ser refatorado para ser autoexplicativo. Comentários compensatórios mascaram código confuso em vez de corrigi-lo.

**Por que isso acontece:**
- O código foi escrito de forma apressada e comentários foram adicionados como desculpa
- Nomes de variáveis e funções são genéricos demais e precisam de explicação extra
- Algoritmos complexos foram colocados em um único bloco sem decomposição

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Comentário que apenas repete o que o código faz (`// incrementa contador`)
- [ ] Bloco de código precedido por um comentário descritivo longo
- [ ] Comentários `// TODO` ou `// FIXME` que existem há muito tempo
- [ ] Código comentado que nunca foi removido
- [ ] Comentário necessário para entender uma condição complexa (`if x > 0 && y < 10 // verifica se está no intervalo válido`)

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Bloco de código com comentário explicativo | Extract Function com nome descritivo |
| Condição complexa comentada | Introduce Explaining Variable |
| Comentário descreve o propósito de um grupo de linhas | Extract Function |
| Comentário justifica uma decisão de design | Manter — comentários de *por que* são válidos |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
func processarPedido(pedido *Pedido) float64 {
    // verifica se o pedido tem itens e se o cliente existe
    if len(pedido.Itens) == 0 || pedido.Cliente == nil {
        return 0
    }

    // calcula o subtotal somando preço * quantidade de cada item
    subtotal := 0.0
    for _, item := range pedido.Itens {
        subtotal += item.Preco * float64(item.Quantidade)
    }

    // aplica desconto de 10% se o pedido for maior que 500
    desconto := 0.0
    if subtotal > 500 {
        desconto = subtotal * 0.10
    }

    return subtotal - desconto
}
```

**DEPOIS — esperado:**
```go
func processarPedido(pedido *Pedido) float64 {
    if !pedidoValido(pedido) {
        return 0
    }
    subtotal := calcularSubtotal(pedido.Itens)
    desconto := calcularDesconto(subtotal)
    return subtotal - desconto
}

func pedidoValido(pedido *Pedido) bool {
    return len(pedido.Itens) > 0 && pedido.Cliente != nil
}

func calcularSubtotal(itens []ItemPedido) float64 {
    total := 0.0
    for _, item := range itens {
        total += item.Preco * float64(item.Quantidade)
    }
    return total
}

func calcularDesconto(subtotal float64) float64 {
    const limiteDesconto = 500.0
    const taxaDesconto = 0.10
    if subtotal > limiteDesconto {
        return subtotal * taxaDesconto
    }
    return 0
}
```

**Por que este padrão:**
- Nomes de função descrevem a intenção — comentários se tornam redundantes
- Constantes nomeadas (`limiteDesconto`, `taxaDesconto`) eliminam números mágicos

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Apenas remover o comentário sem refatorar o código**
```go
// Não aceito — código ainda é confuso, só perdeu a documentação
if len(pedido.Itens) == 0 || pedido.Cliente == nil {
    return 0
}
```

**Erro 2: Transformar comentário em nome de variável sem extrair função**
```go
// Não aceito — variável explica mas a função ainda é longa demais
pedidoInvalido := len(pedido.Itens) == 0 || pedido.Cliente == nil
```

---

## 6. Benefícios

- **Legibilidade:** Código autoexplicativo é mais fácil de ler do que código com comentários
- **Manutenção:** Comentários ficam desatualizados; nomes de função não mentem
- **Descobribilidade:** Funções extraídas podem ser reutilizadas em outros contextos
