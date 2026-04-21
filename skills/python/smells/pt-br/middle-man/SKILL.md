# SMELL: Middle Man — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/middle-man

---

## 1. O que é?

Uma classe cujo único propósito é delegar cada chamada a outra classe. Ela adiciona uma camada de indireção sem agregar nenhum valor. Este é o inverso de Feature Envy: em vez de uma classe que alcança outras, esta é uma classe pela qual outros alcançam desnecessariamente.

---

## 2. Sinais de alerta

- [ ] Mais da metade dos métodos de uma classe apenas delegam para outra classe sem lógica adicional
- [ ] Remover a classe e chamar o delegado diretamente simplificaria o código
- [ ] A classe foi introduzida como uma fachada mas a fachada agora não faz nada útil
- [ ] A classe não tem estado próprio e todos os métodos são pass-throughs

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Remove Middle Man** | Deixe os clientes chamarem o delegado diretamente e delete a classe intermediária |
| **Inline Class** | Quando o intermediário é pequeno, absorva-o em seu chamador |
| **Replace Delegation with Inheritance** | Se o intermediário sempre delega para o mesmo objeto, considere herança (somente quando IS-A é genuíno) |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
# FachadaPedido delega tudo para RepositorioPedido sem valor agregado
class FachadaPedido:
    def __init__(self, repositorio: 'RepositorioPedido') -> None:
        self._repositorio = repositorio

    def buscar_por_id(self, pedido_id: int):
        return self._repositorio.buscar_por_id(pedido_id)   # delegação pura

    def salvar(self, pedido) -> None:
        self._repositorio.salvar(pedido)                    # delegação pura

    def buscar_todos(self) -> list:
        return self._repositorio.buscar_todos()             # delegação pura


# Cliente usa a fachada desnecessariamente
class ControladorPedido:
    def __init__(self, fachada: FachadaPedido) -> None:
        self._fachada = fachada

    def get_pedido(self, pedido_id: int):
        return self._fachada.buscar_por_id(pedido_id)
```

**DEPOIS — esperado:**
```python
# Cliente depende diretamente do repositório
class ControladorPedido:
    def __init__(self, repositorio: 'RepositorioPedido') -> None:
        self._repositorio = repositorio

    def get_pedido(self, pedido_id: int):
        return self._repositorio.buscar_por_id(pedido_id)
```

**Por que este padrão:**
- `FachadaPedido` não adicionava nenhum comportamento — removê-la simplifica o grafo de dependências
- Uma classe a menos para testar, navegar e manter

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Remover uma fachada que adiciona preocupações transversais**
```python
# Cuidado — se FachadaPedido adiciona logging, cache ou controle de acesso, NÃO é um intermediário
def buscar_por_id(self, pedido_id: int):
    self._logger.info(f"Buscando pedido {pedido_id}")   # valor real adicionado
    return self._repositorio.buscar_por_id(pedido_id)
```

**Erro 2: Remover uma fachada usada como seam para testes**
```python
# Cuidado — se testes mocam FachadaPedido para isolar o controlador, ela serve a um propósito
```

---

## 6. Benefícios

- **Menos camadas:** Menos saltos entre o chamador e o trabalho real
- **Modelo mental mais simples:** Desenvolvedores seguem um nível a menos de indireção
- **Menos manutenção:** Remover delegação desnecessária reduz a superfície a atualizar quando o delegado muda
