# SKILL: Detectando e Refatorando Parallel Inheritance Hierarchies — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/parallel-inheritance-hierarchies

---

## 1. O que é?

Toda vez que você cria um novo tipo em um conjunto de structs/interfaces, você precisa
criar um tipo correspondente em outro conjunto. As duas hierarquias crescem em paralelo
e estão acopladas: uma não pode evoluir sem a outra.

Em Go, isso se manifesta como dois conjuntos paralelos de tipos (ou implementações de
interface) que sempre mudam juntos.

**Por que isso acontece:**
- Uma responsabilidade foi dividida entre duas famílias de tipos em vez de centralizada
- Tentativa de separar "dados" de "comportamento" resultou em espelhos acoplados
- Crescimento gradual sem perceber o padrão emergente

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Criar um novo tipo em um pacote exige criar um tipo correspondente em outro pacote
- [ ] Dois conjuntos de tipos têm nomes com o mesmo prefixo ou sufixo
- [ ] Alterar um tipo obriga a alterar seu "par" no outro conjunto
- [ ] Os dois conjuntos sempre crescem juntos e nunca de forma independente

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| Uma hierarquia pode ser absorvida pela outra | Move Method + Move Field |
| As hierarquias têm responsabilidades distintas reais | Manter — pode ser design intencional |
| Uma hierarquia é puro comportamento da outra | Unificar em uma única família de interfaces |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// Hierarquia 1: tipos de funcionário
type Funcionario interface{ Nome() string }
type Engenheiro struct{ nome string }
type Gerente struct{ nome string }

// Hierarquia 2: paralela — calculadores de bônus (um para cada funcionário)
type CalculadorBonus interface{ Calcular(salario float64) float64 }
type CalculadorBonusEngenheiro struct{}
type CalculadorBonusGerente struct{}

// Toda vez que um novo tipo de funcionário é criado,
// um novo calculador também precisa ser criado
```

**DEPOIS — esperado:**
```go
// Unificar: o comportamento de bônus pertence ao próprio funcionário
type Funcionario interface {
	Nome() string
	CalcularBonus(salario float64) float64
}

type Engenheiro struct{ nome string }
func (e *Engenheiro) Nome() string { return e.nome }
func (e *Engenheiro) CalcularBonus(salario float64) float64 { return salario * 0.10 }

type Gerente struct{ nome string }
func (g *Gerente) Nome() string { return g.nome }
func (g *Gerente) CalcularBonus(salario float64) float64 { return salario*0.20 + 1000 }
```

**Por que esse padrão:**
- Adicionar um novo tipo de funcionário requer apenas uma nova struct — não duas
- A interface `Funcionario` é o único contrato a manter

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter as duas hierarquias por "separação de responsabilidades"**
```go
// Cuidado — se as duas hierarquias sempre evoluem juntas, a separação
// é artificial e só aumenta o acoplamento
```

**Erro 2: Criar uma terceira hierarquia para "resolver" o problema**
```go
// Não aceito — adicionar mais camadas sem remover as existentes piora o problema
type AdaptadorFuncionario interface { ... }
```

---

## 6. Benefícios

- **Coesão:** Comportamento relacionado vive junto no mesmo tipo
- **Manutenção:** Adicionar um novo tipo é uma única operação, não duas
- **Legibilidade:** O código cliente interage com uma única interface
