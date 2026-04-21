# SMELL: Inappropriate Intimacy — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/inappropriate-intimacy

---

## 1. O que é?

Uma classe acessa os atributos privados, listas internas ou detalhes de implementação de outra classe de forma excessiva. As duas classes estão fortemente acopladas de um modo que dificulta a mudança de uma sem afetar a outra.

---

## 2. Sinais de alerta

- [ ] Uma classe lê múltiplos atributos de outra classe para realizar um cálculo que deveria viver nessa outra classe
- [ ] Uma classe muta diretamente a lista ou estado interno de outra classe
- [ ] Duas classes se referenciam mutuamente em uma dependência bidirecional
- [ ] Uma classe conhece a estrutura interna de outra
- [ ] "Enfileiramentos de trens" repetidos: `a.get_b().get_c().fazer_algo()`

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Move Method** | Mova o método que acessa internos alheios para a classe que possui esses internos |
| **Move Field** | Mova um atributo para a classe que mais o utiliza |
| **Hide Delegate** | Adicione um método de encaminhamento para que o cliente não precise acessar através de um intermediário |
| **Extract Class** | Se duas classes são muito íntimas porque compartilham responsabilidades, separe-as de forma limpa |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
class Pedido:
    def __init__(self, itens: list, email_cliente: str) -> None:
        self.itens = itens             # público — expõe internos
        self.email_cliente = email_cliente

# Relatorio vasculha os internos de Pedido
class RelatorioPedido:
    def gerar(self, pedido: 'Pedido') -> str:
        total = sum(item.preco * item.quantidade for item in pedido.itens)  # acessando internos
        return f"Pedido para {pedido.email_cliente} total: {total}"
```

**DEPOIS — esperado:**
```python
class Pedido:
    def __init__(self, itens: list, email_cliente: str) -> None:
        self._itens = itens
        self._email_cliente = email_cliente

    def total(self) -> float:
        return sum(item.preco * item.quantidade for item in self._itens)

    @property
    def email_cliente(self) -> str:
        return self._email_cliente

# Relatorio apenas pergunta ao Pedido o que precisa
class RelatorioPedido:
    def gerar(self, pedido: 'Pedido') -> str:
        return f"Pedido para {pedido.email_cliente} total: {pedido.total()}"
```

**Por que este padrão:**
- `RelatorioPedido` não depende mais da estrutura interna de `Pedido`
- Mudar como `Pedido` armazena itens não afeta `RelatorioPedido`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Usar uma expressão geradora mas ainda puxando a lista**
```python
# Não aceito — pedido.itens ainda expõe os internos
total = sum(item.preco * item.quantidade for item in pedido.itens)
```

**Erro 2: Criar um dataclass com todos os atributos de Pedido**
```python
# Não aceito — um PedidoData com os mesmos atributos é apenas indireção; o acoplamento permanece
```

---

## 6. Benefícios

- **Menor acoplamento:** Classes podem mudar seus internos sem afetar seus clientes
- **Melhor encapsulamento:** A classe proprietária controla como seus dados são calculados e acessados
- **Mais fácil de testar:** Classes com menos dependências são mais simples de testar em isolamento
