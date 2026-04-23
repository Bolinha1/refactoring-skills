# SKILL: Detectando e Refatorando Temporary Field — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/temporary-field

---

## 1. O que é?

Uma struct contém um campo que só é definido e usado durante uma chamada de método
ou algoritmo. No restante do tempo, ele tem valor zero ou não tem significado. Outro
código que lê o campo não consegue saber quando ele contém um valor válido e quando não.

---

## 2. Sinais de alerta

- [ ] Um campo é zero ou `nil` na maior parte do tempo
- [ ] Um campo é definido em um método e consumido em outro, mas nunca mantido durante o ciclo de vida da struct
- [ ] Um campo existe apenas porque um algoritmo longo precisava de algum lugar para guardar estado intermediário
- [ ] A struct tem um padrão "preparar e executar" onde campos são preenchidos logo antes do uso
- [ ] Verificações de nil/zero para campos da struct aparecem em vários métodos receptores

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| Campos temporários e os métodos que os usam podem ser extraídos | Extract Class (Extract Struct) |
| Estado nil precisa ser tratado pelos chamadores | Introduce Null Object |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// CalculadoraRota contém campos que só são válidos durante Calcular()
type CalculadoraRota struct {
	paradas        []string // só definido durante o cálculo
	distanciaTotal int      // só válido após Calcular()
	caminhoRapido  string   // vazio a não ser que Calcular() tenha sido chamado
}

func (r *CalculadoraRota) DefinirParadas(paradas []string) {
	r.paradas = paradas
}

func (r *CalculadoraRota) Calcular() {
	r.distanciaTotal = calcularDistancia(r.paradas)
	r.caminhoRapido = encontrarCaminhoRapido(r.paradas)
}

func (r *CalculadoraRota) CaminhoRapido() string {
	return r.caminhoRapido // string vazia se Calcular() não foi chamado
}
```

**DEPOIS — esperado:**
```go
// ResultadoRota só existe quando tem dados válidos
type ResultadoRota struct {
	DistanciaTotal int
	CaminhoRapido  string
}

// CalculadoraRota agora é sem estado
type CalculadoraRota struct{}

func (r *CalculadoraRota) Calcular(paradas []string) ResultadoRota {
	return ResultadoRota{
		DistanciaTotal: calcularDistancia(paradas),
		CaminhoRapido:  encontrarCaminhoRapido(paradas),
	}
}
```

**Por que esse padrão:**
- `ResultadoRota` só existe quando contém dados válidos — sem estado zero para proteger
- `CalculadoraRota` sem estado é segura para uso concorrente em múltiplas goroutines

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Adicionar flag `pronto` em vez de corrigir o design**
```go
// Não aceito — disciplina de flag é frágil; Extract Struct elimina a necessidade
type CalculadoraRota struct {
	calculado     bool
	caminhoRapido string
}

func (r *CalculadoraRota) CaminhoRapido() string {
	if !r.calculado {
		panic("chame Calcular() primeiro")
	}
	return r.caminhoRapido
}
```

**Erro 2: Mover o campo temporário para uma struct embutida**
```go
// Não aceito — o campo ainda viaja com a struct externa; o smell permanece
type estadoCalculo struct {
	caminhoRapido string
}

type CalculadoraRota struct {
	estadoCalculo // mesmo problema, só renomeado
}
```

---

## 6. Benefícios

- **Correção:** Sem estado zero mascarando valor válido
- **Clareza:** Todo campo da struct tem significado a qualquer momento
- **Concorrência:** Calculadoras sem estado podem ser chamadas de múltiplas goroutines com segurança
