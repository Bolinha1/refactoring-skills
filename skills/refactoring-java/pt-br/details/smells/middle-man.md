# SMELL: Middle Man — Java

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
| **Replace Delegation with Inheritance** | Se o intermediário sempre delega para o mesmo objeto, considere ter a classe estendendo o delegado (somente quando o relacionamento IS-A é genuíno) |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
// FachadaPedido delega tudo para RepositorioPedido sem valor agregado
public class FachadaPedido {
    private final RepositorioPedido repositorio;

    public FachadaPedido(RepositorioPedido repositorio) {
        this.repositorio = repositorio;
    }

    public Pedido buscarPorId(long id) {
        return repositorio.buscarPorId(id);  // delegação pura
    }

    public void salvar(Pedido pedido) {
        repositorio.salvar(pedido);          // delegação pura
    }

    public List<Pedido> buscarTodos() {
        return repositorio.buscarTodos();    // delegação pura
    }
}

// Cliente usa a fachada desnecessariamente
public class ControladorPedido {
    private final FachadaPedido fachada;

    public Pedido getPedido(long id) {
        return fachada.buscarPorId(id);
    }
}
```

**DEPOIS — esperado:**
```java
// Cliente depende diretamente do repositório
public class ControladorPedido {
    private final RepositorioPedido repositorio;

    public Pedido getPedido(long id) {
        return repositorio.buscarPorId(id);
    }
}
```

**Por que este padrão:**
- `FachadaPedido` não adicionava nenhum comportamento — removê-la simplifica o grafo de dependências
- Uma classe a menos para testar, navegar e manter

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Remover uma fachada que adiciona preocupações transversais**
```java
// Cuidado — se FachadaPedido adiciona logging, cache ou controle de acesso, NÃO é um intermediário
public Pedido buscarPorId(long id) {
    log.info("Buscando pedido {}", id);     // valor real adicionado
    return repositorio.buscarPorId(id);
}
```

**Erro 2: Remover uma fachada usada como seam para testes**
```java
// Cuidado — se testes mocam FachadaPedido para isolar o controlador, ela serve a um propósito;
// considere se injeção de dependência no repositório real é suficiente antes de remover
```

---

## 6. Benefícios

- **Menos camadas:** Menos saltos entre o chamador e o trabalho real
- **Modelo mental mais simples:** Desenvolvedores seguem um nível a menos de indireção
- **Menos manutenção:** Remover delegação desnecessária reduz a superfície a atualizar quando o delegado muda
