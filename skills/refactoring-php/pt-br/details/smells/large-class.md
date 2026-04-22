# SKILL: Detecção e Refatoração de Large Class — PHP

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
- [ ] Classe com mais de 5 propriedades injetadas via construtor
- [ ] Classe com responsabilidades claramente distintas (violação de SRP)
- [ ] Classe que é modificada por razões diferentes
- [ ] Muitos `use` no topo que não se relacionam entre si
- [ ] Nome da classe é genérico demais (Manager, Processor, Handler, Utils)
- [ ] Propriedades usadas apenas por um subconjunto dos métodos

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
```php
class PedidoService
{
    public function __construct(
        private PedidoRepository $repository,
        private EmailService $emailService,
        private EstoqueService $estoqueService,
        private NotaFiscalService $nfService,
    ) {}

    public function criar(Pedido $pedido): void
    {
        if (empty($pedido->getItens())) {
            throw new \RuntimeException("Sem itens");
        }
        if ($pedido->getCliente() === null) {
            throw new \RuntimeException("Sem cliente");
        }

        $total = 0;
        foreach ($pedido->getItens() as $item) {
            $total += $item->getPreco() * $item->getQuantidade();
        }
        $pedido->setTotal($total);

        foreach ($pedido->getItens() as $item) {
            $this->estoqueService->reservar($item->getProdutoId(), $item->getQuantidade());
        }

        $this->repository->save($pedido);

        $nf = $this->nfService->gerar($pedido);
        $pedido->setNotaFiscal($nf);

        $this->emailService->enviarConfirmacao($pedido->getCliente()->getEmail(), $pedido);
    }

    public function cancelar(Pedido $pedido): void { /* ... */ }
    public function recalcularFrete(Pedido $pedido): void { /* ... */ }
    public function aplicarCupom(Pedido $pedido, string $cupom): void { /* ... */ }
    public function buscarPorCliente(int $clienteId): array { /* ... */ }
    public function buscarPorPeriodo(\DateTime $inicio, \DateTime $fim): array { /* ... */ }
    public function gerarRelatorio(array $pedidos): string { /* ... */ }
}
```

**DEPOIS — esperado (Extract Class):**
```php
class PedidoService
{
    public function __construct(
        private PedidoRepository $repository,
        private PedidoValidator $validator,
        private CalculadoraDeTotal $calculadora,
        private ReservaDeEstoque $reservaEstoque,
        private NotificacaoDePedido $notificacao,
    ) {}

    public function criar(Pedido $pedido): void
    {
        $this->validator->validar($pedido);
        $pedido->setTotal($this->calculadora->calcular($pedido));
        $this->reservaEstoque->reservar($pedido);
        $this->repository->save($pedido);
        $this->notificacao->enviarConfirmacao($pedido);
    }
}

class PedidoValidator
{
    public function validar(Pedido $pedido): void
    {
        if (empty($pedido->getItens())) {
            throw new \RuntimeException("Sem itens");
        }
        if ($pedido->getCliente() === null) {
            throw new \RuntimeException("Sem cliente");
        }
    }
}

class CalculadoraDeTotal
{
    public function calcular(Pedido $pedido): float
    {
        return array_reduce($pedido->getItens(), function (float $carry, Item $item) {
            return $carry + $item->getPreco() * $item->getQuantidade();
        }, 0.0);
    }
}

class ReservaDeEstoque
{
    public function __construct(private EstoqueService $estoqueService) {}

    public function reservar(Pedido $pedido): void
    {
        foreach ($pedido->getItens() as $item) {
            $this->estoqueService->reservar($item->getProdutoId(), $item->getQuantidade());
        }
    }
}

class NotificacaoDePedido
{
    public function __construct(
        private EmailService $emailService,
        private NotaFiscalService $nfService,
    ) {}

    public function enviarConfirmacao(Pedido $pedido): void
    {
        $nf = $this->nfService->gerar($pedido);
        $pedido->setNotaFiscal($nf);
        $this->emailService->enviarConfirmacao($pedido->getCliente()->getEmail(), $pedido);
    }
}
```

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair classe sem coesão**
```php
// Não aceito — a classe extraída ainda mistura responsabilidades
class PedidoHelper
{
    public function validar(Pedido $pedido): void { ... }
    public function calcularFrete(Pedido $pedido): float { ... }
    public function gerarRelatorio(array $pedidos): string { ... }
}
```

**Erro 2: Criar classes anêmicas**
```php
// Não aceito — apenas delega sem comportamento real
class PedidoValidator
{
    public function validate(Pedido $pedido): bool
    {
        return $pedido->isValid();
    }
}
```

**Erro 3: Nomes genéricos nas classes extraídas**
```php
// Não aceito
class PedidoUtils { ... }
class PedidoManager { ... }
class PedidoHelper2 { ... }
```

---

## 6. Benefícios

- **Carga cognitiva:** Desenvolvedores não precisam memorizar atributos e métodos excessivos
- **Eliminação de duplicação:** Dividir classes grandes frequentemente elimina código redundante
- **Manutenção:** Cada classe com responsabilidade única é mais fácil de testar e modificar
- **Reutilização:** Classes menores e focadas são mais fáceis de reutilizar em outros contextos
