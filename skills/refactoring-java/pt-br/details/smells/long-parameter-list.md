# SKILL: Detectando e Refatorando Long Parameter List — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/long-parameter-list

---

## 1. O que é Long Parameter List

Um método tem parâmetros demais — tipicamente mais de três ou quatro. Listas longas de parâmetros são difíceis de entender, fáceis de confundir e dolorosas de chamar. Elas frequentemente indicam que dados relacionados deveriam ser agrupados em um objeto.

**Por que isso acontece:**
- Métodos foram mesclados sem criar uma abstração adequada para os dados combinados
- Algoritmos foram ficando mais complexos ao longo do tempo, exigindo mais flags de controle
- Dados que naturalmente pertencem juntos são passados como primitivos individuais

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Assinatura de método com 4 ou mais parâmetros
- [ ] Vários parâmetros do mesmo tipo em sequência (fácil de trocar acidentalmente)
- [ ] Parâmetros que sempre aparecem juntos em vários locais de chamada
- [ ] Parâmetros booleanos "flag" que mudam o comportamento do método
- [ ] O chamador precisa construir muitas variáveis locais só para chamar o método

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                      | Técnica recomendada                |
|----------------------------------------------------------|------------------------------------|
| Parâmetros são todos campos de um objeto existente       | Preserve Whole Object              |
| Parâmetros representam um conceito novo não no modelo    | Introduce Parameter Object         |
| Um parâmetro pode ser computado a partir de outros       | Replace Parameter with Method Call |
| Um flag booleano muda o comportamento do método          | Dividir em dois métodos explícitos |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
public class ServiçoPedido {
    public Recibo criarPedido(
            String idCliente,
            String nomeCliente,
            String emailCliente,
            String rua,
            String cidade,
            String cep,
            String pais,
            List<String> idsProdutos,
            String codigoCupom) {
        // ... lógica longa
    }
}
```

**DEPOIS — esperado:**
```java
public class Endereco {
    private final String rua;
    private final String cidade;
    private final String cep;
    private final String pais;
    // construtor + getters
}

public class RequisicaoPedido {
    private final String idCliente;
    private final String nomeCliente;
    private final String emailCliente;
    private final Endereco enderecoEntrega;
    private final List<String> idsProdutos;
    private final String codigoCupom;
    // construtor + getters
}

public class ServiçoPedido {
    public Recibo criarPedido(RequisicaoPedido requisicao) {
        // ... lógica usa requisicao.getIdCliente(), requisicao.getEnderecoEntrega(), etc.
    }
}
```

**Por que este padrão:**
- `RequisicaoPedido` e `Endereco` são conceitos de primeira classe — podem ser validados, reutilizados e testados de forma independente
- Os chamadores constroem um objeto significativo, não uma lista de argumentos posicionais

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Agrupar parâmetros não relacionados em um objeto só para reduzir a contagem**
```java
// Não aceito — DataHolder não tem significado de domínio
public Recibo criarPedido(DataHolder holder) { ... }
```

**Erro 2: Usar Map<String, Object> como "objeto de parâmetro"**
```java
// Não aceito — perde segurança de tipos, suporte de IDE e descobribilidade
public Recibo criarPedido(Map<String, Object> params) { ... }
```

**Erro 3: Manter a lista longa mas adicionar sobrecargas com valores padrão**
```java
// Não aceito — multiplica o problema sem abordar a causa
public Recibo criarPedido(String idCliente, String nome) { ... }
public Recibo criarPedido(String idCliente, String nome, String email) { ... }
public Recibo criarPedido(String idCliente, String nome, String email, Endereco endereco) { ... }
```

---

## 6. Benefícios

- **Legibilidade:** Objetos de parâmetro nomeados tornam os locais de chamada autodocumentados
- **Segurança:** A confusão de argumentos posicionais é eliminada
- **Extensibilidade:** Adicionar um campo ao objeto de parâmetro não muda cada local de chamada
