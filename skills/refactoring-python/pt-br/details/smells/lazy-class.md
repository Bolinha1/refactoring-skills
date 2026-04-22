# SMELL: Lazy Class — Python

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
- [ ] A classe poderia ser substituída por uma única função em outro lugar
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
```python
# CalculadoraImposto faz apenas uma coisa trivial
class CalculadoraImposto:
    def calcular(self, valor: float) -> float:
        return valor * 0.2


# ServicoPedido apenas delega para CalculadoraImposto
class ServicoPedido:
    def __init__(self):
        self._calculadora_imposto = CalculadoraImposto()

    def total_com_imposto(self, valor: float) -> float:
        return valor + self._calculadora_imposto.calcular(valor)
```

**DEPOIS — esperado:**
```python
_TAXA_IMPOSTO = 0.2

class ServicoPedido:
    def total_com_imposto(self, valor: float) -> float:
        return valor * (1 + _TAXA_IMPOSTO)
```

**Por que este padrão:**
- A lógica de imposto é simples o suficiente para viver diretamente no serviço
- Uma classe a menos significa um arquivo a menos para navegar, testar e manter

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Fazer inline de uma classe que encapsula uma regra mutável**
```python
# Cuidado — se a taxa de imposto difere por país ou categoria de produto,
# CalculadoraImposto pode estar prestes a crescer; verifique o futuro antes de fazer inline
```

**Erro 2: Fazer inline de uma classe compartilhada por múltiplos chamadores**
```python
# Não aceito — se CalculadoraImposto é usada por ServicoPedido, ServicoFatura e
# ServicoOrcamento, fazer inline duplica a lógica em três lugares
```

---

## 6. Benefícios

- **Complexidade reduzida:** Menos classes significa um mapa mental menor do sistema
- **Navegação mais simples:** Desenvolvedores não seguem uma cadeia de delegações para encontrar lógica trivial
- **Menos boilerplate:** Nenhum `__init__`, import ou instanciação necessária para uma classe que quase não faz nada
