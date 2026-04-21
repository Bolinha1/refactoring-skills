# SKILL: Detectando e Refatorando Feature Envy — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/feature-envy

---

## 1. O que é Feature Envy

Um método acessa os dados de outro objeto mais do que acessa os dados da sua própria classe. O método tem "inveja" da outra classe — parece querer viver lá.

**Por que isso acontece:**
- Campos foram movidos para uma nova classe mas os métodos que os usam não foram
- Lógica de negócio foi colocada em uma classe utilitária que opera nos internos de um objeto de domínio
- Um método foi gradualmente estendido para depender cada vez mais dos getters de outro objeto

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Um método chama vários getters de outro objeto em sequência
- [ ] Um método recebe um objeto como parâmetro e usa principalmente seus campos/métodos
- [ ] Um método ignora completamente o `this` e só manipula o estado de outro objeto
- [ ] O nome do método naturalmente pertence à outra classe ("calcularTotalPedido" em um ServiçoRelatorio)

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                     | Técnica recomendada               |
|---------------------------------------------------------|-----------------------------------|
| O método inteiro pertence à outra classe                | Move Method                       |
| Apenas parte do método tem inveja de outra classe       | Extract Method, depois Move Method |
| O método usa dados de várias classes                    | Mover para a classe com mais dados |
| O método é um cálculo que pertence ao modelo            | Move Method para objeto de domínio |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
public class ImpressoraFatura {
    public double calcularTotalFatura(Fatura fatura) {
        double subtotal = 0;
        for (ItemFatura item : fatura.getItens()) {
            subtotal += item.getPreco() * item.getQuantidade();
        }
        double imposto = subtotal * fatura.getTaxaImposto();
        double desconto = fatura.temDesconto() ? subtotal * fatura.getTaxaDesconto() : 0;
        return subtotal + imposto - desconto;
    }
}
```

**DEPOIS — esperado:**
```java
public class Fatura {
    public double calcularTotal() {
        double subtotal = itens.stream()
            .mapToDouble(i -> i.getPreco() * i.getQuantidade())
            .sum();
        double imposto = subtotal * taxaImposto;
        double desconto = temDesconto() ? subtotal * taxaDesconto : 0;
        return subtotal + imposto - desconto;
    }
}

public class ImpressoraFatura {
    public void imprimir(Fatura fatura) {
        double total = fatura.calcularTotal(); // delega ao dono
        // ... lógica de impressão
    }
}
```

**Por que este padrão:**
- `calcularTotal` naturalmente pertence à `Fatura` — só usa dados da Fatura
- `ImpressoraFatura` é liberada de conhecer os internos da Fatura

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter o método invejoso e adicionar um pass-through**
```java
// Não aceito — delegação sem mover cria indireção, não clareza
public double calcularTotalFatura(Fatura fatura) {
    return fatura.calcularTotal(); // apenas um wrapper — delete o método invejoso completamente
}
```

**Erro 2: Mover o método mas manter parâmetros que expõem internos**
```java
// Não aceito — movido mas ainda passando valores de campo brutos
public double calcularTotal(double subtotal, double taxaImposto, double taxaDesconto, boolean temDesconto) { ... }
```

**Erro 3: Mover um método que usa dados de muitas classes**
```java
// Não aceito — se o método usa Pedido, Cliente e Cupom igualmente,
// movê-lo para qualquer uma das classes apenas desloca a inveja; Extract Class é melhor
```

---

## 6. Benefícios

- **Coesão:** Cada classe possui o comportamento que opera sobre seus próprios dados
- **Encapsulamento:** O objeto de domínio expõe intenção, não campos brutos
- **Manutenibilidade:** As regras de negócio são colocalizadas com os dados que governam
