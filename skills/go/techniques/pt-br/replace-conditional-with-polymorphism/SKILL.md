# TÉCNICA: Substituir Condicional por Polimorfismo — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/replace-conditional-with-polymorphism

---

## 1. Problema

Um switch ou cadeia de if/else seleciona comportamentos diferentes com base no tipo de um objeto. Toda vez que um novo tipo é adicionado, o switch precisa ser atualizado em vários lugares do código. Em Go, polimorfismo é obtido via interfaces, não herança.

---

## 2. Solução

Defina uma interface que expresse o comportamento variante. Crie uma struct para cada caso, implementando a interface. Substitua o switch por dispatch via interface.

---

## 3. Quando aplicar

- O mesmo switch (ou cadeia de if/else) sobre um tipo aparece em múltiplos lugares
- Adicionar um novo tipo requer alterar vários switches espalhados no código
- Cada branch do switch representa um comportamento coeso que poderia ser encapsulado
- O comportamento varia por tipo de objeto, não por estado transitório

---

## 4. Passos de refatoração

1. Defina uma interface que declara o método que varia por tipo
2. Para cada branch do switch, crie uma struct que implementa a interface
3. Substitua o switch por uma chamada ao método da interface
4. Certifique-se de que a criação das structs concretas está centralizada (factory ou construtor)
5. Execute os testes
6. Remova o switch original

---

## 5. Exemplo

**ANTES — não aceito:**
```go
type Funcionario struct {
    Tipo        string
    SalarioBase float64
}

func (f *Funcionario) CalcularBonus() float64 {
    switch f.Tipo {
    case "ENGENHEIRO":
        return f.SalarioBase * 0.10
    case "GERENTE":
        return f.SalarioBase*0.20 + 1000
    case "VENDEDOR":
        return f.SalarioBase * 0.15
    }
    return 0
}
```

**DEPOIS — esperado:**
```go
type CalculadorBonus interface {
    CalcularBonus() float64
}

type Engenheiro struct {
    SalarioBase float64
}

func (e *Engenheiro) CalcularBonus() float64 {
    return e.SalarioBase * 0.10
}

type Gerente struct {
    SalarioBase float64
}

func (g *Gerente) CalcularBonus() float64 {
    return g.SalarioBase*0.20 + 1000
}

type Vendedor struct {
    SalarioBase float64
}

func (v *Vendedor) CalcularBonus() float64 {
    return v.SalarioBase * 0.15
}

// uso via interface — sem switch
func processarBonus(calc CalculadorBonus) float64 {
    return calc.CalcularBonus()
}
```

**Por que esse padrão:**
- Adicionar um novo tipo de funcionário não requer alterar nenhum switch existente
- Cada struct encapsula as regras de bonificação do seu tipo

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter o switch e apenas mover para outro lugar**
```go
// Não aceito — o switch ainda existe, apenas mudou de endereço
func novaFuncao(tipo string) CalculadorBonus {
    switch tipo {
    case "ENGENHEIRO": return &Engenheiro{}
    // ...
    }
}
```

**Erro 2: Usar type assertion em vez de interface dispatch**
```go
// Não aceito — type assertion é outro tipo de switch disfarçado
if eng, ok := calc.(*Engenheiro); ok {
    return eng.SalarioBase * 0.10
}
```

---

## 7. Benefícios

- **Aberto/Fechado:** Novos tipos podem ser adicionados sem modificar código existente
- **Coesão:** Cada struct encapsula o comportamento do seu tipo
- **Testabilidade:** Cada implementação pode ser testada independentemente
