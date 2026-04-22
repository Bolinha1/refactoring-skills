# TÉCNICA: Decompor Condicional — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/decompose-conditional

---

## 1. Problema

Uma expressão condicional complexa (e o código em seus branches) torna difícil entender o que está sendo testado e o que acontece em cada caso. Ler a condição exige conhecimento de regras de negócio que não estão nomeadas em lugar algum.

---

## 2. Solução

Extraia a condição e cada branch para funções bem nomeadas. Os nomes das funções explicam a intenção; os corpos explicam a implementação.

---

## 3. Quando aplicar

- A condição é uma expressão booleana composta que requer reflexão para analisar
- O código em um dos branches tem várias linhas e merece um nome próprio
- O condicional aparece dentro de uma função já longa
- Ler a condição requer conhecimento de regras de domínio, não apenas lógica de programação

---

## 4. Passos de refatoração

1. Extraia a condição para uma função nomeada conforme a regra de negócio que verifica
2. Extraia o branch verdadeiro para uma função nomeada conforme o que faz
3. Extraia o branch falso para uma função nomeada conforme o que faz
4. Substitua o `if` original pelas chamadas às funções extraídas
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func calcularCobranca(data time.Time, quantidade int, precoUnitario float64) float64 {
    var cobranca float64
    if !data.Before(inicioVerao) && !data.After(fimVerao) {
        cobranca = float64(quantidade) * precoUnitario * taxaVerao
    } else {
        cobranca = float64(quantidade) * precoUnitario * taxaInverno
        if quantidade > limiteServicoInverno {
            cobranca += cobrancaServicoInverno
        }
    }
    return cobranca
}
```

**DEPOIS — esperado:**
```go
func calcularCobranca(data time.Time, quantidade int, precoUnitario float64) float64 {
    if eVerao(data) {
        return cobrancaVerao(quantidade, precoUnitario)
    }
    return cobrancaInverno(quantidade, precoUnitario)
}

func eVerao(data time.Time) bool {
    return !data.Before(inicioVerao) && !data.After(fimVerao)
}

func cobrancaVerao(quantidade int, precoUnitario float64) float64 {
    return float64(quantidade) * precoUnitario * taxaVerao
}

func cobrancaInverno(quantidade int, precoUnitario float64) float64 {
    base := float64(quantidade) * precoUnitario * taxaInverno
    if quantidade > limiteServicoInverno {
        return base + cobrancaServicoInverno
    }
    return base
}
```

**Por que esse padrão:**
- `eVerao` nomeia a regra de negócio — leitores entendem sem analisar datas
- `cobrancaVerao` e `cobrancaInverno` expressam intenção de precificação, não implementação

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair a função mas dar um nome técnico**
```go
// Não aceito — o nome descreve o mecanismo, não a regra de negócio
func verificarIntervaloData(data time.Time) bool {
    return !data.Before(inicioVerao) && !data.After(fimVerao)
}
```

**Erro 2: Extrair apenas a condição mas não os branches**
```go
// Não aceito — extração parcial; os branches ainda precisam de nomes
if eVerao(data) {
    cobranca = float64(quantidade) * precoUnitario * taxaVerao // ainda inline
}
```

---

## 7. Benefícios

- **Legibilidade:** O `if` lê como uma regra de negócio, não uma fórmula
- **Testabilidade:** Cada branch pode ser testado isoladamente
- **Documentação:** Nomes de funções substituem a necessidade de comentários
