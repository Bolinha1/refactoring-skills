# TÉCNICA: Extrair Variável — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/extract-variable

---

## 1. Problema

Uma expressão complexa ou difícil de ler está embutida diretamente em uma condição, retorno ou cálculo. Sem um nome, o leitor precisa decodificar a expressão para entender a intenção do código.

---

## 2. Solução

Atribua a expressão a uma variável local com nome que expresse sua intenção. Use a variável no lugar da expressão original.

---

## 3. Quando aplicar

- Uma expressão condicional é longa ou combina múltiplas verificações
- O resultado de um cálculo intermediário não tem nome
- A mesma subexpressão aparece mais de uma vez na mesma função
- Adicionar um comentário seria a única forma de explicar o que a expressão faz

---

## 4. Passos de refatoração

1. Certifique-se de que a expressão não tem efeitos colaterais
2. Declare uma nova variável com nome que expresse o propósito da expressão
3. Atribua a expressão à nova variável
4. Substitua a expressão original pela variável
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func calcularDesconto(pedido *Pedido) float64 {
    if pedido.Quantidade > 100 && pedido.PrecoUnitario > 50.0 && !pedido.EhUrgente {
        return pedido.Quantidade * pedido.PrecoUnitario * 0.10
    }
    return 0
}
```

**DEPOIS — esperado:**
```go
func calcularDesconto(pedido *Pedido) float64 {
    quantidadeAlta := pedido.Quantidade > 100
    precoAlto := pedido.PrecoUnitario > 50.0
    pedidoNaoUrgente := !pedido.EhUrgente
    elegívelParaDesconto := quantidadeAlta && precoAlto && pedidoNaoUrgente

    if elegívelParaDesconto {
        return pedido.Quantidade * pedido.PrecoUnitario * 0.10
    }
    return 0
}
```

**Por que esse padrão:**
- `elegivelParaDesconto` nomeia a regra de negócio sem exigir que o leitor avalie cada parte
- Cada variável intermediária documenta o que cada subcondição verifica

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Nome que repete a expressão sem acrescentar significado**
```go
// Não aceito — o nome não explica a intenção
quantidadeMaiorQue100 := pedido.Quantidade > 100
```

**Erro 2: Extrair expressão com efeito colateral**
```go
// Não aceito — chamado duas vezes pode causar comportamento inesperado
proximoID := gerarProximoID() // efeito colateral: incrementa contador
```

---

## 7. Benefícios

- **Legibilidade:** Expressões complexas ganham nomes que comunicam intenção
- **Depuração:** Variáveis nomeadas podem ser inspecionadas no debugger
- **Documentação:** Reduz necessidade de comentários explicativos inline
