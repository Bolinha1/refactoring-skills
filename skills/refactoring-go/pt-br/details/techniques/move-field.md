# TÉCNICA: Mover Campo — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/move-field

---

## 1. Problema

Um campo de uma struct é mais usado por outra struct do que pela que o contém. Isso indica que o campo está na struct errada, criando acoplamento desnecessário entre elas.

---

## 2. Solução

Mova o campo para a struct que mais o utiliza. Atualize todas as referências ao campo para acessá-lo pelo novo local.

---

## 3. Quando aplicar

- Métodos de outra struct acessam o campo com mais frequência do que a struct dona
- Ao mover um método, você percebe que ele precisa de um campo da struct original
- O campo representa um conceito que pertence semanticamente a outra struct

---

## 4. Passos de refatoração

1. Declare o campo na struct de destino
2. Decida como a struct original acessará o campo: via referência à struct de destino ou por parâmetro
3. Atualize todos os usos do campo na struct original para acessar via struct de destino
4. Remova o campo da struct original
5. Execute os testes após cada passo

---

## 5. Exemplo

**ANTES — não aceito:**
```go
type Conta struct {
    Saldo         float64
    TaxaJuros     float64 // acessado principalmente por ContaPoupanca
}

type ContaPoupanca struct {
    Conta *Conta
    Saldo float64
}

func (cp *ContaPoupanca) CalcularRendimento() float64 {
    return cp.Saldo * cp.Conta.TaxaJuros // acessa TaxaJuros de Conta
}
```

**DEPOIS — esperado:**
```go
type Conta struct {
    Saldo float64
    // TaxaJuros movida para ContaPoupanca
}

type ContaPoupanca struct {
    Conta     *Conta
    Saldo     float64
    TaxaJuros float64 // campo movido para cá
}

func (cp *ContaPoupanca) CalcularRendimento() float64 {
    return cp.Saldo * cp.TaxaJuros // acessa campo local
}
```

**Por que esse padrão:**
- `TaxaJuros` pertence semanticamente a `ContaPoupanca`, não a `Conta`
- Elimina a dependência de `ContaPoupanca` em campos de `Conta`

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover campo sem atualizar todas as referências**
```go
// Não aceito — campo duplicado nas duas structs cria duas fontes da verdade
type Conta struct {
    TaxaJuros float64 // antigo — deve ser removido
}
type ContaPoupanca struct {
    TaxaJuros float64 // novo
}
```

**Erro 2: Mover campo para uma struct que não tem contexto para mantê-lo**
```go
// Não aceito — mover TaxaJuros para uma struct genérica Configuracao
// quebra a coesão e torna o acesso mais complexo
```

---

## 7. Benefícios

- **Coesão:** Cada struct contém os campos que realmente lhe pertencem
- **Baixo acoplamento:** Structs dependem menos umas das outras
- **Clareza:** O local do campo comunica a qual conceito ele pertence
