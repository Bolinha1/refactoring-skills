# TÉCNICA: Extract Method — Java

## Fonte
Baseado em: https://refactoring.guru/extract-method

---

## 1. Problema

Você tem um fragmento de código que pode ser agrupado em uma unidade com sentido próprio,
mas ele está embutido em um método maior junto com outras responsabilidades.

---

## 2. Solução

Mova esse fragmento para um novo método privado com nome que descreva a intenção.
Substitua o trecho original por uma chamada ao novo método.

---

## 3. Quando aplicar

- O trecho de código mereceria um comentário explicativo
- O mesmo bloco de lógica aparece em mais de um lugar
- O método atual faz mais de uma coisa claramente distinta
- Existe dificuldade em dar um nome curto e preciso ao método atual

---

## 4. Passos de refatoração

1. Crie um novo método com nome que expresse a intenção do trecho
2. Copie o fragmento de código para o novo método
3. Identifique variáveis locais usadas pelo trecho:
   - Usadas apenas dentro do trecho → tornam-se variáveis locais do novo método
   - Declaradas antes e lidas dentro → tornam-se parâmetros
   - Modificadas dentro e usadas depois → o novo método deve retorná-las
4. Substitua o trecho original pela chamada ao novo método
5. Compile e execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public void imprimirFatura(Fatura fatura) {
    // imprimir cabeçalho
    System.out.println("***********************");
    System.out.println("***   FATURA #" + fatura.getNumero() + "   ***");
    System.out.println("***********************");

    // imprimir itens
    for (ItemFatura item : fatura.getItens()) {
        System.out.println(item.getNome() + "\t" + item.getQuantidade() + "\t" + item.getPreco());
    }

    // imprimir total
    double total = fatura.getItens().stream()
        .mapToDouble(i -> i.getQuantidade() * i.getPreco())
        .sum();
    System.out.println("TOTAL: R$ " + total);
}
```

**DEPOIS — esperado:**
```java
public void imprimirFatura(Fatura fatura) {
    imprimirCabecalho(fatura);
    imprimirItens(fatura);
    imprimirTotal(fatura);
}

private void imprimirCabecalho(Fatura fatura) {
    System.out.println("***********************");
    System.out.println("***   FATURA #" + fatura.getNumero() + "   ***");
    System.out.println("***********************");
}

private void imprimirItens(Fatura fatura) {
    for (ItemFatura item : fatura.getItens()) {
        System.out.println(item.getNome() + "\t" + item.getQuantidade() + "\t" + item.getPreco());
    }
}

private void imprimirTotal(Fatura fatura) {
    double total = fatura.getItens().stream()
        .mapToDouble(i -> i.getQuantidade() * i.getPreco())
        .sum();
    System.out.println("TOTAL: R$ " + total);
}
```

**Variante — método com retorno:**
```java
// ANTES — variável temporária modificada e usada depois
public double calcularDesconto(Pedido pedido) {
    double subtotal = 0;
    for (Item item : pedido.getItens()) {
        subtotal += item.getPreco() * item.getQuantidade();
    }
    // ... outras coisas ...
    return subtotal * pedido.getTaxaDesconto();
}

// DEPOIS — extração com retorno
public double calcularDesconto(Pedido pedido) {
    double subtotal = calcularSubtotal(pedido);
    return subtotal * pedido.getTaxaDesconto();
}

private double calcularSubtotal(Pedido pedido) {
    return pedido.getItens().stream()
        .mapToDouble(i -> i.getPreco() * i.getQuantidade())
        .sum();
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Nome vago que não expressa intenção**
```java
// Não aceito
private void processarParte1(Fatura fatura) { ... }
private void helper(Fatura fatura) { ... }
```

**Erro 2: Extrair trecho muito pequeno sem ganho de legibilidade**
```java
// Não aceito — uma linha não precisa virar método
private void imprimirNewLine() {
    System.out.println();
}
```

**Erro 3: Deixar o método original ainda grande após extração**
```java
// Não aceito — extraiu um trecho mas o método ainda tem 50 linhas
public void imprimirFatura(Fatura fatura) {
    imprimirCabecalho(fatura);
    // ... 40 linhas restantes sem extração ...
}
```

---

## 7. Benefícios

- **Legibilidade:** O método principal vira uma narrativa de alto nível
- **Reuso:** O método extraído pode ser reutilizado em outros contextos
- **Testabilidade:** Métodos menores são mais fáceis de testar isoladamente
- **Isolamento de erros:** Mudanças afetam apenas o método que as contém
