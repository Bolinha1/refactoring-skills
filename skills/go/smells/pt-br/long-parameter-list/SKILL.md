# SKILL: Detectando e Refatorando Long Parameter List — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/long-parameter-list

---

## 1. O que é Long Parameter List

Uma função ou método que recebe mais parâmetros do que é possível entender e lembrar
facilmente. Mais de três ou quatro parâmetros já tornam a assinatura difícil de usar
corretamente.

**Por que isso acontece:**
- Vários algoritmos foram mesclados em um único método
- Tentativa de tornar a função mais genérica adicionando parâmetros de controle
- Ausência de structs de domínio para agrupar dados relacionados

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Função com mais de 3 ou 4 parâmetros
- [ ] Vários parâmetros que sempre aparecem juntos em chamadas
- [ ] Parâmetros booleanos que controlam o comportamento da função
- [ ] Parâmetro passado apenas para repassá-lo a outra função
- [ ] Dificuldade de lembrar a ordem dos parâmetros ao chamar a função

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| Parâmetros que sempre andam juntos | Introduce Parameter Object |
| Parâmetro obtido de outro objeto | Replace Parameter with Method Call |
| Objeto inteiro disponível, mas só partes são passadas | Preserve Whole Object |
| Parâmetro booleano que bifurca o comportamento | Separar em duas funções distintas |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
func gerarRelatorio(
	dataInicio time.Time,
	dataFim time.Time,
	valorMinimo float64,
	valorMaximo float64,
	tipoCliente string,
	incluirCancelados bool,
) []Relatorio {
	// ...
}
```

**DEPOIS — esperado:**
```go
type FiltroRelatorio struct {
	DataInicio        time.Time
	DataFim           time.Time
	ValorMinimo       float64
	ValorMaximo       float64
	TipoCliente       string
	IncluirCancelados bool
}

func gerarRelatorio(filtro FiltroRelatorio) []Relatorio {
	// ...
}

// Chamada muito mais legível:
relatorios := gerarRelatorio(FiltroRelatorio{
	DataInicio:  time.Now().AddDate(0, -1, 0),
	DataFim:     time.Now(),
	TipoCliente: "premium",
})
```

**Por que esse padrão:**
- Os campos do struct têm nomes — não é possível inverter a ordem por engano
- Novos filtros podem ser adicionados sem quebrar chamadas existentes
- A chamada com struct literal é autodocumentada

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Criar um struct genérico para tudo**
```go
// Não aceito — struct genérico sem semântica de domínio
type Opcoes map[string]interface{}
func gerarRelatorio(opts Opcoes) []Relatorio { ... }
```

**Erro 2: Usar parâmetro booleano para bifurcar comportamento**
```go
// Não aceito — o booleano esconde dois comportamentos distintos em uma função
func calcularFrete(peso float64, expresso bool) float64 { ... }
// Melhor: duas funções separadas
func calcularFreteStandard(peso float64) float64 { ... }
func calcularFreteExpresso(peso float64) float64 { ... }
```

---

## 6. Benefícios

- **Legibilidade:** Chamadas com structs literais são autodocumentadas
- **Segurança:** Não é possível trocar a ordem dos argumentos por engano
- **Extensibilidade:** Adicionar um novo parâmetro não quebra chamadas existentes
- **Manutenção:** A struct agrupa parâmetros relacionados em um conceito de domínio
