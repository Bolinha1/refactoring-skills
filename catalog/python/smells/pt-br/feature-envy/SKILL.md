# SKILL: Detectando e Refatorando Feature Envy — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/feature-envy

---

## 1. O que é Feature Envy

Um método acessa os dados de outro objeto mais do que acessa os dados da sua própria classe. O método tem "inveja" da outra classe — parece querer viver lá.

**Por que isso acontece:**
- Atributos foram movidos para uma nova classe mas os métodos que os usam não foram
- Lógica de negócio foi colocada em uma classe utilitária que opera nos internos de um objeto de domínio
- Um método foi gradualmente estendido para depender cada vez mais dos atributos de outro objeto

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Um método acessa vários atributos de outro objeto em sequência
- [ ] Um método recebe um objeto como parâmetro e usa principalmente seus atributos/métodos
- [ ] Um método ignora completamente o `self` e só manipula o estado de outro objeto
- [ ] O nome do método naturalmente pertence à outra classe ("calcular_total_pedido" em ServiçoRelatorio)

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
```python
class ImpressoraFatura:
    def calcular_total_fatura(self, fatura) -> float:
        subtotal = sum(item.preco * item.quantidade for item in fatura.itens)
        imposto = subtotal * fatura.taxa_imposto
        desconto = subtotal * fatura.taxa_desconto if fatura.tem_desconto else 0
        return subtotal + imposto - desconto
```

**DEPOIS — esperado:**
```python
class Fatura:
    def calcular_total(self) -> float:
        subtotal = sum(item.preco * item.quantidade for item in self.itens)
        imposto = subtotal * self.taxa_imposto
        desconto = subtotal * self.taxa_desconto if self.tem_desconto else 0
        return subtotal + imposto - desconto


class ImpressoraFatura:
    def imprimir(self, fatura: Fatura) -> None:
        total = fatura.calcular_total()  # delega ao dono
        # ... lógica de impressão
```

**Por que este padrão:**
- `calcular_total` naturalmente pertence à `Fatura` — só usa dados da Fatura
- `ImpressoraFatura` é liberada de conhecer os internos da Fatura

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter o método invejoso como um wrapper fino**
```python
# Não aceito — apenas um wrapper; delete o método invejoso completamente
def calcular_total_fatura(self, fatura) -> float:
    return fatura.calcular_total()
```

**Erro 2: Mover o método mas manter parâmetros de campo brutos**
```python
# Não aceito — movido mas ainda recebendo valores de campo brutos
def calcular_total(self, subtotal: float, taxa_imposto: float, taxa_desconto: float, tem_desconto: bool) -> float: ...
```

**Erro 3: Mover um método que usa dados de muitas classes igualmente**
```python
# Não aceito — se o método usa Pedido, Cliente e Cupom igualmente,
# movê-lo para qualquer uma das classes apenas desloca a inveja; Extract Class é melhor
```

---

## 6. Benefícios

- **Coesão:** Cada classe possui o comportamento que opera sobre seus próprios dados
- **Encapsulamento:** O objeto de domínio expõe intenção, não atributos brutos
- **Manutenibilidade:** As regras de negócio são colocalizadas com os dados que governam
