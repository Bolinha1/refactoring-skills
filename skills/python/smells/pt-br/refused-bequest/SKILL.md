# SMELL: Refused Bequest — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/refused-bequest

---

## 1. O que é?

Uma subclasse herda métodos ou dados de sua classe pai mas não os utiliza ou os sobrescreve ativamente lançando exceções. O filho rejeita parte do que o pai lhe dá, o que viola o Princípio de Substituição de Liskov: uma subclasse deve ser utilizável onde quer que seu pai seja esperado.

---

## 2. Sinais de alerta

- [ ] Uma subclasse sobrescreve um método pai apenas para lançar `NotImplementedError` ou similar
- [ ] Atributos herdados nunca são lidos ou escritos pela subclasse
- [ ] Testes que esperam comportamento da classe base falham quando recebem a subclasse
- [ ] A subclasse usa apenas uma pequena fração da API pública do pai
- [ ] O relacionamento é "é-um" apenas no papel — comportamentalmente não é

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Replace Inheritance with Delegation** | A subclasse quer algum comportamento do pai mas não todos — componha em vez de herdar |
| **Extract Superclass** | Puxe apenas o comportamento compartilhado para um novo pai; deixe o resto em classes irmãs |
| **Push Down Method / Push Down Field** | Mova membros herdados indesejados para baixo, fora do pai, para que a subclasse nunca os receba |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
class Ave:
    def get_nome(self) -> str:
        return self.nome

    def voar(self) -> None:
        pass  # bater asas

    def pousar(self) -> None:
        pass  # aterrissar


# Pinguim É-UMA Ave no papel, mas não consegue voar
class Pinguim(Ave):
    def voar(self) -> None:
        raise NotImplementedError("Pinguins não podem voar")


# Chamador quebra em tempo de execução
a: Ave = Pinguim()
a.voar()  # lança exceção!
```

**DEPOIS — esperado:**
```python
from abc import ABC, abstractmethod
from typing import Protocol

class Ave(ABC):
    @abstractmethod
    def get_nome(self) -> str: ...


class Voavel(Protocol):
    def voar(self) -> None: ...
    def pousar(self) -> None: ...


class Pardal(Ave):
    def get_nome(self) -> str:
        return "Pardal"

    def voar(self) -> None:
        pass  # bater asas

    def pousar(self) -> None:
        pass  # aterrissar


class Pinguim(Ave):
    def get_nome(self) -> str:
        return "Pinguim"

    def nadar(self) -> None:
        pass  # nadar
```

**Por que este padrão:**
- `Ave` agora contém apenas comportamento compartilhado por TODAS as aves
- `Voavel` é um Protocol estrutural — pinguins simplesmente não o implementam
- Código que chama `voar()` deve aceitar `Voavel`, não apenas qualquer `Ave`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter a exceção e documentá-la**
```python
# Não aceito — o LSP ainda é violado; chamadores não podem confiar no contrato
def voar(self) -> None:
    # Pinguins não voam — documentado mas ainda quebrado
    raise NotImplementedError
```

**Erro 2: Retornar um no-op em vez de lançar exceção**
```python
# Não aceito — falhas silenciosas escondem bugs; chamadores assumem que a ação aconteceu
def voar(self) -> None:
    pass  # não faz nada
```

---

## 6. Benefícios

- **Conformidade com LSP:** Cada subclasse honra o contrato completo de seu pai
- **Polimorfismo mais seguro:** Código que itera uma lista de objetos `Ave` não irá quebrar
- **Hierarquia mais limpa:** Capacidades se tornam Protocols explícitos em vez de exceções ocultas
