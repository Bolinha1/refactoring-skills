# TÉCNICA: Replace Temp with Query — Java

## Fonte
Baseado em: https://refactoring.guru/replace-temp-with-query

---

## 1. Problema

Uma variável local armazena o resultado de uma expressão. Esse valor é usado mais adiante no método, mas a variável impede que a expressão seja reutilizada em outros métodos ou subclasses. O método cresce porque carrega estado que poderia ser movido para um método de consulta reutilizável.

---

## 2. Solução

Extraia a expressão para um método. Substitua cada uso da variável local por uma chamada ao novo método.

---

## 3. Quando aplicar

- A mesma variável local é calculada no início e usada bem mais abaixo, tornando o método longo
- A expressão seria útil em outros métodos da mesma classe
- A variável é somente leitura (nunca reatribuída após a inicialização)
- Uma subclasse pode querer sobrescrever a expressão com lógica diferente

---

## 4. Passos de refatoração

1. Verifique que a expressão não tem efeitos colaterais
2. Extraia a expressão para um método privado com nome claro
3. Substitua cada referência à variável local por uma chamada ao novo método
4. Remova a variável local
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public double preco(Pedido pedido) {
    final double precoBase = pedido.getQuantidade() * pedido.getPrecoUnitario();
    final double fatorDesconto = precoBase > 1000 ? 0.95 : 0.98;
    return precoBase * fatorDesconto;
}
```

**DEPOIS — esperado:**
```java
public double preco(Pedido pedido) {
    return precoBase(pedido) * fatorDesconto(pedido);
}

private double precoBase(Pedido pedido) {
    return pedido.getQuantidade() * pedido.getPrecoUnitario();
}

private double fatorDesconto(Pedido pedido) {
    return precoBase(pedido) > 1000 ? 0.95 : 0.98;
}
```

**Por que esse padrão:**
- `precoBase()` e `fatorDesconto()` são métodos de consulta reutilizáveis
- Uma subclasse pode sobrescrever `fatorDesconto()` para aplicar uma regra de precificação diferente

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Aplicar a técnica quando a expressão tem efeitos colaterais**
```java
// Não aceito — um método chamado múltiplas vezes não deve ter efeitos colaterais
private int proximoId() {
    return ++this.contador; // efeito colateral; chamá-lo duas vezes dá resultados diferentes
}
```

**Erro 2: Criar um método de consulta que requer configuração complexa não disponível fora do método atual**
```java
// Não aceito — se a expressão depende de muitas variáveis locais que todas
// precisariam virar parâmetros, Extract Method é mais apropriado
```

---

## 7. Benefícios

- **Reuso:** O método extraído pode ser chamado a partir de outros métodos da mesma classe
- **Extensibilidade:** Subclasses podem sobrescrever métodos de consulta individuais
- **Legibilidade:** O método principal lê como uma fórmula de alto nível com subexpressões nomeadas
