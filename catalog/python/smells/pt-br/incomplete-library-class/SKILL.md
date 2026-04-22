# SMELL: Incomplete Library Class — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/incomplete-library-class

---

## 1. O que é?

Uma biblioteca ou classe de terceiros não possui um método que você precisa e, como não é possível (ou recomendável) modificar a biblioteca, você acaba colocando o comportamento ausente em uma função utilitária, um helper estático ou uma subclasse. A lógica que pertence à biblioteca vaza para seu próprio código.

---

## 2. Sinais de alerta

- [ ] Um módulo utilitário contém funções que existem apenas porque uma classe de biblioteca não as possui
- [ ] Você cria uma subclasse de uma classe de biblioteca apenas para adicionar um ou dois métodos ausentes
- [ ] Funções helper recebem um objeto de biblioteca como seu único parâmetro
- [ ] As mesmas operações "ausentes" são reimplementadas em múltiplos módulos
- [ ] Comentários como "deveria estar na biblioteca X" aparecem próximos a funções helper

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Introduce Foreign Method** | Adicione a operação ausente como uma função em seu próprio módulo, recebendo o objeto de biblioteca como primeiro parâmetro |
| **Introduce Local Extension** | Crie uma classe wrapper em torno da classe de biblioteca que adiciona todos os métodos ausentes em um único lugar |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
from datetime import date, timedelta

# Helper espalhado como função standalone
def is_fim_de_semana(d: date) -> bool:
    return d.weekday() >= 5

def proximo_dia_util(d: date) -> date:
    candidato = d + timedelta(days=1)
    while is_fim_de_semana(candidato):
        candidato += timedelta(days=1)
    return candidato

# Uso é estranho
proximo = proximo_dia_util(date.today())
```

**DEPOIS — esperado:**
```python
from datetime import date, timedelta

# Introduce Local Extension: envolva date com o comportamento ausente
class DataUtil:
    def __init__(self, d: date) -> None:
        self._date = d

    @classmethod
    def hoje(cls) -> 'DataUtil':
        return cls(date.today())

    def is_fim_de_semana(self) -> bool:
        return self._date.weekday() >= 5

    def proximo_dia_util(self) -> 'DataUtil':
        candidato = DataUtil(self._date + timedelta(days=1))
        while candidato.is_fim_de_semana():
            candidato = DataUtil(candidato._date + timedelta(days=1))
        return candidato

    def to_date(self) -> date:
        return self._date

# Uso lê naturalmente
proximo = DataUtil.hoje().proximo_dia_util()
```

**Por que este padrão:**
- Toda a lógica de data de negócios vive em `DataUtil` — sem funções utilitárias espalhadas
- Adicionar mais comportamento ausente tem um lugar óbvio

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Duplicar a função ausente em múltiplos módulos utilitários**
```python
# Não aceito — order_utils.is_fim_de_semana(), report_utils.is_fim_de_semana(), etc.
# cria divergência quando a regra precisa mudar
```

**Erro 2: Fazer monkey-patch na classe de biblioteca**
```python
# Não aceito — modificar classes de terceiros em tempo de execução causa bugs sutis
date.is_fim_de_semana = lambda self: self.weekday() >= 5  # monkey-patch
```

---

## 6. Benefícios

- **Única fonte da verdade:** Comportamento ausente vive em uma classe de extensão
- **Pontos de chamada legíveis:** A extensão lê como métodos nativos da classe original
- **Isolado de mudanças da biblioteca:** Wrapping é mais seguro que monkey-patching quando a biblioteca evolui
