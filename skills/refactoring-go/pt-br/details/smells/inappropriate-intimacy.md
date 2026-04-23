# SKILL: Detectando e Refatorando Intimidade Inapropriada — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/inappropriate-intimacy

---

## 1. O que é Intimidade Inapropriada

Duas structs ou pacotes conhecem detalhes internos um do outro em excesso — acessam campos não exportados via reflexão, dependem de comportamento interno indocumentado ou criam dependências circulares. Esse acoplamento bidirecional torna qualquer mudança em um perigosa para o outro.

**Por que isso acontece:**
- Funcionalidades que deveriam ser separadas foram desenvolvidas de forma entrelaçada
- Campos ou funções foram deixados exportados por conveniência sem considerar o encapsulamento
- Dependências circulares entre pacotes surgiram gradualmente sem revisão de arquitetura

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] O pacote A importa o pacote B e o pacote B importa o pacote A (dependência circular)
- [ ] Uma struct acessa campos de outra struct que deveriam ser privados
- [ ] Uma struct tem métodos que parecem pertencer à outra struct
- [ ] Testes de uma struct precisam conhecer detalhes de implementação de outra
- [ ] Mudanças em uma struct frequentemente forçam mudanças em outra

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Método de A que usa mais dados de B | Move Method para B |
| A e B compartilham comportamento comum | Extract Class para um terceiro C |
| Dependência circular entre pacotes | Introduzir interface para inverter a dependência |
| Campo exportado acessado diretamente | Encapsulate Field — tornar privado, expor via método |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// pacote pedido
type Pedido struct {
    Cliente  *usuario.Usuario
    Itens    []Item
    desconto float64 // campo interno
}

// pacote usuario — acessa internos de Pedido
type Usuario struct {
    Nome string
}

func (u *Usuario) CalcularHistoricoGastos(pedidos []*pedido.Pedido) float64 {
    total := 0.0
    for _, p := range pedidos {
        // acessa detalhes internos de Pedido
        for _, item := range p.Itens {
            total += item.Preco
        }
    }
    return total
}
```

**DEPOIS — esperado:**
```go
// pacote pedido — Pedido expõe o que é necessário via interface
type Pedido struct {
    cliente  clienteInfo
    itens    []Item
    desconto float64
}

func (p *Pedido) Total() float64 {
    total := 0.0
    for _, item := range p.itens {
        total += item.Preco
    }
    return total * (1 - p.desconto)
}

// Interface que rompe a dependência circular
type PedidoResumo interface {
    Total() float64
}

// pacote usuario — depende da interface, não da struct concreta
type Usuario struct {
    Nome string
}

func (u *Usuario) CalcularHistoricoGastos(pedidos []PedidoResumo) float64 {
    total := 0.0
    for _, p := range pedidos {
        total += p.Total()
    }
    return total
}
```

**Por que este padrão:**
- `Usuario` depende de uma interface mínima, não dos internos de `Pedido`
- A dependência circular é eliminada — cada pacote pode evoluir de forma independente

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Usar reflexão para acessar campos não exportados**
```go
// Não aceito — viola o encapsulamento de forma ainda mais grave
val := reflect.ValueOf(pedido).Elem().FieldByName("desconto")
```

**Erro 2: Tornar tudo exportado para facilitar o acesso**
```go
// Não aceito — remove toda barreira de encapsulamento
type Pedido struct {
    Desconto float64 // exportado só para que Usuario possa acessar
}
```

---

## 6. Benefícios

- **Independência:** Cada struct e pacote pode ser modificado sem propagar mudanças para o outro
- **Testabilidade:** Dependências via interface são facilmente substituídas por mocks
- **Arquitetura:** Ausência de dependências circulares mantém o grafo de importações limpo e compilável
