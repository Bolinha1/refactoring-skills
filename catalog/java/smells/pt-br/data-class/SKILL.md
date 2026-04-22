# SMELL: Data Class — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/data-class

---

## 1. O que é?

Uma classe que contém apenas campos, getters e setters — mas nenhum comportamento real. Toda a lógica que opera sobre esses dados vive em outras classes. A classe é essencialmente um contêiner de dados passivo e age como um registro sem responsabilidade.

---

## 2. Sinais de alerta

- [ ] Uma classe possui apenas campos, getters e setters (e talvez um construtor)
- [ ] A lógica de negócios que deveria pertencer à classe está espalhada por classes de serviço ou gerenciador
- [ ] Outras classes manipulam os campos da classe de dados diretamente através de getters/setters
- [ ] A classe é usada apenas como um saco de parâmetros passado entre métodos
- [ ] Nenhum método existe além de acessores triviais

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Move Method** | Mova o comportamento das classes de serviço para a classe de dados, dando-lhe responsabilidade |
| **Extract Class** | Se a classe de dados cresceu muito, separe um subconjunto coesivo em sua própria classe com comportamento |
| **Hide Method** | Torne os setters privados ou package-private se código externo não deveria modificar o campo diretamente |
| **Encapsulate Field** | Substitua o acesso direto ao campo por getters/setters como primeiro passo para adicionar validação ou lógica |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
// Contêiner de dados puro — sem comportamento
public class Pedido {
    private List<ItemPedido> itens;
    private String status;

    public List<ItemPedido> getItens() { return itens; }
    public void setItens(List<ItemPedido> itens) { this.itens = itens; }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
}

// Toda a lógica vive em um serviço externo
public class ServicoPedido {
    public double calcularTotal(Pedido pedido) {
        return pedido.getItens().stream()
            .mapToDouble(i -> i.getPreco() * i.getQuantidade())
            .sum();
    }

    public boolean podeCancelar(Pedido pedido) {
        return pedido.getStatus().equals("PENDENTE");
    }
}
```

**DEPOIS — esperado:**
```java
public class Pedido {
    private final List<ItemPedido> itens;
    private String status;

    public Pedido(List<ItemPedido> itens) {
        this.itens = List.copyOf(itens);
        this.status = "PENDENTE";
    }

    public double total() {
        return itens.stream()
            .mapToDouble(i -> i.getPreco() * i.getQuantidade())
            .sum();
    }

    public boolean podeCancelar() {
        return "PENDENTE".equals(status);
    }

    public void cancelar() {
        if (!podeCancelar()) throw new IllegalStateException("Não é possível cancelar pedido com status: " + status);
        this.status = "CANCELADO";
    }
}
```

**Por que este padrão:**
- `Pedido` agora possui suas regras de negócio — chamadores perguntam ao próprio pedido
- O campo `status` é protegido: transições acontecem através de métodos de domínio

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover todos os métodos de serviço para a classe de dados sem discriminação**
```java
// Não aceito — a classe se torna uma Large Class; mova apenas comportamento coesivo
public class Pedido {
    public void enviarEmailConfirmacao() { /* não relacionado ao domínio do Pedido */ }
    public void salvarNoBancoDados() { /* preocupação de infraestrutura */ }
}
```

**Erro 2: Manter setters públicos após adicionar lógica de domínio**
```java
// Não aceito — qualquer chamador pode contornar a regra de domínio chamando setStatus() diretamente
public void setStatus(String status) { this.status = status; }
```

---

## 6. Benefícios

- **Encapsulamento:** Regras de negócio vivem próximas aos dados que governam
- **Redução de duplicação:** Lógica que estava copiada entre classes de serviço agora está em um único lugar
- **Diga, não pergunte:** Chamadores pedem ao objeto para fazer algo em vez de ler seus campos e decidir externamente
