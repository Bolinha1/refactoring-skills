# TÉCNICA: Decompose Conditional — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/decompose-conditional

---

## 1. Problema

Uma expressão condicional complexa (e o código em seus branches) torna difícil entender o que está sendo testado e o que acontece em cada caso.

---

## 2. Solução

Extraia a condição e cada branch para métodos bem nomeados. Os nomes dos métodos explicam a intenção; os corpos explicam a implementação.

---

## 3. Quando aplicar

- A condição em si é uma expressão booleana complexa que requer reflexão para analisar
- O código no branch verdadeiro ou falso tem várias linhas e merece um nome
- O condicional aparece dentro de um método já longo (combina com Long Method)
- Ler a condição requer conhecimento de regras de domínio, não apenas lógica de programação

---

## 4. Passos de refatoração

1. Extraia a condição para um método nomeado conforme a regra de negócio que verifica
2. Extraia o branch verdadeiro para um método nomeado conforme o que faz
3. Extraia o branch falso para um método nomeado conforme o que faz
4. Substitua o `if` original por chamadas aos métodos extraídos
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public double calcularCobrança(LocalDate data, int quantidade, double precoUnitario) {
    double cobrança;
    if (!data.isBefore(INICIO_VERAO) && !data.isAfter(FIM_VERAO)) {
        cobrança = quantidade * precoUnitario * TAXA_VERAO;
    } else {
        cobrança = quantidade * precoUnitario * TAXA_INVERNO +
                   (quantidade > LIMITE_SERVICO_INVERNO ? COBRANÇA_SERVICO_INVERNO : 0);
    }
    return cobrança;
}
```

**DEPOIS — esperado:**
```java
public double calcularCobrança(LocalDate data, int quantidade, double precoUnitario) {
    if (eVerao(data)) {
        return cobranças Verao(quantidade, precoUnitario);
    } else {
        return cobrançaInverno(quantidade, precoUnitario);
    }
}

private boolean eVerao(LocalDate data) {
    return !data.isBefore(INICIO_VERAO) && !data.isAfter(FIM_VERAO);
}

private double cobrançaVerao(int quantidade, double precoUnitario) {
    return quantidade * precoUnitario * TAXA_VERAO;
}

private double cobrançaInverno(int quantidade, double precoUnitario) {
    double base = quantidade * precoUnitario * TAXA_INVERNO;
    double servicoExtra = quantidade > LIMITE_SERVICO_INVERNO ? COBRANÇA_SERVICO_INVERNO : 0;
    return base + servicoExtra;
}
```

**Por que este padrão:**
- `eVerao` nomeia a regra de negócio — leitores entendem sem analisar a lógica de datas
- `cobrançaVerao` e `cobrançaInverno` expressam intenção de precificação, não implementação

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair o método mas dar um nome técnico**
```java
// Não aceito — o nome descreve o mecanismo, não a regra de negócio
private boolean verificarIntervaloData(LocalDate data) {
    return !data.isBefore(INICIO_VERAO) && !data.isAfter(FIM_VERAO);
}
```

**Erro 2: Extrair apenas a condição mas não os branches**
```java
// Não aceito — extração parcial; os branches ainda precisam de nomes
if (eVerao(data)) {
    cobrança = quantidade * precoUnitario * TAXA_VERAO; // ainda inline
}
```

**Erro 3: Decompor uma condição trivial**
```java
// Não aceito — over-engineering para uma verificação simples
private boolean eNulo(Object obj) { return obj == null; }
if (eNulo(cliente)) { ... }
```

---

## 7. Benefícios

- **Legibilidade:** A declaração `if` lê como uma regra de negócio, não uma fórmula
- **Testabilidade:** Cada branch pode ser testado isoladamente
- **Documentação:** Nomes de métodos substituem a necessidade de comentários
