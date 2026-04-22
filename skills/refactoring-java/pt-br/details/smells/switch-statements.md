# SKILL: Detectando e Refatorando Switch Statements — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/switch-statements

---

## 1. O que é Switch Statements

Cadeias complexas de switch ou if/else que ramificam no tipo ou estado de um objeto. A mesma lógica de switch tende a ser duplicada pela base de código — quando um novo caso é adicionado, cada switch precisa ser encontrado e atualizado.

**Por que isso acontece:**
- Comportamento baseado em tipo foi implementado de forma procedural em vez de usar polimorfismo
- Uma única classe cresceu para lidar com múltiplas variantes de um conceito
- O conceito de "tipo" foi codificado como uma string ou inteiro em vez de uma hierarquia de classes

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Bloco `switch` ou `if/else if` que ramifica em um campo de tipo, status ou enum
- [ ] O mesmo switch aparece em mais de um lugar
- [ ] Adicionar um novo "tipo" requer buscar e atualizar cada switch na base de código
- [ ] Cada branch do switch faz algo muito diferente dos outros
- [ ] Switch em um tipo para decidir qual objeto instanciar

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                       | Técnica recomendada                    |
|-----------------------------------------------------------|----------------------------------------|
| Switch em tipo para decidir comportamento                 | Replace Conditional with Polymorphism  |
| Switch em tipo para decidir qual classe criar             | Replace Constructor with Factory Method |
| Poucos casos e adicionar novos é raro                     | Replace Type Code with Subclasses      |
| O tipo tem estado mutável que muda em runtime             | Replace Type Code with State/Strategy  |
| Switch é simples e usado em apenas um lugar               | Deixe assim — nem todo switch é um smell |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
public class CalculadoraFrete {
    public double calcularCusto(Pedido pedido) {
        switch (pedido.getTipoFrete()) {
            case "padrao":
                return pedido.getPeso() * 1.5;
            case "expresso":
                return pedido.getPeso() * 3.0 + 5.0;
            case "noturno":
                return pedido.getPeso() * 5.0 + 20.0;
            default:
                throw new IllegalArgumentException("Tipo de frete desconhecido");
        }
    }
}
```

**DEPOIS — esperado:**
```java
public interface EstrategiaFrete {
    double calcularCusto(double peso);
}

public class FretePadrao implements EstrategiaFrete {
    public double calcularCusto(double peso) { return peso * 1.5; }
}

public class FreteExpresso implements EstrategiaFrete {
    public double calcularCusto(double peso) { return peso * 3.0 + 5.0; }
}

public class FreteNoturno implements EstrategiaFrete {
    public double calcularCusto(double peso) { return peso * 5.0 + 20.0; }
}

public class Pedido {
    private final EstrategiaFrete estrategiaFrete;

    public double calcularCustoFrete() {
        return estrategiaFrete.calcularCusto(this.peso);
    }
}
```

**Por que este padrão:**
- Adicionar um novo tipo de frete requer apenas uma nova classe, não busca e atualização em cada switch
- Cada estratégia é independentemente testável e coesa

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Substituir o switch por uma cadeia de if/else**
```java
// Não aceito — mesmo smell, sintaxe diferente
if (tipo.equals("padrao")) { ... }
else if (tipo.equals("expresso")) { ... }
else if (tipo.equals("noturno")) { ... }
```

**Erro 2: Centralizar em uma fábrica deus sem polimorfismo**
```java
// Não aceito — o switch ainda existe, apenas movido
public static CalculadoraFrete criar(String tipo) {
    switch (tipo) { ... }
}
```

**Erro 3: Aplicar polimorfismo a um switch que nunca vai crescer**
```java
// Não aceito — criar 3 classes para um toggle verdadeiro/falso é over-engineering
switch (pedido.isPrioritario()) {
    case true: return precoPrioritario;
    case false: return precoPadrao;
}
```

---

## 6. Benefícios

- **Open/Closed:** Adicionar uma nova variante requer uma nova classe, não editar as existentes
- **Localidade:** O comportamento de cada variante está contido em um único lugar
- **Testabilidade:** Cada variante polimórfica pode ser testada isoladamente
