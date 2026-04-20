# SKILL: Detecção e Refatoração de Primitive Obsession — Java

## Fonte
Baseado em: https://refactoring.guru/smells/primitive-obsession

---

## 1. O que é Primitive Obsession

Uso excessivo de tipos primitivos (`String`, `int`, `double`, `boolean`) para
representar conceitos de domínio que mereceriam classes próprias.

**Por que isso acontece:**
- Criar uma nova classe parece exagero para algo "simples"
- Primitivos são fáceis e rápidos de usar no início
- O código cresce e o primitivo vira um campo mágico difícil de rastrear

---

## 2. Sinais de alerta (gatilhos para acionar este SKILL)

- [ ] `String` representando CPF, telefone, CEP, e-mail, código de produto
- [ ] `int` ou `String` constante simulando um tipo enumerado (ex: `ROLE_ADMIN = 1`)
- [ ] Múltiplos parâmetros primitivos que sempre aparecem juntos (ex: `double valor, String moeda`)
- [ ] Array ou `Map` com índices/chaves mágicas de String para estruturar dados
- [ ] Validação do mesmo primitivo repetida em vários lugares

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                  | Técnica indicada               |
|------------------------------------------------------|-------------------------------|
| Primitivo com regras de validação próprias            | Replace Data Value with Object |
| Múltiplos primitivos que andam juntos                | Introduce Parameter Object     |
| Primitivo passado como inteiro grupo                 | Preserve Whole Object          |
| `int`/`String` simulando tipo enumerado              | Replace Type Code with Class   |
| Código condicional baseado no tipo simulado          | Replace Type Code with Subclasses / State/Strategy |
| `Object[]` ou `Map<String, Object>` como estrutura  | Replace Array with Object      |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
public class Cliente {
    private String nome;
    private String cpf;          // string crua, sem validação centralizada
    private String telefone;     // string crua
    private String email;        // string crua
}

public class PedidoService {
    public void criar(String cpfCliente, double valor, String moeda, int tipoPagamento) {
        // tipoPagamento: 1 = cartão, 2 = boleto, 3 = pix — constante mágica
        if (tipoPagamento == 1) {
            // ...
        }
        // validação de CPF duplicada em vários pontos
        if (cpfCliente == null || cpfCliente.length() != 11) {
            throw new IllegalArgumentException("CPF inválido");
        }
    }
}
```

**DEPOIS — esperado:**
```java
// Value Objects para primitivos com semântica de domínio
public final class Cpf {
    private final String valor;

    public Cpf(String valor) {
        if (valor == null || !valor.matches("\\d{11}")) {
            throw new IllegalArgumentException("CPF inválido");
        }
        this.valor = valor;
    }

    public String getValor() { return valor; }
}

public final class Email {
    private final String endereco;

    public Email(String endereco) {
        if (endereco == null || !endereco.contains("@")) {
            throw new IllegalArgumentException("E-mail inválido");
        }
        this.endereco = endereco;
    }
}

// Parameter Object para dados que andam juntos
public class Dinheiro {
    private final double valor;
    private final String moeda;

    public Dinheiro(double valor, String moeda) {
        this.valor = valor;
        this.moeda = moeda;
    }
}

// Enum no lugar de constante inteira
public enum TipoPagamento {
    CARTAO, BOLETO, PIX
}

public class Cliente {
    private String nome;
    private Cpf cpf;
    private Email email;
}

public class PedidoService {
    public void criar(Cpf cpfCliente, Dinheiro valor, TipoPagamento tipoPagamento) {
        if (tipoPagamento == TipoPagamento.CARTAO) {
            // ...
        }
    }
}
```

**Por que esse padrão:**
- A validação do CPF está centralizada no `Cpf` — não se repete
- `TipoPagamento` é autoexplicativo, sem inteiros mágicos
- `Dinheiro` agrupa valor e moeda — evita que andem separados

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: String para tudo**
```java
// Não aceito
public void processar(String cpf, String status, String tipoPagamento) {
    if (status.equals("ATIVO") && tipoPagamento.equals("PIX")) { ... }
}
```

**Erro 2: Constantes inteiras no lugar de enum**
```java
// Não aceito
public static final int PAGAMENTO_CARTAO = 1;
public static final int PAGAMENTO_BOLETO = 2;
if (tipo == PAGAMENTO_CARTAO) { ... }
```

**Erro 3: Array com índices mágicos**
```java
// Não aceito
String[] endereco = new String[4];
endereco[0] = "Rua A";
endereco[1] = "123";
endereco[2] = "SP";
endereco[3] = "01310-100";
```

---

## 6. Benefícios

- **Flexibilidade:** Regras de negócio ficam encapsuladas no objeto correto
- **Legibilidade:** Parâmetros e campos expressam intenção de domínio
- **Manutenção:** Validação centralizada — muda em um lugar, vale em todos
- **Segurança:** Impossível passar CPF onde se espera e-mail (tipagem forte)
