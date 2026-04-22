# TÉCNICA: Extrair Método — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/extract-method

---

## 1. Problema

Você tem um fragmento de código que pode ser agrupado em uma unidade com sentido próprio, mas ele está embutido em uma função maior junto com outras responsabilidades. O resultado é uma função longa difícil de ler e testar.

---

## 2. Solução

Mova esse fragmento para uma nova função com nome que descreva a intenção. Substitua o trecho original por uma chamada à nova função.

---

## 3. Quando aplicar

- O trecho de código mereceria um comentário explicativo
- O mesmo bloco de lógica aparece em mais de um lugar
- A função atual faz mais de uma coisa claramente distinta
- Existe dificuldade em dar um nome curto e preciso à função atual

---

## 4. Passos de refatoração

1. Crie uma nova função com nome que expresse a intenção do trecho
2. Copie o fragmento de código para a nova função
3. Identifique variáveis usadas pelo trecho:
   - Usadas apenas dentro do trecho → variáveis locais da nova função
   - Declaradas antes e lidas dentro → tornam-se parâmetros
   - Modificadas dentro e usadas depois → a nova função deve retorná-las
4. Substitua o trecho original pela chamada à nova função
5. Compile e execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func imprimirFatura(fatura *Fatura) {
    // imprimir cabeçalho
    fmt.Println("***********************")
    fmt.Printf("***   FATURA #%d   ***\n", fatura.Numero)
    fmt.Println("***********************")

    // imprimir itens
    for _, item := range fatura.Itens {
        fmt.Printf("%s\t%d\t%.2f\n", item.Nome, item.Quantidade, item.Preco)
    }

    // imprimir total
    total := 0.0
    for _, item := range fatura.Itens {
        total += float64(item.Quantidade) * item.Preco
    }
    fmt.Printf("TOTAL: R$ %.2f\n", total)
}
```

**DEPOIS — esperado:**
```go
func imprimirFatura(fatura *Fatura) {
    imprimirCabecalho(fatura)
    imprimirItens(fatura)
    imprimirTotal(fatura)
}

func imprimirCabecalho(fatura *Fatura) {
    fmt.Println("***********************")
    fmt.Printf("***   FATURA #%d   ***\n", fatura.Numero)
    fmt.Println("***********************")
}

func imprimirItens(fatura *Fatura) {
    for _, item := range fatura.Itens {
        fmt.Printf("%s\t%d\t%.2f\n", item.Nome, item.Quantidade, item.Preco)
    }
}

func imprimirTotal(fatura *Fatura) {
    total := calcularTotal(fatura)
    fmt.Printf("TOTAL: R$ %.2f\n", total)
}

func calcularTotal(fatura *Fatura) float64 {
    total := 0.0
    for _, item := range fatura.Itens {
        total += float64(item.Quantidade) * item.Preco
    }
    return total
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Nome vago que não expressa intenção**
```go
// Não aceito — o nome não diz o que a função faz
func processarParte1(fatura *Fatura) { ... }
func helper(fatura *Fatura) { ... }
```

**Erro 2: Extrair trecho muito pequeno sem ganho de legibilidade**
```go
// Não aceito — uma linha simples não precisa virar função
func imprimirNovaLinha() {
    fmt.Println()
}
```

**Erro 3: Deixar a função original ainda grande após a extração**
```go
// Não aceito — extraiu um trecho mas a função ainda tem 50 linhas
func imprimirFatura(fatura *Fatura) {
    imprimirCabecalho(fatura)
    // ... 40 linhas restantes sem extração ...
}
```

---

## 7. Benefícios

- **Legibilidade:** A função principal vira uma narrativa de alto nível
- **Reuso:** A função extraída pode ser reutilizada em outros contextos
- **Testabilidade:** Funções menores são mais fáceis de testar isoladamente
- **Isolamento de erros:** Mudanças afetam apenas a função que as contém
