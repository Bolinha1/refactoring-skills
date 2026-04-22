# TECHNIQUE: Extract Variable — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/extract-variable

---

## 1. Problema

Uma expressão complexa é difícil de entender de relance. Resultados intermediários são calculados inline, tornando difícil ver o que cada parte significa ou depurar o cálculo.

---

## 2. Solução

Atribua a expressão (ou uma sub-expressão) a uma variável temporária com nome claro. Use o nome da variável para transmitir o significado do resultado.

---

## 3. Quando aplicar

- Uma expressão condicional tem múltiplas partes que cada uma merece um nome
- Uma longa expressão aritmética ou de string oculta o que está calculando
- A mesma sub-expressão aparece mais de uma vez no mesmo escopo
- Um ponto de interrupção de depuração é necessário em um resultado intermediário

---

## 4. Passos de refatoração

1. Identifique a expressão ou sub-expressão a nomear
2. Declare uma variável local `final` (ou efetivamente final) e atribua a expressão a ela
3. Substitua a expressão original (e todas as duplicatas no mesmo escopo) pela variável
4. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public double preco(Pedido pedido) {
    return pedido.getQuantidade() * pedido.getPrecoItem()
        - Math.max(0, pedido.getQuantidade() - 500) * pedido.getPrecoItem() * 0.05
        + Math.min(pedido.getQuantidade() * pedido.getPrecoItem() * 0.1, 100.0);
}
```

**DEPOIS — esperado:**
```java
public double preco(Pedido pedido) {
    final double precoBase = pedido.getQuantidade() * pedido.getPrecoItem();
    final double descontoQuantidade = Math.max(0, pedido.getQuantidade() - 500)
        * pedido.getPrecoItem() * 0.05;
    final double frete = Math.min(precoBase * 0.1, 100.0);
    return precoBase - descontoQuantidade + frete;
}
```

**Por que este padrão:**
- Cada nome de variável explica a intenção de sua sub-expressão
- `precoBase` é calculado uma vez e reutilizado, removendo duplicação

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Nomear uma variável pelo tipo ou posição em vez do significado**
```java
// Não aceito — "resultado", "temp", "valor" não adicionam informação
final double resultado = pedido.getQuantidade() * pedido.getPrecoItem();
```

**Erro 2: Extrair excessivamente cada sub-expressão individual**
```java
// Não aceito — muito granular; operações simples raramente precisam de variável
final int qtd = pedido.getQuantidade();
final double preco = pedido.getPrecoItem();
final double produto = qtd * preco;
```

**Erro 3: Mutar a variável após a extração**
```java
// Não aceito — uma variável extraída para clareza deve ser final
double precoBase = pedido.getQuantidade() * pedido.getPrecoItem();
precoBase = precoBase * 1.1; // confuso — deve ser uma nova variável
```

---

## 7. Benefícios

- **Legibilidade:** Nomes tornam expressões complexas autodocumentadas
- **Depurabilidade:** Você pode definir um ponto de interrupção em cada etapa nomeada
- **Duplicação reduzida:** Uma sub-expressão usada mais de uma vez é calculada uma vez e nomeada
