# SKILL: Detectando e Refatorando Large Class — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/large-class

---

## 1. O que é Large Class

Uma struct que acumula campos, métodos e responsabilidades demais. Em Go isso se manifesta como um arquivo com uma única struct enorme ou um conjunto de funções com o mesmo receiver que cobre domínios completamente diferentes.

**Por que isso acontece:**
- Structs começam pequenas e crescem conforme o programa evolui
- É mentalmente mais fácil adicionar funcionalidade a uma struct existente do que criar uma nova
- A acumulação é gradual — ninguém percebe até a struct virar um monolito

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Arquivo com mais de 200 linhas dedicado a uma única struct
- [ ] Struct com mais de 10 métodos públicos
- [ ] Struct com mais de 5 campos
- [ ] Struct com responsabilidades claramente distintas (violação do SRP)
- [ ] Campos que só são usados por um subconjunto dos métodos
- [ ] Nome genérico demais: `Manager`, `Processor`, `Handler`, `Service`

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Comportamentos agrupáveis em unidade autônoma | Extract Class (nova struct) |
| Funcionalidade especializada raramente usada | Extract Class separada |
| Clientes usam apenas parte da interface | Extract Interface |
| Campos usados apenas por subconjunto de métodos | Extract Class com campos relacionados |

**Regra de ouro:** Se você pode descrever a struct usando a conjunção "e" (ex: "esta struct valida pedidos **e** calcula frete **e** envia email"), ela tem responsabilidades demais.

---

## 4. Exemplo

**ANTES — não aceito:**
```go
type ServicoPedido struct {
    db           *sql.DB
    emailCliente string
    estoque      *ServicoEstoque
    notificacao  *ServicoNotificacao
}

func (s *ServicoPedido) Criar(pedido *Pedido) error {
    if len(pedido.Itens) == 0 {
        return fmt.Errorf("pedido sem itens")
    }
    if pedido.Cliente == nil {
        return fmt.Errorf("cliente não informado")
    }
    total := 0.0
    for _, item := range pedido.Itens {
        total += item.Preco * float64(item.Quantidade)
    }
    pedido.Total = total
    for _, item := range pedido.Itens {
        s.estoque.Reservar(item.ProdutoID, item.Quantidade)
    }
    // salvar no banco, gerar nota, enviar email...
    return nil
}

func (s *ServicoPedido) Cancelar(pedido *Pedido) error        { /* lógica extensa */ return nil }
func (s *ServicoPedido) RecalcularFrete(pedido *Pedido) error { /* lógica extensa */ return nil }
func (s *ServicoPedido) AplicarCupom(pedido *Pedido, cupom string) error { return nil }
func (s *ServicoPedido) BuscarPorCliente(clienteID int) ([]*Pedido, error) { return nil, nil }
func (s *ServicoPedido) GerarRelatorio(pedidos []*Pedido) ([]byte, error)  { return nil, nil }
```

**DEPOIS — esperado:**
```go
type ValidadorPedido struct{}

func (v *ValidadorPedido) Validar(pedido *Pedido) error {
    if len(pedido.Itens) == 0 {
        return fmt.Errorf("pedido sem itens")
    }
    if pedido.Cliente == nil {
        return fmt.Errorf("cliente não informado")
    }
    return nil
}

type CalculadorTotal struct{}

func (c *CalculadorTotal) Calcular(itens []ItemPedido) float64 {
    total := 0.0
    for _, item := range itens {
        total += item.Preco * float64(item.Quantidade)
    }
    return total
}

type ReservadorEstoque struct {
    estoque *ServicoEstoque
}

func (r *ReservadorEstoque) Reservar(itens []ItemPedido) error {
    for _, item := range itens {
        if err := r.estoque.Reservar(item.ProdutoID, item.Quantidade); err != nil {
            return err
        }
    }
    return nil
}

type ServicoPedido struct {
    validador   *ValidadorPedido
    calculador  *CalculadorTotal
    reservador  *ReservadorEstoque
    repositorio *RepositorioPedido
}

func (s *ServicoPedido) Criar(pedido *Pedido) error {
    if err := s.validador.Validar(pedido); err != nil {
        return err
    }
    pedido.Total = s.calculador.Calcular(pedido.Itens)
    if err := s.reservador.Reservar(pedido.Itens); err != nil {
        return err
    }
    return s.repositorio.Salvar(pedido)
}
```

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair uma struct sem coesão**
```go
// Não aceito — PedidoHelper ainda mistura responsabilidades
type PedidoHelper struct{}
func (h *PedidoHelper) Validar(p *Pedido) error      { ... }
func (h *PedidoHelper) CalcularFrete(p *Pedido) error { ... }
func (h *PedidoHelper) GerarRelatorio() []byte        { ... }
```

**Erro 2: Criar structs anêmicas sem comportamento real**
```go
// Não aceito — apenas delega sem agregar valor
type ValidadorPedido struct{}
func (v *ValidadorPedido) Validar(p *Pedido) bool { return p.Valido() }
```

---

## 6. Benefícios

- **Carga cognitiva:** Desenvolvedores não precisam memorizar dezenas de métodos e campos
- **Eliminação de duplicatas:** Dividir structs grandes frequentemente expõe código redundante
- **Manutenibilidade:** Cada struct com responsabilidade única é mais fácil de testar e modificar
- **Reutilização:** Structs menores e focadas são reutilizáveis em outros contextos
