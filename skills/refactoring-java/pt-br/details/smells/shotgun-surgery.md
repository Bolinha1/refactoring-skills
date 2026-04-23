# SKILL: Detectando e Refatorando Shotgun Surgery — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/shotgun-surgery

---

## 1. O que é Shotgun Surgery

Uma única mudança lógica requer modificar muitas classes diferentes ao mesmo tempo. Fazer uma mudança conceitual "dispara" edições pela base de código como um tiro de espingarda — tocando muitos arquivos para o que deveria ser uma correção localizada.

O inverso do Divergent Change: muitas classes, uma razão para mudar.

**Por que isso acontece:**
- Uma responsabilidade foi fragmentada em muitas classes em vez de viver em um único lugar
- Lógica que deveria ser centralizada foi duplicada ou distribuída por "flexibilidade"
- Preocupações transversais (logging, auditoria, notificações) foram tratadas inline em todo lugar

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Adicionar uma nova funcionalidade força você a editar 5+ classes não relacionadas
- [ ] Renomear um conceito requer tocar dezenas de arquivos
- [ ] Uma única regra de negócio é aplicada em múltiplos lugares
- [ ] Você sempre precisa fazer a mesma mudança em vários lugares simultaneamente
- [ ] Falhas de teste em cascata por muitas classes de teste para uma única mudança conceitual

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                            | Técnica recomendada     |
|----------------------------------------------------------------|-------------------------|
| Comportamento espalhado todo relacionado a um conceito         | Move Method + Move Field |
| Múltiplas classes pequenas contribuindo para um conceito       | Inline Class            |
| Preocupação transversal repetida em todo lugar                 | Extract Class           |
| Regra de negócio duplicada aplicada em muitos lugares          | Move Method para um dono |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
// Adicionar o conceito de "moeda" requer mudar TODAS essas classes:

public class Produto {
    private double preco; // deve adicionar campo CodigoMoeda aqui
}

public class Pedido {
    public double getTotal() { return itens.stream()... } // deve formatar com moeda
}

public class Fatura {
    public String formatarValor(double valor) { return "R$" + valor; } // deve usar moeda
}

public class Recibo {
    public void imprimirTotal(double valor) { System.out.println(valor); } // deve usar moeda
}
```

**DEPOIS — esperado:**
```java
public class Dinheiro {
    private final BigDecimal valor;
    private final Currency moeda;

    public Dinheiro(BigDecimal valor, Currency moeda) {
        this.valor = valor;
        this.moeda = moeda;
    }

    public String formatar() {
        return NumberFormat.getCurrencyInstance(moeda.getLocale()).format(valor);
    }

    public Dinheiro somar(Dinheiro outro) {
        if (!this.moeda.equals(outro.moeda)) throw new MoedaDiferenteException();
        return new Dinheiro(this.valor.add(outro.valor), this.moeda);
    }
}

// Agora Produto, Pedido, Fatura, Recibo usam Dinheiro — um conceito, um lugar
public class Produto {
    private Dinheiro preco;
}
```

**Por que este padrão:**
- `Dinheiro` centraliza todo comportamento de moeda — adicionar um novo formato de moeda toca uma classe
- Todos os chamadores se beneficiam automaticamente

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Centralizar os dados mas não o comportamento**
```java
// Não aceito — DinheiroDto agrupa os campos mas formatação/aritmética ficam espalhadas
public class DinheiroDto {
    public double valor;
    public String codigoMoeda;
}
```

**Erro 2: Usar uma classe utilitária estática como ponto único**
```java
// Não aceito — DinheiroUtil é um smell escondido; lógica ainda fragmentada entre chamadores
public class DinheiroUtil {
    public static String formatar(double valor, String moeda) { ... }
}
```

**Erro 3: Fazer inline de classes agressivamente, criando uma nova god class**
```java
// Não aceito — fazer inline de todas as classes pequenas em uma megaclasse troca
// Shotgun Surgery por Divergent Change
```

---

## 6. Benefícios

- **Localidade:** Um conceito de negócio = uma classe para mudar
- **Segurança:** Mudanças são mais fáceis de raciocinar quando o blast radius é pequeno
- **Descobribilidade:** Todo comportamento de um conceito está em um lugar óbvio
