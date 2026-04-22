# SMELL: Speculative Generality — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/speculative-generality

---

## 1. O que é?

Código escrito para lidar com requisitos que ainda não existem — e talvez nunca existam. Classes base abstratas com apenas uma subclasse concreta, hooks que nunca são chamados, parâmetros que são sempre passados como `None` e protocols com uma única implementação são todos sinais de que alguém construiu flexibilidade "por precaução".

---

## 2. Sinais de alerta

- [ ] Uma ABC ou Protocol tem apenas uma implementação concreta
- [ ] Uma função tem parâmetros que são sempre `None` ou a mesma constante em todos os pontos de chamada
- [ ] Uma classe ou função existe apenas para suportar requisitos futuros mencionados em comentários
- [ ] Um hook ou ponto de extensão nunca foi usado desde que foi introduzido
- [ ] Testes são os únicos chamadores de certas funções

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Collapse Hierarchy** | Quando uma ABC tem apenas uma subclasse concreta e elas podem ser mescladas |
| **Inline Class** | Quando uma classe existe puramente como ponto de extensão futuro sem consumidores atuais |
| **Remove Parameter** | Quando um parâmetro é sempre passado com o mesmo valor em todos os pontos de chamada |
| **Rename Method** | Quando nomes abstratos foram escolhidos para acomodar uma variedade que nunca se materializou |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
from abc import ABC, abstractmethod

# NotificadorAbstrato existe apenas porque alguém assumiu que outros notificadores viriam
class NotificadorAbstrato(ABC):
    @abstractmethod
    def enviar(self, mensagem: str, canal: str) -> None: ...


class NotificadorEmail(NotificadorAbstrato):
    def enviar(self, mensagem: str, canal: str) -> None:
        # canal é sempre "email" em todos os pontos de chamada
        self._cliente_email.enviar(mensagem)


# Todo ponto de chamada passa "email" — o parâmetro não agrega valor
notificador.enviar("Olá", "email")
```

**DEPOIS — esperado:**
```python
class NotificadorEmail:
    def enviar(self, mensagem: str) -> None:
        self._cliente_email.enviar(mensagem)


notificador.enviar("Olá")
```

**Por que este padrão:**
- A ABC e o parâmetro não utilizado eram complexidade especulativa
- Se um segundo tipo de notificador for necessário, a abstração pode ser introduzida com requisitos reais

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Remover abstrações que estão genuinamente abertas para extensão**
```python
# Cuidado — se o roadmap já especifica SMS e push, a abstração não é especulativa
```

**Erro 2: Remover uma ABC apenas porque tem uma implementação hoje**
```python
# Cuidado — ABCs usadas para injeção de dependência ou testes servem a um propósito real
```

---

## 6. Benefícios

- **Complexidade reduzida:** O código contém apenas código que atende às necessidades atuais
- **Integração mais fácil:** Desenvolvedores não precisam entender hooks e abstrações que ainda não fazem nada
- **Refatoração mais simples:** Quando requisitos reais chegam, você adiciona abstrações informadas por casos de uso reais
