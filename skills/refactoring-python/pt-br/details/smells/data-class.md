# SMELL: Data Class — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/data-class

---

## 1. O que é?

Uma classe que contém apenas atributos e acessores — mas nenhum comportamento real. Toda a lógica que opera sobre esses dados vive em outras classes. A classe é essencialmente um contêiner de dados passivo e age como um registro sem responsabilidade.

---

## 2. Sinais de alerta

- [ ] Uma classe possui apenas atributos e talvez `__init__` sem métodos
- [ ] A lógica de negócios que deveria pertencer à classe está espalhada por classes de serviço ou gerenciador
- [ ] Outras classes manipulam os atributos da classe de dados diretamente
- [ ] A classe é usada apenas como um saco de parâmetros passado entre funções/métodos
- [ ] A classe é um dataclass puro (`@dataclass`) mas todas as operações sobre ela vivem em outro lugar

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Move Method** | Mova o comportamento das classes de serviço para a classe de dados, dando-lhe responsabilidade |
| **Extract Class** | Se a classe de dados cresceu muito, separe um subconjunto coesivo em sua própria classe com comportamento |
| **Hide Method** | Use properties com apenas getter (sem setter) para proteger atributos de mutação externa |
| **Encapsulate Field** | Substitua o acesso direto ao atributo por properties como primeiro passo para adicionar validação ou lógica |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
from dataclasses import dataclass, field

@dataclass
class Pedido:
    itens: list['ItemPedido'] = field(default_factory=list)
    status: str = 'PENDENTE'

# Toda a lógica vive em um serviço externo
class ServicoPedido:
    def calcular_total(self, pedido: Pedido) -> float:
        return sum(item.preco * item.quantidade for item in pedido.itens)

    def pode_cancelar(self, pedido: Pedido) -> bool:
        return pedido.status == 'PENDENTE'
```

**DEPOIS — esperado:**
```python
class Pedido:
    def __init__(self, itens: list['ItemPedido']) -> None:
        self._itens = list(itens)
        self._status = 'PENDENTE'

    def total(self) -> float:
        return sum(item.preco * item.quantidade for item in self._itens)

    def pode_cancelar(self) -> bool:
        return self._status == 'PENDENTE'

    def cancelar(self) -> None:
        if not self.pode_cancelar():
            raise ValueError(f"Não é possível cancelar pedido com status: {self._status}")
        self._status = 'CANCELADO'
```

**Por que este padrão:**
- `Pedido` agora possui suas regras de negócio — chamadores perguntam ao próprio pedido
- O atributo `_status` é protegido: transições acontecem através de métodos de domínio

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover todos os métodos de serviço para a classe de dados sem discriminação**
```python
# Não aceito — a classe se torna uma Large Class; mova apenas comportamento coesivo
class Pedido:
    def enviar_email_confirmacao(self): ...  # não relacionado ao domínio do Pedido
    def salvar_no_banco(self): ...           # preocupação de infraestrutura
```

**Erro 2: Manter mutação pública do atributo após adicionar lógica de domínio**
```python
# Não aceito — qualquer chamador pode contornar a regra de domínio definindo pedido.status diretamente
pedido.status = 'CANCELADO'  # ignora a verificação pode_cancelar()
```

---

## 6. Benefícios

- **Encapsulamento:** Regras de negócio vivem próximas aos atributos que governam
- **Redução de duplicação:** Lógica que estava copiada entre classes de serviço agora está em um único lugar
- **Diga, não pergunte:** Chamadores pedem ao objeto para fazer algo em vez de ler seus atributos e decidir externamente
