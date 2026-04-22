# SKILL: Detectando e Refatorando Código Duplicado — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/duplicate-code

---

## 1. O que é Código Duplicado

Dois ou mais fragmentos de código são quase idênticos ou estruturalmente similares em partes diferentes da base de código. Qualquer mudança na lógica precisa ser replicada em cada cópia, e esquecer uma delas cria um bug silencioso.

**Por que isso acontece:**
- Copy-paste usado como atalho no lugar de abstração
- Desenvolvimento paralelo onde dois desenvolvedores resolveram o mesmo problema de forma independente
- Lógica que começou similar e divergiu ligeiramente, tornando a duplicação invisível

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Mesmo bloco de código (ou quase idêntico) em dois ou mais lugares
- [ ] Mesma computação ou expressão repetida em vários arquivos
- [ ] Dois pacotes contêm funções que fazem essencialmente a mesma coisa
- [ ] Você copiou e colou código e ajustou apenas uma variável

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Blocos duplicados na mesma função | Extract Function |
| Blocos duplicados em pacotes diferentes | Move Function para pacote compartilhado |
| Lógica similar mas não idêntica | Extract Function com parâmetro de variação |
| Algoritmos equivalentes com implementações distintas | Substitute Algorithm |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// relatorio_vendas.go
func calcularReceita(pedidos []Pedido) float64 {
    total := 0.0
    for _, pedido := range pedidos {
        if pedido.Status == StatusPago {
            total += pedido.Valor
        }
    }
    return total
}

// relatorio_financeiro.go
func calcularTotalPedidosPagos(pedidos []Pedido) float64 {
    total := 0.0
    for _, pedido := range pedidos {
        if pedido.Status == StatusPago {
            total += pedido.Valor
        }
    }
    return total
}
```

**DEPOIS — esperado:**
```go
// pedido/calculadora.go — lógica compartilhada em um único lugar
func SomarPedidosPagos(pedidos []Pedido) float64 {
    total := 0.0
    for _, pedido := range pedidos {
        if pedido.Status == StatusPago {
            total += pedido.Valor
        }
    }
    return total
}

// relatorio_vendas.go
func calcularReceita(pedidos []Pedido) float64 {
    return pedido.SomarPedidosPagos(pedidos)
}

// relatorio_financeiro.go
func calcularTotalPedidosPagos(pedidos []Pedido) float64 {
    return pedido.SomarPedidosPagos(pedidos)
}
```

**Por que este padrão:**
- A lógica de negócio vive em um único lugar — uma única mudança corrige todos os chamadores
- Ambas as funções de relatório delegam para o utilitário compartilhado

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Aceitar pequenas diferenças como justificativa para duplicação**
```go
// Não aceito — "quase igual" ainda significa duplicado
func calcularReceita(pedidos []Pedido) float64 { /* loop */ }
func calcularReceitaProjetada(pedidos []Pedido) float64 { /* mesmo loop, um multiplicador extra */ }
```

**Erro 2: Extrair para função privada mas duplicar a função em cada pacote**
```go
// Não aceito — a própria função privada está duplicada em dois pacotes
func somarPagos(pedidos []Pedido) float64 { ... }
```

**Erro 3: Fundir duplicatas em uma função com muitas flags booleanas**
```go
// Não aceito — cria Long Method com controle de fluxo complexo
func calcular(pedidos []Pedido, pago bool, projetado bool, comImpostos bool) float64 { ... }
```

---

## 6. Benefícios

- **Manutenção:** Uma única correção de bug ou mudança de regra se aplica em todos os lugares automaticamente
- **Clareza:** Código que expressa um conceito compartilhado é mais fácil de entender do que cópias espalhadas
- **Confiança:** Desenvolvedores podem ter certeza de que não há cópias divergentes escondendo lógica obsoleta
