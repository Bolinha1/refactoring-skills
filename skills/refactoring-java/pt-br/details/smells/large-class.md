# SKILL: Detecção e Refatoração de Large Class — Java

## Fonte
Baseado em: https://refactoring.guru/smells/large-class

---

## 1. O que é Large Class

Uma classe que contém muitos campos, métodos ou linhas de código.
Quando uma classe tenta fazer coisas demais, ela acumula responsabilidades
que deveriam estar distribuídas.

**Por que isso acontece:**
- Classes começam pequenas e crescem conforme o programa evolui
- É mentalmente mais fácil adicionar funcionalidade a uma classe existente do que criar uma nova
- O acúmulo é gradual — ninguém percebe até que a classe já virou um monolito

---

## 2. Sinais de alerta (gatilhos para acionar este SKILL)

- [ ] Classe com mais de 200 linhas
- [ ] Classe com mais de 10 métodos públicos
- [ ] Classe com mais de 5 campos/atributos
- [ ] Classe com responsabilidades claramente distintas (violação de SRP)
- [ ] Classe que é modificada por razões diferentes
- [ ] Muitos imports que não se relacionam entre si
- [ ] Nome da classe é genérico demais (Manager, Processor, Handler, Utils)
- [ ] Campos que são usados apenas por um subconjunto dos métodos

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                               | Técnica indicada        |
|----------------------------------------------------|-------------------------|
| Comportamentos agrupáveis em unidade autônoma      | Extract Class           |
| Funcionalidade especializada ou raramente usada    | Extract Subclass        |
| Clientes usam apenas parte da interface            | Extract Interface       |
| Classe GUI com dados e lógica misturados           | Duplicate Observed Data |

**Regra de ouro:** Se você consegue descrever a classe com a conjunção "e"
(ex: "essa classe valida pedidos **e** calcula frete **e** envia email"),
ela tem responsabilidades demais.

---

## 4. Exemplo

**ANTES — não aceito:**
```java
public class PedidoService {
    private PedidoRepository repository;
    private EmailService emailService;
    private EstoqueService estoqueService;
    private NotaFiscalService nfService;

    public void criar(Pedido pedido) {
        if (pedido.getItens().isEmpty()) throw new RuntimeException("Sem itens");
        if (pedido.getCliente() == null) throw new RuntimeException("Sem cliente");

        double total = 0;
        for (Item item : pedido.getItens()) {
            total += item.getPreco() * item.getQuantidade();
        }
        pedido.setTotal(total);

        for (Item item : pedido.getItens()) {
            estoqueService.reservar(item.getProdutoId(), item.getQuantidade());
        }

        repository.save(pedido);

        NotaFiscal nf = nfService.gerar(pedido);
        pedido.setNotaFiscal(nf);

        emailService.enviarConfirmacao(pedido.getCliente().getEmail(), pedido);
    }

    public void cancelar(Pedido pedido) { /* ... lógica extensa ... */ }
    public void recalcularFrete(Pedido pedido) { /* ... lógica extensa ... */ }
    public void aplicarCupom(Pedido pedido, String cupom) { /* ... lógica extensa ... */ }
    public List<Pedido> buscarPorCliente(Long clienteId) { /* ... */ }
    public List<Pedido> buscarPorPeriodo(Date inicio, Date fim) { /* ... */ }
    public byte[] gerarRelatorio(List<Pedido> pedidos) { /* ... */ }
}
```

**DEPOIS — esperado (Extract Class):**
```java
public class PedidoService {
    private PedidoRepository repository;
    private PedidoValidator validator;
    private CalculadoraDeTotal calculadora;
    private ReservaDeEstoque reservaEstoque;
    private NotificacaoDePedido notificacao;

    public void criar(Pedido pedido) {
        validator.validar(pedido);
        double total = calculadora.calcular(pedido);
        pedido.setTotal(total);
        reservaEstoque.reservar(pedido);
        repository.save(pedido);
        notificacao.enviarConfirmacao(pedido);
    }
}

public class PedidoValidator {
    public void validar(Pedido pedido) {
        if (pedido.getItens().isEmpty()) throw new RuntimeException("Sem itens");
        if (pedido.getCliente() == null) throw new RuntimeException("Sem cliente");
    }
}

public class CalculadoraDeTotal {
    public double calcular(Pedido pedido) {
        return pedido.getItens().stream()
            .mapToDouble(i -> i.getPreco() * i.getQuantidade())
            .sum();
    }
}

public class ReservaDeEstoque {
    private EstoqueService estoqueService;

    public void reservar(Pedido pedido) {
        for (Item item : pedido.getItens()) {
            estoqueService.reservar(item.getProdutoId(), item.getQuantidade());
        }
    }
}

public class NotificacaoDePedido {
    private EmailService emailService;
    private NotaFiscalService nfService;

    public void enviarConfirmacao(Pedido pedido) {
        NotaFiscal nf = nfService.gerar(pedido);
        pedido.setNotaFiscal(nf);
        emailService.enviarConfirmacao(pedido.getCliente().getEmail(), pedido);
    }
}
```

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair classe sem coesão**
```java
// Não aceito — a classe extraída ainda mistura responsabilidades
public class PedidoHelper {
    public void validar(Pedido pedido) { ... }
    public void calcularFrete(Pedido pedido) { ... }
    public byte[] gerarRelatorio(List<Pedido> pedidos) { ... }
}
```

**Erro 2: Criar classes anêmicas**
```java
// Não aceito — apenas delega sem comportamento real
public class PedidoValidator {
    public boolean validate(Pedido pedido) {
        return pedido.isValid();
    }
}
```

**Erro 3: Nomes genéricos nas classes extraídas**
```java
// Não aceito
public class PedidoUtils { ... }
public class PedidoManager { ... }
public class PedidoHelper2 { ... }
```

---

## 6. Benefícios

- **Carga cognitiva:** Desenvolvedores não precisam memorizar atributos e métodos excessivos
- **Eliminação de duplicação:** Dividir classes grandes frequentemente elimina código redundante
- **Manutenção:** Cada classe com responsabilidade única é mais fácil de testar e modificar
- **Reutilização:** Classes menores e focadas são mais fáceis de reutilizar em outros contextos
