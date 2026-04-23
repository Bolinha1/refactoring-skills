# SKILL: Detectando e Refatorando Lazy Class — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/lazy-class

---

## 1. O que é Lazy Class

Uma struct, arquivo ou pacote que não faz o suficiente para justificar sua existência. Pode ser um wrapper trivial que apenas delega, uma struct criada para uma funcionalidade que nunca foi implementada, ou um pacote com um único arquivo de uma linha. Toda abstração tem um custo cognitivo — se ela não agrega valor, deve ser eliminada.

**Por que isso acontece:**
- Refatorações anteriores deixaram structs esvaziadas de responsabilidade
- Funcionalidade planejada nunca foi implementada
- Over-engineering antecipou uma complexidade que nunca se materializou

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Struct com apenas um ou dois métodos que simplesmente delegam para outra
- [ ] Pacote com um único arquivo que contém apenas uma função trivial
- [ ] Interface com apenas um implementador e sem perspectiva de um segundo
- [ ] Struct criada para herança (embedding) que nunca foi especializada
- [ ] Wrapper que não adiciona nenhuma lógica, transformação ou validação

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Struct que só delega para outra | Inline Class — mover o comportamento para o chamador |
| Subcomponente sem comportamento próprio | Collapse Hierarchy — fundir com o pai |
| Interface com único implementador desnecessário | Remover a interface, usar o tipo concreto diretamente |
| Pacote com uma única função trivial | Mover a função para o pacote chamador |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// LogadorPedido não agrega nenhuma lógica — só delega
type LogadorPedido struct {
    logger *log.Logger
}

func (l *LogadorPedido) LogarCriacao(pedidoID int) {
    l.logger.Printf("Pedido %d criado", pedidoID)
}

func (l *LogadorPedido) LogarCancelamento(pedidoID int) {
    l.logger.Printf("Pedido %d cancelado", pedidoID)
}

// Chamador cria LogadorPedido só para ter um log
type ServicoPedido struct {
    logador *LogadorPedido
}
```

**DEPOIS — esperado:**
```go
// Log feito diretamente onde faz sentido — sem intermediário vazio
type ServicoPedido struct {
    logger *log.Logger
}

func (s *ServicoPedido) Criar(pedido *Pedido) error {
    // ... lógica
    s.logger.Printf("Pedido %d criado", pedido.ID)
    return nil
}

func (s *ServicoPedido) Cancelar(pedido *Pedido) error {
    // ... lógica
    s.logger.Printf("Pedido %d cancelado", pedido.ID)
    return nil
}
```

**Por que este padrão:**
- `LogadorPedido` não justificava sua existência — apenas adicionava uma camada de indireção
- O logging direto é mais simples e igualmente fácil de testar

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter a struct lazy "por conveniência futura"**
```go
// Não aceito — YAGNI (You Aren't Gonna Need It); remover agora, adicionar quando necessário
type GerenciadorFuturo struct {
    servico *Servico
}
func (g *GerenciadorFuturo) Executar() { g.servico.Executar() }
```

**Erro 2: Criar uma interface para uma struct lazy**
```go
// Não aceito — interface sobre uma struct vazia multiplica a indireção sem valor
type ILogadorPedido interface {
    LogarCriacao(id int)
}
```

---

## 6. Benefícios

- **Simplicidade:** Menos arquivos e structs para navegar e entender
- **Navegação:** Chamadores chegam diretamente à lógica real sem atravessar wrappers vazios
- **Manutenção:** Menos código para manter, testar e documentar
