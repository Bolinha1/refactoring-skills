# TÉCNICA: Replace Method with Method Object — Java

## Fonte
Baseado em: https://refactoring.guru/replace-method-with-method-object

---

## 1. Problema

Um método é tão grande e complexo que extrair submétodos fica difícil porque todos compartilhariam muitas variáveis locais. As variáveis locais estão tão entrelaçadas que não podem ser facilmente passadas como parâmetros.

---

## 2. Solução

Transforme o método em uma classe separada. Cada variável local vira um campo dessa classe. O corpo do método vira o método principal `calcular()` da nova classe, que agora pode ser livremente dividido em submétodos sem necessidade de passagem de parâmetros.

---

## 3. Quando aplicar

- O método possui muitas variáveis locais usadas em múltiplas fases lógicas
- Extract Method exigiria passar muitos parâmetros entre os novos métodos
- O algoritmo é complexo o suficiente para merecer uma abstração própria
- Você quer poder testar o algoritmo isoladamente ou sobrescrever partes dele em subclasses

---

## 4. Passos de refatoração

1. Crie uma nova classe com o nome derivado do método
2. Adicione um campo para o objeto que originalmente continha o método (`origem`)
3. Adicione um campo para cada variável local e cada parâmetro do método original
4. Crie um construtor que receba o objeto de origem e todos os parâmetros, atribuindo-os aos campos
5. Copie o corpo do método original para um método `calcular()`; referências a variáveis locais viram referências a campos
6. Substitua o corpo do método original por: `return new NomeMetodoObjeto(this, param1, param2).calcular();`
7. Aplique Extract Method livremente em `calcular()` — sem parâmetros necessários, pois o estado compartilhado vive nos campos
8. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public class Pedido {
    public double preco() {
        double precoBasePrimario;
        double precoBaseSecundario;
        double precoBaseTerciario;
        // cálculo longo e complexo envolvendo todas as três variáveis
        precoBasePrimario = quantidade * taxaPrimaria();
        precoBaseSecundario = precoBasePrimario * 0.7;
        precoBaseTerciario = Math.max(precoBasePrimario, precoBaseSecundario);
        return precoBaseTerciario - desconto(precoBasePrimario, precoBaseSecundario);
    }
}
```

**DEPOIS — esperado:**
```java
public class Pedido {
    public double preco() {
        return new CalculadoraPreco(this).calcular();
    }
}

class CalculadoraPreco {
    private final Pedido pedido;
    private double precoBasePrimario;
    private double precoBaseSecundario;
    private double precoBaseTerciario;

    CalculadoraPreco(Pedido pedido) {
        this.pedido = pedido;
    }

    double calcular() {
        precoBasePrimario = pedido.getQuantidade() * pedido.taxaPrimaria();
        precoBaseSecundario = precoBasePrimario * 0.7;
        precoBaseTerciario = Math.max(precoBasePrimario, precoBaseSecundario);
        return precoBaseTerciario - calcularDesconto();
    }

    private double calcularDesconto() {
        return pedido.isPremium()
            ? precoBasePrimario * 0.1
            : precoBaseSecundario * 0.05;
    }
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Aplicar quando Extract Method seria suficiente**
```java
// Não aceito — se o método tem apenas 2–3 variáveis locais,
// Extract Method com parâmetros explícitos é mais simples
```

**Erro 2: Nomear a classe de forma vaga**
```java
// Não aceito — Helper, Processador, Calculadora são nomes genéricos demais
class PedidoHelper { ... }
// Prefira: nomeie pelo algoritmo específico
class CalculadoraPrecoPedido { ... }
```

---

## 7. Benefícios

- **Habilita Extract Method:** Submétodos podem compartilhar estado via campos sem passar parâmetros
- **Testabilidade:** O objeto de cálculo pode ser instanciado e testado independentemente
- **Extensibilidade:** A classe do objeto método pode ser subclassificada para variar partes do algoritmo
