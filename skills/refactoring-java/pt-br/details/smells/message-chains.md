# SMELL: Message Chains — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/message-chains

---

## 1. O que é?

Um cliente pede a um objeto outro objeto, que pede a outro objeto ainda, formando uma cadeia: `a.getB().getC().getD().fazerAlgo()`. O cliente está acoplado a toda a cadeia de intermediários. Se qualquer passo mudar sua estrutura, o cliente quebra.

---

## 2. Sinais de alerta

- [ ] Chamadas de método são encadeadas três ou mais níveis de profundidade: `pedido.getCliente().getEndereco().getCidade()`
- [ ] A mesma cadeia aparece em múltiplos lugares no código
- [ ] É necessário navegar por uma cadeia de objetos apenas para obter um único valor
- [ ] Uma cadeia aparece em um loop ou condicional, amplificando o acoplamento
- [ ] Adicionar uma camada entre dois objetos na cadeia requer atualizar cada ponto de chamada

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Hide Delegate** | Adicione um método de atalho no primeiro objeto da cadeia para que clientes não precisem navegar pelos intermediários |
| **Extract Method** | Extraia a cadeia para um método nomeado que oculte a navegação |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
// Cliente navega por todo o grafo de objetos apenas para obter um nome de cidade
public class ServicoFatura {
    public String getCidadeEntrega(Pedido pedido) {
        return pedido.getCliente().getEndereco().getCidade();
    }

    public boolean isEntregaLocal(Pedido pedido) {
        return pedido.getCliente().getEndereco().getCidade().equals("São Paulo");
    }
}
```

**DEPOIS — esperado:**
```java
// Adicionar um atalho em Pedido (Hide Delegate)
public class Pedido {
    public String getCidadeEntrega() {
        return cliente.getEndereco().getCidade();
    }
}

// Cliente não conhece mais Cliente nem Endereco
public class ServicoFatura {
    public String getCidadeEntrega(Pedido pedido) {
        return pedido.getCidadeEntrega();
    }

    public boolean isEntregaLocal(Pedido pedido) {
        return "São Paulo".equals(pedido.getCidadeEntrega());
    }
}
```

**Por que este padrão:**
- `ServicoFatura` está desacoplado de `Cliente` e `Endereco`
- Renomear `Endereco.getCidade()` para `Endereco.getNomeCidade()` requer apenas uma mudança em vez de muitas

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Atribuir a cadeia a uma variável local sem quebrar a dependência**
```java
// Não aceito — o acoplamento ainda está lá; apenas a sintaxe mudou
Endereco endereco = pedido.getCliente().getEndereco();
String cidade = endereco.getCidade();
```

**Erro 2: Adicionar um método que retorna o objeto intermediário**
```java
// Não aceito — agora os chamadores encadeiam no objeto retornado; mesmo problema, ponto de entrada diferente
public Cliente getCliente() { return this.cliente; }
```

---

## 6. Benefícios

- **Menor acoplamento:** Clientes dependem apenas de seu colaborador direto
- **Refatoração mais fácil:** A navegação interna pode ser alterada sem atualizar cada ponto de chamada
- **Mais legível:** `pedido.getCidadeEntrega()` é mais claro que `pedido.getCliente().getEndereco().getCidade()`
