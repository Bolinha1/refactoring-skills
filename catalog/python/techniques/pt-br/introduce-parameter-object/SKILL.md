# TÉCNICA: Introduce Parameter Object — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/introduce-parameter-object

---

## 1. Problema

Vários parâmetros que naturalmente pertencem juntos são sempre passados como grupo para as mesmas funções. O grupo representa um conceito ainda não formalizado na base de código.

---

## 2. Solução

Substitua o grupo de parâmetros por um único dataclass (ou classe) que representa o conceito. O objeto se torna um cidadão de primeira classe no modelo de domínio.

---

## 3. Quando aplicar

- Três ou mais parâmetros que sempre aparecem juntos em assinaturas de função
- O mesmo grupo aparece em múltiplas funções pela base de código
- O grupo representa um conceito de domínio reconhecível (intervalo de datas, endereço, coordenadas)
- Você quer adicionar comportamento (validação, formatação) ao grupo de dados

---

## 4. Passos de refatoração

1. Crie um novo dataclass ou classe para representar o grupo de parâmetros
2. Adicione os parâmetros como campos — use `frozen=True` se o objeto representa um valor
3. Adicione `__post_init__` ou `__init__` para validar invariantes
4. Atualize cada função que recebe o grupo de parâmetros para aceitar a nova classe
5. Atualize todos os locais de chamada para construir o novo objeto e passá-lo
6. Procure por lógica nos chamadores que poderia ser movida para dentro da nova classe como método
7. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
def buscar_vendas_por_periodo(data_inicio: date, data_fim: date) -> list: ...
def somar_receita_por_periodo(data_inicio: date, data_fim: date) -> float: ...
def faturas_por_periodo(data_inicio: date, data_fim: date) -> list: ...
```

**DEPOIS — esperado:**
```python
from dataclasses import dataclass
from datetime import date

@dataclass(frozen=True)
class Periodo:
    inicio: date
    fim: date

    def __post_init__(self):
        if self.fim < self.inicio:
            raise ValueError("fim deve ser após início")

    def inclui(self, d: date) -> bool:
        return self.inicio <= d <= self.fim


def buscar_vendas_por_periodo(periodo: Periodo) -> list: ...
def somar_receita_por_periodo(periodo: Periodo) -> float: ...
def faturas_por_periodo(periodo: Periodo) -> list: ...
```

**Por que este padrão:**
- `Periodo` valida seu próprio invariante uma vez — chamadores não podem passar um período inválido
- O método `inclui()` captura lógica de consulta que antes era duplicada em cada chamador

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Criar o objeto mas não mover comportamento para ele**
```python
# Não aceito — Periodo ainda é apenas um recipiente de duas datas sem comportamento
@dataclass
class Periodo:
    inicio: date
    fim: date
```

**Erro 2: Tornar o objeto mutável quando representa um valor**
```python
# Não aceito — um Periodo mutável pode ser alterado pelos chamadores após passá-lo
class Periodo:
    def set_inicio(self, d: date): self.inicio = d  # ruim — valores deveriam ser imutáveis
```

**Erro 3: Agrupar parâmetros não relacionados só para reduzir a contagem**
```python
# Não aceito — ContextoRequisicao agrupa id_cliente, id_sessao e locale
# sem razão além de serem três
@dataclass
class ContextoRequisicao:
    id_cliente: int
    id_sessao: str
    locale: str
```

---

## 7. Benefícios

- **Expressividade:** O grupo de parâmetros recebe um nome significativo no domínio
- **Validação:** O objeto valida seus próprios invariantes uma vez, no momento da construção
- **Migração de comportamento:** Lógica que trabalha com o grupo pode ser movida para dentro do objeto
