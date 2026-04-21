# SMELL: Lazy Class — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/lazy-class

---

## 1. O que é?

Uma classe que faz tão pouco que não vale o esforço de entendê-la e mantê-la. Pode ter sido útil no passado, existir em antecipação a crescimento futuro, ou ser a casca restante de uma refatoração que extraiu a maior parte de sua lógica.

---

## 2. Sinais de alerta

- [ ] Uma classe possui apenas um ou dois métodos triviais
- [ ] Uma classe delega tudo a outra única classe com valor agregado mínimo
- [ ] Uma classe foi criada para abstrair algo que nunca precisou ser abstraído
- [ ] A classe poderia ser substituída por um único método ou função em outro lugar
- [ ] A classe tem poucos usos no código

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Inline Class** | Quando a classe faz tão pouco que sua lógica pode ser absorvida pelo chamador |
| **Collapse Hierarchy** | Quando a classe preguiçosa é uma subclasse ou superclasse quase vazia em uma hierarquia de herança |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
// CalculadoraImposto faz apenas uma coisa trivial
public class CalculadoraImposto {
    public double calcular(double valor) {
        return valor * 0.2;
    }
}

// ServicoPedido apenas delega para CalculadoraImposto
public class ServicoPedido {
    private final CalculadoraImposto calculadoraImposto = new CalculadoraImposto();

    public double totalComImposto(double valor) {
        return valor + calculadoraImposto.calcular(valor);
    }
}
```

**DEPOIS — esperado:**
```java
public class ServicoPedido {
    private static final double TAXA_IMPOSTO = 0.2;

    public double totalComImposto(double valor) {
        return valor * (1 + TAXA_IMPOSTO);
    }
}
```

**Por que este padrão:**
- A lógica de imposto é simples o suficiente para viver diretamente no serviço
- Uma classe a menos significa um arquivo a menos para navegar, testar e manter

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Fazer inline de uma classe que encapsula uma regra mutável**
```java
// Cuidado — se a taxa de imposto difere por país ou categoria de produto,
// CalculadoraImposto pode estar prestes a crescer; verifique o futuro antes de fazer inline
```

**Erro 2: Fazer inline de uma classe compartilhada por múltiplos chamadores**
```java
// Não aceito — se CalculadoraImposto é usada por ServicoPedido, ServicoFatura e
// ServicoOrcamento, fazer inline duplica a lógica em três lugares
```

---

## 6. Benefícios

- **Complexidade reduzida:** Menos classes significa um mapa mental menor do sistema
- **Navegação mais simples:** Desenvolvedores não seguem uma cadeia de delegações para encontrar lógica trivial
- **Menos boilerplate:** Nenhum construtor, campo ou import necessário para uma classe que quase não faz nada
