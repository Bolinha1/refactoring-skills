# SKILL: Detectando e Refatorando Primitive Obsession — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/primitive-obsession

---

## 1. O que é Primitive Obsession

Uso excessivo de tipos primitivos (`string`, `int`, `float64`, `bool`) para representar
conceitos de domínio que mereceriam tipos próprios. Em Go, isso também inclui usar
`string` como tipo enumerado sem restrição de valores válidos.

**Por que isso acontece:**
- Criar um novo tipo parece exagero para algo "simples"
- Tipos primitivos são fáceis e rápidos de usar no início
- O código cresce e o primitivo vira um campo mágico difícil de rastrear

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] `string` representando CPF, telefone, CEP, e-mail, código de produto
- [ ] `string` ou `int` simulando um tipo enumerado sem iota ou tipo definido
- [ ] Múltiplos parâmetros primitivos que sempre aparecem juntos (ex: `float64, string` para dinheiro)
- [ ] Validação do mesmo primitivo repetida em vários lugares
- [ ] `map[string]interface{}` usado como estrutura de dados com chaves mágicas

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| Primitivo com regras de validação próprias | Replace Data Value with Object (tipo Go definido) |
| Múltiplos primitivos que andam juntos | Introduce Parameter Object (struct) |
| `string`/`int` simulando enumeração | Usar `type Status string` com constantes ou `iota` |
| `map[string]interface{}` como estrutura | Replace Array with Object (struct tipada) |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
type Cliente struct {
	Nome     string
	CPF      string  // sem validação centralizada
	Telefone string
	Email    string
}

func criarPedido(cpfCliente string, valor float64, moeda string, tipoPagamento int) {
	// tipoPagamento: 1 = cartão, 2 = boleto, 3 = pix — constante mágica
	if tipoPagamento == 1 { ... }
	// validação de CPF duplicada em vários pontos
	if len(cpfCliente) != 11 { ... }
}
```

**DEPOIS — esperado:**
```go
// Tipo com validação centralizada
type CPF string

func NovoCPF(valor string) (CPF, error) {
	if len(valor) != 11 {
		return "", fmt.Errorf("CPF inválido: %s", valor)
	}
	return CPF(valor), nil
}

// Struct para dados que andam juntos
type Dinheiro struct {
	Valor  float64
	Moeda  string
}

// Tipo enumerado com iota
type TipoPagamento int

const (
	Cartao TipoPagamento = iota + 1
	Boleto
	Pix
)

type Cliente struct {
	Nome     string
	CPF      CPF
	Email    string
}

func criarPedido(cpf CPF, valor Dinheiro, tipo TipoPagamento) {
	if tipo == Cartao { ... }
}
```

**Por que esse padrão:**
- A validação do CPF está centralizada em `NovoCPF` — não se repete
- `TipoPagamento` com `iota` é autoexplicativo, sem inteiros mágicos
- `Dinheiro` agrupa valor e moeda — evita que andem separados

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Usar string para tudo**
```go
// Não aceito
func processar(cpf string, status string, tipoPagamento string) { ... }
```

**Erro 2: Constantes inteiras no lugar de tipo enumerado**
```go
// Não aceito
const PagamentoCartao = 1
const PagamentoBoleto = 2
if tipo == PagamentoCartao { ... }
```

**Erro 3: map com chaves mágicas no lugar de struct**
```go
// Não aceito
endereco := map[string]string{
	"rua": "Av. Paulista",
	"numero": "1000",
	"cidade": "São Paulo",
}
```

---

## 6. Benefícios

- **Segurança de tipos:** Impossível passar CPF onde se espera e-mail
- **Validação centralizada:** Muda em um lugar, vale em todos
- **Legibilidade:** Parâmetros e campos expressam conceitos de domínio
- **Manutenção:** Regras de negócio ficam encapsuladas no tipo correto
