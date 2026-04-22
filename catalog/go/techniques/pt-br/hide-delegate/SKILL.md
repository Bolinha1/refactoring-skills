# TÉCNICA: Ocultar Delegado — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/hide-delegate

---

## 1. Problema

O cliente precisa conhecer o objeto intermediário (delegado) para obter um serviço. Isso cria acoplamento: se o delegado mudar, o cliente precisa ser atualizado. O cliente navega por uma cadeia de objetos para chegar ao que precisa.

---

## 2. Solução

Crie um método na struct intermediária que encapsula a chamada ao delegado. O cliente chama apenas a struct intermediária, sem precisar saber que um delegado existe.

---

## 3. Quando aplicar

- O cliente chama `a.B().Metodo()` repetidamente
- O campo intermediário `B` é um detalhe de implementação que não deveria vazar para fora
- Mudar o tipo de `B` obrigaria a alterar todos os clientes
- A chain de chamadas viola a Lei de Demeter

---

## 4. Passos de refatoração

1. Identifique os métodos do delegado que o cliente usa
2. Para cada método usado, crie um método de encaminhamento na struct intermediária
3. Atualize o cliente para chamar o método de encaminhamento em vez de navegar até o delegado
4. Se o campo do delegado não for mais acessado diretamente de fora, torne-o unexported
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
type Departamento struct {
    Gerente *Pessoa
}

type Pessoa struct {
    Nome       string
    Departamento *Departamento
}

// cliente precisa conhecer Departamento para chegar ao Gerente
gerente := pessoa.Departamento.Gerente
```

**DEPOIS — esperado:**
```go
type Departamento struct {
    gerente *Pessoa // unexported — detalhe interno
}

func (d *Departamento) Gerente() *Pessoa {
    return d.gerente
}

type Pessoa struct {
    Nome         string
    departamento *Departamento
}

// método de encaminhamento — oculta o delegado
func (p *Pessoa) Gerente() *Pessoa {
    return p.departamento.Gerente()
}

// cliente não conhece Departamento
gerente := pessoa.Gerente()
```

**Por que esse padrão:**
- O cliente não precisa saber que `Pessoa` tem um `Departamento`
- Se a estrutura interna mudar, apenas `Pessoa` precisa ser atualizada

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Expor o campo delegado diretamente**
```go
// Não aceito — o cliente acessa o campo público e cria acoplamento
type Pessoa struct {
    Departamento *Departamento // exposto — o cliente pode navegar livremente
}
```

**Erro 2: Criar encaminhamentos para todos os métodos do delegado sem necessidade**
```go
// Não aceito — se o cliente precisa de toda a interface de Departamento,
// talvez seja melhor repensar o design em vez de criar dezenas de encaminhamentos
```

---

## 7. Benefícios

- **Baixo acoplamento:** O cliente depende apenas da struct intermediária, não do delegado
- **Encapsulamento:** Detalhes de implementação ficam ocultos
- **Manutenibilidade:** Mudar o delegado não impacta os clientes
