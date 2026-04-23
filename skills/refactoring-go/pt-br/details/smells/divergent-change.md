# SKILL: Detectando e Refatorando Divergent Change — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/divergent-change

---

## 1. O que é Divergent Change

Uma única struct ou arquivo precisa ser modificada por razões diferentes e não relacionadas. Você abre o mesmo arquivo para corrigir um bug de banco de dados, para ajustar uma regra de negócio e para mudar o formato de resposta da API — tudo no mesmo lugar. Isso viola o Princípio da Responsabilidade Única (SRP).

**Por que isso acontece:**
- A struct acumulou responsabilidades ao longo do tempo sem refatoração
- Desenvolvedores adicionaram funcionalidades ao arquivo mais óbvio em vez de criar uma abstração nova
- Ausência de separação clara entre camadas (persistência, domínio, apresentação)

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] A mesma struct é modificada por mudanças em banco de dados, regras de negócio e formato de API
- [ ] Um único arquivo tem muitos commits com mensagens completamente diferentes
- [ ] A struct tem métodos que pertencem claramente a grupos de responsabilidade distintos
- [ ] Adicionar um novo banco de dados ou uma nova regra força mudanças no mesmo lugar

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Responsabilidades distintas em uma struct | Extract Class para cada responsabilidade |
| Lógica de persistência misturada com domínio | Separar em repositório e serviço de domínio |
| Lógica de apresentação misturada com negócio | Move Method para struct de resposta/DTO |
| Comportamentos variantes independentemente | Extract Interface para cada eixo de variação |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// ServicoPedido muda por 3 razões distintas: BD, regra de negócio e formato de resposta
type ServicoPedido struct {
    db *sql.DB
}

func (s *ServicoPedido) BuscarPedido(id int) (*Pedido, error) {
    // lógica de banco de dados
    row := s.db.QueryRow("SELECT * FROM pedidos WHERE id = ?", id)
    // ...
    return pedido, nil
}

func (s *ServicoPedido) AplicarDesconto(pedido *Pedido, cupom string) {
    // regra de negócio
    if cupom == "PROMO10" {
        pedido.Total *= 0.90
    }
}

func (s *ServicoPedido) FormatarResposta(pedido *Pedido) map[string]interface{} {
    // lógica de apresentação
    return map[string]interface{}{
        "id":    pedido.ID,
        "total": fmt.Sprintf("R$ %.2f", pedido.Total),
    }
}
```

**DEPOIS — esperado:**
```go
// Cada struct muda por apenas uma razão

type RepositorioPedido struct {
    db *sql.DB
}

func (r *RepositorioPedido) BuscarPorID(id int) (*Pedido, error) {
    // apenas persistência
    return nil, nil
}

type ServicoPedido struct {
    repo *RepositorioPedido
}

func (s *ServicoPedido) AplicarDesconto(pedido *Pedido, cupom string) {
    // apenas regra de negócio
    if cupom == "PROMO10" {
        pedido.Total *= 0.90
    }
}

type PedidoResposta struct {
    ID    int    `json:"id"`
    Total string `json:"total"`
}

func NovaPedidoResposta(pedido *Pedido) PedidoResposta {
    // apenas apresentação
    return PedidoResposta{
        ID:    pedido.ID,
        Total: fmt.Sprintf("R$ %.2f", pedido.Total),
    }
}
```

**Por que este padrão:**
- Cada struct tem um único eixo de mudança — modificar o banco de dados não toca a regra de negócio
- Estrutura de pacotes reflete as camadas: `repositorio`, `servico`, `handler`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Separar em arquivos mas manter na mesma struct**
```go
// Não aceito — dividir em arquivos não resolve se a struct ainda acumula responsabilidades
// pedido_bd.go, pedido_negocio.go, pedido_api.go — todos estendendo ServicoPedido
```

**Erro 2: Criar uma struct "Util" ou "Helper" que acumula os métodos removidos**
```go
// Não aceito — PedidoHelper ainda agrupa coisas não relacionadas
type PedidoHelper struct{}
```

---

## 6. Benefícios

- **Estabilidade:** Uma mudança em um eixo não arrisca quebrar código de outro eixo
- **Rastreabilidade:** O histórico de git de cada arquivo reflete um único tipo de mudança
- **Testabilidade:** Cada camada é testável de forma isolada com mocks simples
