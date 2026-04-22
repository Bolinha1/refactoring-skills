# TÉCNICA: Replace Method with Method Object — Python

## Fonte
Baseado em: https://refactoring.guru/replace-method-with-method-object

---

## 1. Problema

Um método é tão grande e complexo que extrair submétodos fica difícil porque todos compartilhariam muitas variáveis locais. As variáveis locais estão tão entrelaçadas que não podem ser facilmente passadas como parâmetros.

---

## 2. Solução

Transforme o método em uma classe separada. Cada variável local vira um atributo de instância dessa classe. O corpo do método vira o método principal `calcular()` da nova classe, que agora pode ser livremente dividido em submétodos sem necessidade de passagem de parâmetros.

---

## 3. Quando aplicar

- O método possui muitas variáveis locais usadas em múltiplas fases lógicas
- Extract Method exigiria passar muitos parâmetros entre os novos métodos
- O algoritmo é complexo o suficiente para merecer uma abstração própria
- Você quer poder testar o algoritmo isoladamente ou sobrescrever partes dele em subclasses

---

## 4. Passos de refatoração

1. Crie uma nova classe com o nome derivado do método
2. Implemente `__init__` que aceite o objeto de origem e todos os parâmetros do método original
3. Armazene cada parâmetro e variável local como atributo de instância
4. Copie o corpo do método original para um método `calcular()`; referências a variáveis locais viram referências `self.`
5. Substitua o corpo do método original por: `return NomeMetodoObjeto(self, param1).calcular()`
6. Aplique Extract Method livremente em `calcular()` — sem parâmetros necessários, pois o estado compartilhado vive em `self`
7. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
class Pedido:
    def preco(self) -> float:
        preco_base_primario = self.quantidade * self._taxa_primaria()
        preco_base_secundario = preco_base_primario * 0.7
        preco_base_terciario = max(preco_base_primario, preco_base_secundario)
        return preco_base_terciario - self._desconto(preco_base_primario, preco_base_secundario)
```

**DEPOIS — esperado:**
```python
class Pedido:
    def preco(self) -> float:
        return CalculadoraPreco(self).calcular()


class CalculadoraPreco:
    def __init__(self, pedido: "Pedido") -> None:
        self.pedido = pedido
        self.preco_base_primario = 0.0
        self.preco_base_secundario = 0.0
        self.preco_base_terciario = 0.0

    def calcular(self) -> float:
        self.preco_base_primario = self.pedido.quantidade * self.pedido._taxa_primaria()
        self.preco_base_secundario = self.preco_base_primario * 0.7
        self.preco_base_terciario = max(self.preco_base_primario, self.preco_base_secundario)
        return self.preco_base_terciario - self._calcular_desconto()

    def _calcular_desconto(self) -> float:
        if self.pedido.is_premium:
            return self.preco_base_primario * 0.1
        return self.preco_base_secundario * 0.05
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Aplicar quando Extract Method seria suficiente**
```python
# Não aceito — se o método tem apenas 2–3 variáveis locais,
# Extract Method com parâmetros explícitos é mais simples
```

**Erro 2: Nomear a classe de forma vaga**
```python
# Não aceito — Helper, Processador, Calculadora são nomes genéricos demais
class PedidoHelper: ...
# Prefira: nomeie pelo algoritmo específico
class CalculadoraPrecoPedido: ...
```

---

## 7. Benefícios

- **Habilita Extract Method:** Submétodos podem compartilhar estado via `self` sem passar parâmetros
- **Testabilidade:** O objeto de cálculo pode ser instanciado e testado independentemente
- **Extensibilidade:** A classe do objeto método pode ser subclassificada para variar partes do algoritmo
