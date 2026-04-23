# TÉCNICA: Replace Method with Method Object — PHP

## Fonte
Baseado em: https://refactoring.guru/replace-method-with-method-object

---

## 1. Problema

Um método é tão grande e complexo que extrair submétodos fica difícil porque todos compartilhariam muitas variáveis locais. As variáveis locais estão tão entrelaçadas que não podem ser facilmente passadas como parâmetros.

---

## 2. Solução

Transforme o método em uma classe separada. Cada variável local vira uma propriedade dessa classe. O corpo do método vira o método principal `calcular()` da nova classe, que agora pode ser livremente dividido em submétodos sem necessidade de passagem de parâmetros.

---

## 3. Quando aplicar

- O método possui muitas variáveis locais usadas em múltiplas fases lógicas
- Extract Method exigiria passar muitos parâmetros entre os novos métodos
- O algoritmo é complexo o suficiente para merecer uma abstração própria
- Você quer poder testar o algoritmo isoladamente ou sobrescrever partes dele em subclasses

---

## 4. Passos de refatoração

1. Crie uma nova classe com o nome derivado do método
2. Adicione uma propriedade para o objeto que originalmente continha o método
3. Adicione uma propriedade para cada variável local e cada parâmetro do método original
4. Crie um construtor que receba o objeto de origem e todos os parâmetros, atribuindo-os às propriedades
5. Copie o corpo do método original para um método `calcular()`; referências a variáveis locais viram referências a propriedades
6. Substitua o corpo do método original por: `return (new NomeMetodoObjeto($this, $param1))->calcular();`
7. Aplique Extract Method livremente em `calcular()` — sem parâmetros necessários, pois o estado compartilhado vive nas propriedades
8. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
class Pedido {
    public function preco(): float {
        $precoBasePrimario = $this->quantidade * $this->taxaPrimaria();
        $precoBaseSecundario = $precoBasePrimario * 0.7;
        $precoBaseTerciario = max($precoBasePrimario, $precoBaseSecundario);
        return $precoBaseTerciario - $this->desconto($precoBasePrimario, $precoBaseSecundario);
    }
}
```

**DEPOIS — esperado:**
```php
class Pedido {
    public function preco(): float {
        return (new CalculadoraPreco($this))->calcular();
    }
}

class CalculadoraPreco {
    private Pedido $pedido;
    private float $precoBasePrimario;
    private float $precoBaseSecundario;
    private float $precoBaseTerciario;

    public function __construct(Pedido $pedido) {
        $this->pedido = $pedido;
    }

    public function calcular(): float {
        $this->precoBasePrimario = $this->pedido->getQuantidade() * $this->pedido->taxaPrimaria();
        $this->precoBaseSecundario = $this->precoBasePrimario * 0.7;
        $this->precoBaseTerciario = max($this->precoBasePrimario, $this->precoBaseSecundario);
        return $this->precoBaseTerciario - $this->calcularDesconto();
    }

    private function calcularDesconto(): float {
        return $this->pedido->isPremium()
            ? $this->precoBasePrimario * 0.1
            : $this->precoBaseSecundario * 0.05;
    }
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Aplicar quando Extract Method seria suficiente**
```php
// Não aceito — se o método tem apenas 2–3 variáveis locais,
// Extract Method com parâmetros explícitos é mais simples
```

**Erro 2: Nomear a classe de forma vaga**
```php
// Não aceito — Helper, Processador, Calculadora são nomes genéricos demais
class PedidoHelper { ... }
// Prefira: nomeie pelo algoritmo específico
class CalculadoraPrecoPedido { ... }
```

---

## 7. Benefícios

- **Habilita Extract Method:** Submétodos podem compartilhar estado via propriedades sem passar parâmetros
- **Testabilidade:** O objeto de cálculo pode ser instanciado e testado independentemente
- **Extensibilidade:** A classe do objeto método pode ser subclassificada para variar partes do algoritmo
