# SKILL: Detectando e Refatorando Código Duplicado — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/duplicate-code

---

## 1. O que é Código Duplicado

Dois ou mais fragmentos de código são quase idênticos ou estruturalmente similares em partes diferentes da base de código. Qualquer mudança na lógica precisa ser replicada em cada cópia, e esquecer uma delas cria um bug silencioso.

**Por que isso acontece:**
- Copy-paste usado como atalho no lugar de abstração
- Desenvolvimento paralelo onde dois desenvolvedores resolveram o mesmo problema de forma independente
- Lógica que começou similar e divergiu ligeiramente, tornando a duplicação invisível

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Mesmo bloco de código (ou quase idêntico) em dois ou mais métodos/funções
- [ ] Mesma computação ou expressão repetida em vários lugares
- [ ] Duas classes contêm métodos que fazem essencialmente a mesma coisa
- [ ] Duas subclasses implementam métodos com o mesmo corpo
- [ ] Você copiou e colou código e ajustou apenas uma variável

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                     | Técnica recomendada               |
|---------------------------------------------------------|-----------------------------------|
| Blocos duplicados na mesma classe                       | Extract Method                    |
| Blocos duplicados em subclasses irmãs                   | Pull Up Method                    |
| Blocos similares mas não idênticos em subclasses irmãs  | Form Template Method              |
| Código duplicado em duas classes não relacionadas       | Extract Class ou Move Method      |
| Métodos longos duplicados que diferem apenas no algoritmo | Substitute Algorithm            |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
class RelatorioVendas:
    def calcular_receita(self, pedidos: list) -> float:
        total = 0
        for pedido in pedidos:
            if pedido.status == "pago":
                total += pedido.valor
        return total


class RelatorioFinanceiro:
    def calcular_total_pedidos_pagos(self, pedidos: list) -> float:
        total = 0
        for pedido in pedidos:
            if pedido.status == "pago":
                total += pedido.valor
        return total
```

**DEPOIS — esperado:**
```python
def somar_pedidos_pagos(pedidos: list) -> float:
    return sum(pedido.valor for pedido in pedidos if pedido.status == "pago")


class RelatorioVendas:
    def calcular_receita(self, pedidos: list) -> float:
        return somar_pedidos_pagos(pedidos)


class RelatorioFinanceiro:
    def calcular_total_pedidos_pagos(self, pedidos: list) -> float:
        return somar_pedidos_pagos(pedidos)
```

**Por que este padrão:**
- A lógica de negócio vive em um único lugar — uma única mudança corrige todos os chamadores
- Ambas as classes de relatório delegam para o utilitário compartilhado em vez de possuir a lógica

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Aceitar pequenas diferenças como justificativa para duplicação**
```python
# Não aceito — "quase igual" ainda significa duplicado
def calcular_receita(self, pedidos): ...         # filtra pagos
def calcular_receita_projetada(self, pedidos): ...  # mesmo loop, um multiplicador extra
```

**Erro 2: Extrair para método privado mas duplicar o método em cada classe**
```python
# Não aceito — _somar_pagos está duplicado em RelatorioVendas e RelatorioFinanceiro
def _somar_pagos(self, pedidos): ...
```

**Erro 3: Fundir duplicatas em uma função deus com flags booleanas**
```python
# Não aceito — cria Feature Envy / Long Method
def calcular(pedidos, pago=False, projetado=False, com_impostos=False): ...
```

---

## 6. Benefícios

- **Manutenção:** Uma única correção de bug ou mudança de regra se aplica em todos os lugares automaticamente
- **Clareza:** Código que expressa um conceito compartilhado é mais fácil de entender do que cópias espalhadas
- **Confiança:** Desenvolvedores podem ter certeza de que não há cópias divergentes escondendo lógica obsoleta
