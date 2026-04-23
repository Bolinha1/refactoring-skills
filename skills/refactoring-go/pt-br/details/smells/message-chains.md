# SKILL: Detectando e Refatorando Message Chains — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/message-chains

---

## 1. O que é Message Chains

Uma cadeia de chamadas encadeadas onde o código precisa navegar por vários objetos
para obter o que precisa: `a.GetB().GetC().GetD()`. Cada chamada cria dependência do
código chamador em toda a estrutura interna da cadeia.

**Por que isso acontece:**
- O código acessa dados diretamente em vez de pedir ao objeto mais próximo
- A Lei de Demeter é ignorada: "fale apenas com seus amigos imediatos"
- Falta de métodos de conveniência nos objetos intermediários

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Cadeia de chamadas com mais de dois pontos: `a.B().C().D()`
- [ ] Código que navega pela estrutura interna de objetos para obter um valor
- [ ] A cadeia precisa ser duplicada em vários lugares
- [ ] Uma mudança na estrutura interna quebra vários pontos no código chamador
- [ ] Variável temporária criada só para guardar resultado intermediário da cadeia

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| Cadeia usada em vários lugares | Hide Delegate — adicionar método de conveniência |
| Cadeia usada em apenas um lugar | Extract Method — encapsular a navegação |
| Objeto intermediário pode ser eliminado | Remove Middle Man |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// O chamador precisa conhecer a estrutura interna de Pedido
cidade := pedido.GetCliente().GetEndereco().GetCidade()
cep := pedido.GetCliente().GetEndereco().GetCep()
```

**DEPOIS — esperado:**
```go
// Adicionar método de conveniência em Pedido
func (p *Pedido) CidadeEntrega() string {
	return p.cliente.endereco.cidade
}

func (p *Pedido) CepEntrega() string {
	return p.cliente.endereco.cep
}

// Chamador usa apenas o vizinho imediato
cidade := pedido.CidadeEntrega()
cep := pedido.CepEntrega()
```

**Por que esse padrão:**
- O chamador não precisa conhecer a estrutura interna de `Pedido`
- Se `Endereco` mudar de lugar, apenas `Pedido` precisa ser atualizado
- O nome do método expressa intenção de negócio

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Variável temporária apenas para esconder a cadeia**
```go
// Não aceito — a dependência estrutural ainda existe
endereco := pedido.GetCliente().GetEndereco()
cidade := endereco.GetCidade()
```

**Erro 2: Adicionar métodos de conveniência para toda e qualquer cadeia**
```go
// Não aceito — não crie métodos de conveniência para acesso raramente usado;
// avalie se o acesso é frequente o suficiente para justificar o novo método
```

---

## 6. Benefícios

- **Encapsulamento:** O chamador depende apenas do objeto com que interage diretamente
- **Manutenibilidade:** Mudanças na estrutura interna ficam contidas no struct proprietário
- **Legibilidade:** Métodos com nome de negócio são mais expressivos que cadeias de accessors
