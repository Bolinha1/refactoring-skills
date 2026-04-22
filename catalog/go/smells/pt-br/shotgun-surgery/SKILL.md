# SKILL: Detectando e Refatorando Shotgun Surgery — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/shotgun-surgery

---

## 1. O que é Shotgun Surgery

Uma única mudança lógica exige modificar muitos pacotes ou structs diferentes ao mesmo
tempo. Fazer uma mudança conceitual "dispara" edições por todo o código como uma
escopeta — tocando muitos arquivos para o que deveria ser uma correção localizada.

O inverso de Divergent Change: muitos tipos, um motivo para mudar.

**Por que isso acontece:**
- Uma responsabilidade foi fragmentada entre muitos tipos em vez de estar em um lugar
- Lógica que deveria ser centralizada foi duplicada por "flexibilidade"
- Conceitos transversais (formatação de moeda, validação, logging) foram tratados inline em todo lugar

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Adicionar uma funcionalidade exige editar 5+ structs ou pacotes não relacionados
- [ ] Renomear um conceito exige tocar dezenas de arquivos
- [ ] Uma única regra de negócio é aplicada em múltiplos lugares
- [ ] Você sempre faz a mesma mudança em vários pacotes simultaneamente
- [ ] Falhas de teste se propagam por muitos pacotes para uma mudança de conceito único

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| Comportamento espalhado relaciona-se a um conceito | Move Method + Move Field |
| Múltiplos tipos pequenos contribuem para um conceito | Inline Class |
| Conceito transversal repetido em todo lugar | Extract Class (nova struct) |
| Regra de negócio duplicada em muitos lugares | Move Method para um único dono |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// Adicionar conceito de "moeda" exige mudar TODOS estes:

type Produto struct {
	Preco float64 // precisa adicionar campo Moeda aqui
}

func (p *Pedido) Total() float64 { ... } // precisa formatar com moeda

func (n *NotaFiscal) FormatarValor(valor float64) string {
	return fmt.Sprintf("R$%.2f", valor) // precisa usar moeda
}

func (r *Recibo) ImprimirTotal(valor float64) {
	fmt.Println(valor) // precisa usar moeda
}
```

**DEPOIS — esperado:**
```go
type Dinheiro struct {
	Valor  float64
	Moeda  string
}

func (d Dinheiro) Formatar() string {
	switch d.Moeda {
	case "BRL":
		return fmt.Sprintf("R$%.2f", d.Valor)
	case "USD":
		return fmt.Sprintf("$%.2f", d.Valor)
	default:
		return fmt.Sprintf("%.2f %s", d.Valor, d.Moeda)
	}
}

func (d Dinheiro) Somar(outro Dinheiro) (Dinheiro, error) {
	if d.Moeda != outro.Moeda {
		return Dinheiro{}, fmt.Errorf("moedas diferentes: %s e %s", d.Moeda, outro.Moeda)
	}
	return Dinheiro{Valor: d.Valor + outro.Valor, Moeda: d.Moeda}, nil
}

// Agora Produto, Pedido, NotaFiscal e Recibo usam Dinheiro
type Produto struct {
	Preco Dinheiro
}
```

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Centralizar dados mas não comportamento**
```go
// Não aceito — agrupa campos mas formatação/aritmética continuam espalhadas
type DinheiroDTO struct {
	Valor       float64
	CodigoMoeda string
}
```

**Erro 2: Função utilitária de pacote como ponto único**
```go
// Não aceito — chamadores ainda espalham lógica; a função é apenas um dump nomeado
func FormatarDinheiro(valor float64, moeda string) string { ... }
```

---

## 6. Benefícios

- **Localidade:** Um conceito de negócio = um único tipo para alterar
- **Segurança:** Mudanças são fáceis de raciocinar quando o raio de impacto é pequeno
- **Descoberta:** Todo comportamento de um conceito está em um lugar óbvio
