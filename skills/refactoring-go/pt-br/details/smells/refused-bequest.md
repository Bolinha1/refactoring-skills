# SKILL: Detectando e Refatorando Refused Bequest — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/refused-bequest

---

## 1. O que é?

Uma struct embute outra (embedding) ou implementa uma interface, mas não usa a maioria
dos métodos promovidos — ou os sobrescreve com `panic`. Em Go, herança é substituída
por embedding e interfaces, portanto esse smell se manifesta como abuso de embedding:
um tipo "herdando" comportamento que na realidade não suporta.

---

## 2. Sinais de alerta

- [ ] Métodos promovidos pelo embedding são sobrescritos com `panic` ou retorno vazio
- [ ] Apenas uma pequena fração da API do tipo embutido é usada
- [ ] Um tipo implementa uma interface mas vários métodos retornam stubs ou erros
- [ ] O relacionamento "é-um" é verdadeiro apenas conceitualmente, não comportamentalmente
- [ ] Testes que exercitam o comportamento embutido falham para o tipo que o embute

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| O tipo quer apenas parte do comportamento | Replace Inheritance with Delegation (campo em vez de embedding) |
| Comportamento compartilhado deve ser extraído | Extract Interface mínima |
| Métodos indesejados vieram do tipo embutido | Push Down Method — mover para quem realmente usa |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
type Ave struct {
	nome string
}

func (a *Ave) Nome() string  { return a.nome }
func (a *Ave) Voar()         { fmt.Println("batendo asas") }
func (a *Ave) Pousar()       { fmt.Println("aterrissando") }

// Pinguim embute Ave mas não consegue voar
type Pinguim struct {
	Ave
}

func (p *Pinguim) Voar() {
	panic("pinguins não podem voar") // legado recusado
}
```

**DEPOIS — esperado:**
```go
type Ave interface {
	Nome() string
}

type Voador interface {
	Ave
	Voar()
	Pousar()
}

type Pardal struct{ nome string }

func (p *Pardal) Nome() string { return p.nome }
func (p *Pardal) Voar()        { fmt.Println("batendo asas") }
func (p *Pardal) Pousar()      { fmt.Println("aterrissando") }

type Pinguim struct{ nome string }

func (p *Pinguim) Nome() string { return p.nome }
func (p *Pinguim) Nadar()       { fmt.Println("nadando") }
```

**Por que esse padrão:**
- `Ave` contém apenas comportamento compartilhado por todas as aves
- `Voador` é uma capacidade opcional — pinguins simplesmente não a implementam
- Código que precisa de voo aceita `Voador`, não qualquer `Ave`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter o panic e documentar**
```go
// Não aceito — o contrato ainda é violado; chamadores não podem confiar nele
func (p *Pinguim) Voar() {
	// pinguins não voam — documentado mas ainda quebra chamadores
	panic("pinguins não podem voar")
}
```

**Erro 2: Retornar no-op em vez de reestruturar**
```go
// Não aceito — falhas silenciosas escondem bugs
func (p *Pinguim) Voar() {} // não faz nada, silenciosamente
```

---

## 6. Benefícios

- **Segurança de interface:** Cada tipo honra o contrato completo que anuncia
- **Embedding correto:** Embedding é usado para reuso genuíno, não herança falsa
- **Conformidade com LSP:** Código que itera `[]Ave` não terá panic inesperado
