# SKILL: Detectando e Refatorando Feature Envy — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/feature-envy

---

## 1. O que é Feature Envy

Uma função ou método acessa os dados de outra struct mais do que acessa os dados de sua própria struct. O método tem "inveja" da outra struct — parece querer viver lá.

**Por que isso acontece:**
- Campos foram movidos para uma nova struct mas as funções que os usam não foram
- Lógica de negócio foi colocada em um serviço que opera nos internos de um objeto de domínio
- Uma função foi gradualmente estendida para depender cada vez mais dos campos de outra struct

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Uma função acessa vários campos de outra struct em sequência
- [ ] Uma função recebe uma struct como parâmetro e usa principalmente seus campos
- [ ] Uma função ignora completamente seu receptor e só manipula o estado de outro objeto
- [ ] O nome da função naturalmente pertence à outra struct (`calcularTotalPedido` em um `ServicoRelatorio`)

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| A função inteira pertence à outra struct | Move Method |
| Apenas parte da função tem inveja de outra struct | Extract Function, depois Move Method |
| A função usa dados de várias structs | Mover para a struct com mais dados |
| A função é um cálculo que pertence ao modelo de domínio | Move Method para struct de domínio |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
type ImpressoraFatura struct{}

// calcularTotalFatura só usa dados de Fatura — inveja clara
func (imp *ImpressoraFatura) calcularTotalFatura(fatura *Fatura) float64 {
    subtotal := 0.0
    for _, item := range fatura.Itens {
        subtotal += item.Preco * float64(item.Quantidade)
    }
    imposto := subtotal * fatura.TaxaImposto
    desconto := 0.0
    if fatura.TemDesconto {
        desconto = subtotal * fatura.TaxaDesconto
    }
    return subtotal + imposto - desconto
}
```

**DEPOIS — esperado:**
```go
type Fatura struct {
    Itens        []ItemFatura
    TaxaImposto  float64
    TaxaDesconto float64
    TemDesconto  bool
}

// Cálculo pertence à Fatura — só usa seus próprios dados
func (f *Fatura) CalcularTotal() float64 {
    subtotal := 0.0
    for _, item := range f.Itens {
        subtotal += item.Preco * float64(item.Quantidade)
    }
    imposto := subtotal * f.TaxaImposto
    desconto := 0.0
    if f.TemDesconto {
        desconto = subtotal * f.TaxaDesconto
    }
    return subtotal + imposto - desconto
}

type ImpressoraFatura struct{}

func (imp *ImpressoraFatura) Imprimir(fatura *Fatura) {
    total := fatura.CalcularTotal() // delega ao dono dos dados
    // ... lógica de impressão
}
```

**Por que este padrão:**
- `CalcularTotal` naturalmente pertence à `Fatura` — só usa dados da Fatura
- `ImpressoraFatura` é liberada de conhecer os internos da Fatura

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter a função invejosa e adicionar um pass-through**
```go
// Não aceito — delegação sem mover cria indireção, não clareza
func (imp *ImpressoraFatura) calcularTotalFatura(fatura *Fatura) float64 {
    return fatura.CalcularTotal() // apenas wrapper — delete a função invejosa
}
```

**Erro 2: Mover o método mas manter parâmetros que expõem internos**
```go
// Não aceito — movido mas ainda passando valores de campo brutos
func (f *Fatura) CalcularTotal(subtotal, taxaImposto, taxaDesconto float64, temDesconto bool) float64 { ... }
```

---

## 6. Benefícios

- **Coesão:** Cada struct possui o comportamento que opera sobre seus próprios dados
- **Encapsulamento:** O objeto de domínio expõe intenção, não campos brutos
- **Manutenibilidade:** As regras de negócio são colocalizadas com os dados que governam
