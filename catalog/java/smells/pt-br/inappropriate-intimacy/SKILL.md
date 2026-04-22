# SMELL: Inappropriate Intimacy — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/inappropriate-intimacy

---

## 1. O que é?

Uma classe acessa os campos privados, coleções internas ou detalhes de implementação de outra classe de forma excessiva. As duas classes estão fortemente acopladas de um modo que dificulta a mudança de uma sem afetar a outra.

---

## 2. Sinais de alerta

- [ ] Uma classe chama múltiplos getters de outra classe para realizar um cálculo que deveria viver nessa outra classe
- [ ] Uma classe muta diretamente a coleção ou estado interno de outra classe
- [ ] Duas classes se referenciam mutuamente em uma dependência bidirecional
- [ ] Uma classe conhece a estrutura interna de outra (ex.: sabe quais campos são null em qual estado)
- [ ] "Enfileiramentos de trens" repetidos: `a.getB().getC().fazerAlgo()`

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Move Method** | Mova o método que acessa internos alheios para a classe que possui esses internos |
| **Move Field** | Mova um campo para a classe que mais o utiliza |
| **Hide Delegate** | Adicione um método de encaminhamento para que o cliente não precise acessar através de um intermediário |
| **Change Bidirectional Association to Unidirectional** | Remova uma direção de uma referência bidirecional quando uma direção é suficiente |
| **Extract Class** | Se duas classes são muito íntimas porque compartilham responsabilidades, separe essas responsabilidades de forma limpa |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
public class Pedido {
    private List<ItemPedido> itens = new ArrayList<>();
    private String emailCliente;

    public List<ItemPedido> getItens() { return itens; } // expõe internos
    public String getEmailCliente() { return emailCliente; }
}

// Relatorio vasculha os internos de Pedido
public class RelatorioPedido {
    public String gerar(Pedido pedido) {
        double total = 0;
        for (ItemPedido item : pedido.getItens()) {   // acessando internos de Pedido
            total += item.getPreco() * item.getQuantidade();
        }
        return "Pedido para " + pedido.getEmailCliente() + " total: " + total;
    }
}
```

**DEPOIS — esperado:**
```java
public class Pedido {
    private final List<ItemPedido> itens;
    private final String emailCliente;

    public double total() {
        return itens.stream()
            .mapToDouble(i -> i.getPreco() * i.getQuantidade())
            .sum();
    }

    public String getEmailCliente() { return emailCliente; }
}

// Relatorio apenas pergunta ao Pedido o que precisa
public class RelatorioPedido {
    public String gerar(Pedido pedido) {
        return "Pedido para " + pedido.getEmailCliente() + " total: " + pedido.total();
    }
}
```

**Por que este padrão:**
- `RelatorioPedido` não depende mais da estrutura interna de `Pedido`
- Mudar como `Pedido` armazena itens não afeta `RelatorioPedido`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Substituir o loop por stream mas ainda puxando a lista**
```java
// Não aceito — getItens() ainda expõe os internos
double total = pedido.getItens().stream()
    .mapToDouble(i -> i.getPreco() * i.getQuantidade())
    .sum();
```

**Erro 2: Criar um DTO que copia todos os campos de Pedido para evitar a cadeia de getters**
```java
// Não aceito — um PedidoDTO com todos os mesmos campos é apenas indireção; o acoplamento permanece
```

---

## 6. Benefícios

- **Menor acoplamento:** Classes podem mudar seus internos sem afetar seus clientes
- **Melhor encapsulamento:** A classe proprietária controla como seus dados são calculados e acessados
- **Mais fácil de testar:** Classes com menos dependências são mais simples de testar em isolamento
