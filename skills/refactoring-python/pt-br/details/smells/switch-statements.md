# SKILL: Detectando e Refatorando Switch Statements — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/switch-statements

---

## 1. O que é Switch Statements

Cadeias complexas de if/elif (ou match/case no Python 3.10+) que ramificam no tipo ou estado de um objeto. A mesma lógica de ramificação tende a ser duplicada pela base de código — quando um novo caso é adicionado, cada branch precisa ser encontrado e atualizado.

**Por que isso acontece:**
- Comportamento baseado em tipo foi implementado de forma procedural em vez de usar polimorfismo
- Uma única classe cresceu para lidar com múltiplas variantes de um conceito
- O conceito de "tipo" foi codificado como uma string ou inteiro em vez de uma hierarquia de classes

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Cadeia `if/elif` ou `match/case` que ramifica em campo de tipo, status ou string
- [ ] A mesma lógica de ramificação aparece em mais de um lugar
- [ ] Adicionar um novo "tipo" requer buscar e atualizar cada branch na base de código
- [ ] Cada branch faz algo muito diferente dos outros
- [ ] Ramificação em tipo para decidir qual objeto instanciar

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                       | Técnica recomendada                    |
|-----------------------------------------------------------|----------------------------------------|
| Ramificação em tipo para decidir comportamento            | Replace Conditional with Polymorphism  |
| Ramificação em tipo para decidir qual classe criar        | Replace Constructor with Factory Method |
| Poucos casos e adicionar novos é raro                     | Replace Type Code with Subclasses      |
| O tipo tem estado mutável que muda em runtime             | Replace Type Code with State/Strategy  |
| Branch simples usado em apenas um lugar                   | Deixe assim — nem todo branch é um smell |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
def calcular_custo_frete(pedido) -> float:
    if pedido.tipo_frete == "padrao":
        return pedido.peso * 1.5
    elif pedido.tipo_frete == "expresso":
        return pedido.peso * 3.0 + 5.0
    elif pedido.tipo_frete == "noturno":
        return pedido.peso * 5.0 + 20.0
    else:
        raise ValueError(f"Tipo de frete desconhecido: {pedido.tipo_frete}")
```

**DEPOIS — esperado:**
```python
from abc import ABC, abstractmethod

class EstrategiaFrete(ABC):
    @abstractmethod
    def calcular_custo(self, peso: float) -> float: ...

class FretePadrao(EstrategiaFrete):
    def calcular_custo(self, peso: float) -> float:
        return peso * 1.5

class FreteExpresso(EstrategiaFrete):
    def calcular_custo(self, peso: float) -> float:
        return peso * 3.0 + 5.0

class FreteNoturno(EstrategiaFrete):
    def calcular_custo(self, peso: float) -> float:
        return peso * 5.0 + 20.0


class Pedido:
    def __init__(self, peso: float, estrategia_frete: EstrategiaFrete):
        self.peso = peso
        self._estrategia_frete = estrategia_frete

    def calcular_custo_frete(self) -> float:
        return self._estrategia_frete.calcular_custo(self.peso)
```

**Por que este padrão:**
- Adicionar um novo tipo de frete requer apenas uma nova classe, não busca e atualização em cada branch
- Cada estratégia é independentemente testável e coesa

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Substituir o branch por dispatch em dict mas manter lógica inline**
```python
# Não aceito — a lógica ainda está espalhada; dispatch em dict só é bom para lookups simples
handlers = {
    "padrao": lambda p: p * 1.5,
    "expresso": lambda p: p * 3.0 + 5.0,
}
```

**Erro 2: Centralizar em uma fábrica sem polimorfismo**
```python
# Não aceito — o branch ainda existe, apenas movido
def criar_calculadora(tipo_frete: str):
    if tipo_frete == "padrao": ...
    elif tipo_frete == "expresso": ...
```

**Erro 3: Aplicar polimorfismo a um branch que nunca vai crescer**
```python
# Não aceito — criar 2 subclasses para um toggle booleano é over-engineering
if pedido.is_prioritario:
    return preco_prioritario
return preco_padrao
```

---

## 6. Benefícios

- **Open/Closed:** Adicionar uma nova variante requer uma nova classe, não editar as existentes
- **Localidade:** O comportamento de cada variante está contido em um único lugar
- **Testabilidade:** Cada variante polimórfica pode ser testada isoladamente
