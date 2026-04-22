# TÉCNICA: Mover Método — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/move-method

---

## 1. Problema

Um método usa mais dados e comportamentos de outra struct do que da struct em que está definido. Isso cria Feature Envy e acopla a struct errada a detalhes que não lhe pertencem.

---

## 2. Solução

Crie um novo método na struct que o método mais utiliza. Transforme o método original em uma delegação simples ou remova-o completamente.

---

## 3. Quando aplicar

- O método acessa campos ou chama métodos de outra struct com mais frequência do que os da própria struct
- A struct atual tem poucas responsabilidades e o método não se encaixa bem
- Mover o método reduziria o acoplamento entre as duas structs
- Você está aplicando Extract Class e precisa levar métodos junto com campos

---

## 4. Passos de refatoração

1. Verifique se a struct de destino é acessível a partir do método
2. Declare o método na struct de destino com a mesma assinatura
3. Copie o corpo do método, ajustando referências a campos e outros métodos
4. Decida como a struct original acessará o método movido: via delegação ou removendo o método original
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
type Conta struct {
    Saldo     float64
    Tipo      *TipoConta
}

// método usa principalmente dados de TipoConta
func (c *Conta) CalcularJuros() float64 {
    return c.Saldo * c.Tipo.TaxaMensal * c.Tipo.FatorRisco
}
```

**DEPOIS — esperado:**
```go
type TipoConta struct {
    TaxaMensal  float64
    FatorRisco  float64
}

// método movido para onde os dados residem
func (t *TipoConta) CalcularJuros(saldo float64) float64 {
    return saldo * t.TaxaMensal * t.FatorRisco
}

type Conta struct {
    Saldo float64
    Tipo  *TipoConta
}

// delegação simples na struct original (ou pode ser removida se não necessária)
func (c *Conta) CalcularJuros() float64 {
    return c.Tipo.CalcularJuros(c.Saldo)
}
```

**Por que esse padrão:**
- `CalcularJuros` usava principalmente dados de `TipoConta`
- Mover o método torna `TipoConta` mais coeso e testável de forma independente

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover método mas manter uma cópia na struct original sem marcá-la como delegação**
```go
// Não aceito — duas implementações idênticas do mesmo comportamento
func (c *Conta) CalcularJuros() float64 {
    return c.Saldo * c.Tipo.TaxaMensal * c.Tipo.FatorRisco // duplicado
}
```

**Erro 2: Mover método para struct errada porque ela é maior**
```go
// Não aceito — mover para a struct maior, não para a que tem os dados relevantes
```

---

## 7. Benefícios

- **Coesão:** Cada struct contém os métodos que pertencem ao seu conceito
- **Baixo acoplamento:** Reduz dependências cruzadas entre structs
- **Testabilidade:** Métodos na struct correta são mais fáceis de testar isoladamente
