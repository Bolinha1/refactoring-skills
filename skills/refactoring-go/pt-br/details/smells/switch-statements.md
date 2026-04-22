# SKILL: Detectando e Refatorando Switch Statements — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/switch-statements

---

## 1. O que é Switch Statements

Cadeias complexas de `switch` ou `if/else` que ramificam com base no tipo ou estado
de um valor. A mesma lógica de switch tende a ser duplicada por todo o código — quando
um novo caso é adicionado, cada switch deve ser encontrado e atualizado.

Em Go, isso também se manifesta como type switches (`switch v := x.(type)`) que
deveriam ser substituídos por dispatch de interface.

**Por que isso acontece:**
- Comportamento baseado em tipo foi implementado de forma procedural em vez de usar interfaces
- Uma string ou constante inteira foi usada para codificar um "tipo" em vez de uma interface Go
- Uma única struct cresceu para lidar com múltiplas variantes de um conceito

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] `switch` ou `if/else if` ramificando em um campo de tipo, string de status ou constante iota
- [ ] Type switch (`switch v := x.(type)`) com lógica de negócio em cada caso
- [ ] O mesmo switch aparece em mais de um lugar
- [ ] Adicionar um novo "tipo" exige buscar e atualizar todo switch no código
- [ ] Cada ramo faz algo muito diferente dos outros

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| Switch em tipo para decidir comportamento | Replace Conditional with Polymorphism (interface) |
| Switch em tipo para decidir qual struct criar | Função factory retornando interface |
| Poucos casos e adicionar novos é raro | Deixar — nem todo switch é um smell |
| Type switch em campo de tipo concreto | Substituir o campo por uma interface |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
type TipoFrete string

const (
	FreteStandard  TipoFrete = "standard"
	FreteExpresso  TipoFrete = "expresso"
	FreteUrgente   TipoFrete = "urgente"
)

func calcularFrete(pedido Pedido) float64 {
	switch pedido.TipoFrete {
	case FreteStandard:
		return pedido.Peso * 1.5
	case FreteExpresso:
		return pedido.Peso*3.0 + 5.0
	case FreteUrgente:
		return pedido.Peso*5.0 + 20.0
	default:
		panic("tipo de frete desconhecido")
	}
}
```

**DEPOIS — esperado:**
```go
type EstrategiaFrete interface {
	Calcular(peso float64) float64
}

type FreteStandard struct{}
func (f FreteStandard) Calcular(peso float64) float64 { return peso * 1.5 }

type FreteExpresso struct{}
func (f FreteExpresso) Calcular(peso float64) float64 { return peso*3.0 + 5.0 }

type FreteUrgente struct{}
func (f FreteUrgente) Calcular(peso float64) float64 { return peso*5.0 + 20.0 }

type Pedido struct {
	Peso   float64
	Frete  EstrategiaFrete
}

func (p *Pedido) CustoFrete() float64 {
	return p.Frete.Calcular(p.Peso)
}
```

**Por que esse padrão:**
- Adicionar um novo tipo de frete requer apenas uma nova struct implementando `EstrategiaFrete`
- Nenhum código existente precisa ser alterado
- Cada estratégia é testável de forma independente

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Substituir switch por cadeia if/else**
```go
// Não aceito — mesmo smell, sintaxe diferente
if pedido.TipoFrete == FreteStandard { ... } else if pedido.TipoFrete == FreteExpresso { ... }
```

**Erro 2: Centralizar em factory sem dispatch de interface**
```go
// Não aceito — o switch ainda existe, só foi movido
func novoFrete(tipo TipoFrete) EstrategiaFrete {
	switch tipo { ... } // mesmo problema
}
```

**Erro 3: Aplicar polimorfismo a flag booleana**
```go
// Não aceito — criar dois structs para true/false é over-engineering
switch pedido.Prioritario { ... }
```

---

## 6. Benefícios

- **Aberto/Fechado:** Novas variantes requerem nova struct, sem editar as existentes
- **Localidade:** Comportamento de cada variante está em um lugar
- **Testabilidade:** Cada implementação de interface pode ser testada isoladamente
