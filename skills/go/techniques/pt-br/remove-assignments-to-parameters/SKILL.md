# TÉCNICA: Remover Atribuições a Parâmetros — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/remove-assignments-to-parameters

---

## 1. Problema

Em Go, parâmetros são passados por valor — reatribuir um parâmetro dentro de uma função não afeta o chamador, mas confunde o leitor que pode pensar que há um efeito externo. Além disso, reatribuir parâmetros mistura dois conceitos distintos: o valor de entrada e o valor de trabalho.

---

## 2. Solução

Use uma variável local separada para o valor modificado. Mantenha o parâmetro como somente leitura ao longo da função.

---

## 3. Quando aplicar

- O parâmetro é reatribuído dentro da função (não apenas modificado via ponteiro)
- O parâmetro serve tanto como valor de entrada quanto como variável de trabalho
- A reatribuição ocorre longe da declaração do parâmetro, dificultando rastreamento
- A leitura do código sugere erroneamente que o chamador será afetado

---

## 4. Passos de refatoração

1. Crie uma variável local com nome descritivo e atribua a ela o valor inicial do parâmetro (ou a expressão que seria atribuída ao parâmetro)
2. Substitua todas as referências ao parâmetro dentro do corpo da função pela nova variável local
3. Use o parâmetro apenas para leitura do valor original
4. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func calcularDesconto(preco float64, quantidade int) float64 {
    if quantidade > 100 {
        preco *= 0.9 // reatribuindo o parâmetro
    }
    return preco
}
```

**DEPOIS — esperado:**
```go
func calcularDesconto(preco float64, quantidade int) float64 {
    precoFinal := preco // variável local separada
    if quantidade > 100 {
        precoFinal = preco * 0.9
    }
    return precoFinal
}
```

**Variante com ponteiro — quando a modificação é intencional:**
```go
// Se o objetivo é modificar o valor apontado, use ponteiro explicitamente
func aplicarDesconto(preco *float64, quantidade int) {
    if quantidade > 100 {
        *preco *= 0.9 // modificação intencional via ponteiro — aceitável
    }
}
```

**Por que esse padrão:**
- `precoFinal` deixa claro que é o resultado calculado, não o valor de entrada
- O parâmetro `preco` permanece imutável, refletindo fielmente o que foi passado

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Reatribuir o parâmetro ponteiro em si (não o valor apontado)**
```go
// Não aceito — reatribuir o ponteiro não afeta o chamador; use variável local
func processar(pedido *Pedido) {
    if pedido == nil {
        pedido = &Pedido{} // reatribuição do ponteiro — sem efeito externo
    }
}
```

**Erro 2: Usar o parâmetro original e a variável local de forma intercambiável**
```go
// Não aceito — misturar preco e precoFinal no mesmo corpo cria confusão
func calcularDesconto(preco float64, quantidade int) float64 {
    precoFinal := preco
    if quantidade > 100 {
        precoFinal = preco * 0.9
    }
    return preco + precoFinal // mistura os dois — qual é o correto?
}
```

---

## 7. Benefícios

- **Clareza:** Distingue o valor de entrada do valor de trabalho
- **Imutabilidade local:** Parâmetros permanecem como referência ao valor original
- **Rastreabilidade:** Mais fácil entender o fluxo de transformação dos dados
