# TÉCNICA: Split Temporary Variable — Java

## Fonte
Baseado em: https://refactoring.guru/split-temporary-variable

---

## 1. Problema

Uma variável local é atribuída mais de uma vez, servindo a propósitos diferentes em momentos distintos do método. Reutilizar o mesmo nome para conceitos diferentes torna o código difícil de acompanhar e impede que cada atribuição seja declarada como `final`.

---

## 2. Solução

Crie uma variável separada para cada atribuição. Dê a cada variável um nome que descreva seu propósito específico.

---

## 3. Quando aplicar

- Uma variável é atribuída em um lugar e depois reatribuída com um significado completamente diferente no mesmo método
- O nome da variável é genérico (ex.: `temp`, `resultado`, `valor`) porque cobre múltiplos conceitos
- Declarar a variável como `final` causaria erro de compilação por causa da reatribuição

---

## 4. Passos de refatoração

1. Identifique a variável que é atribuída mais de uma vez para propósitos diferentes
2. Renomeie a primeira atribuição com um nome que descreva o primeiro propósito
3. Renomeie as atribuições subsequentes com novas variáveis cujos nomes descrevam seus propósitos
4. Declare cada variável como `final`
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public double calcular(double altura, double largura) {
    double temp = 2 * (altura + largura);  // perímetro
    System.out.println("Perímetro: " + temp);
    temp = altura * largura;               // área — mesma variável, conceito diferente
    System.out.println("Área: " + temp);
    return temp;
}
```

**DEPOIS — esperado:**
```java
public double calcular(double altura, double largura) {
    final double perimetro = 2 * (altura + largura);
    System.out.println("Perímetro: " + perimetro);
    final double area = altura * largura;
    System.out.println("Área: " + area);
    return area;
}
```

**Por que esse padrão:**
- `perimetro` e `area` são conceitos distintos que merecem nomes distintos
- Ambas as variáveis agora são `final`, evitando reutilização acidental no futuro

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Dividir um acumulador de loop**
```java
// Cuidado — um acumulador de loop é reatribuído intencionalmente; não divida
double total = 0;
for (Item item : itens) {
    total += item.getPreco(); // esta reatribuição é intencional
}
```

**Erro 2: Manter sufixo genérico ao invés de nomear pelo propósito**
```java
// Não aceito — temp1 e temp2 são tão sem sentido quanto temp
double temp1 = 2 * (altura + largura);
double temp2 = altura * largura;
```

---

## 7. Benefícios

- **Clareza:** Cada nome de variável torna seu propósito único óbvio
- **Imutabilidade:** Variáveis `final` não podem ser reutilizadas acidentalmente para um terceiro propósito
- **Depuração mais fácil:** Cada etapa do cálculo tem um valor nomeado distinto
