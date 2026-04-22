# TÉCNICA: Introduzir Objeto Parâmetro — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/introduce-parameter-object

---

## 1. Problema

Um grupo de parâmetros sempre aparece junto em várias funções. Passar esses parâmetros individualmente cria ruído nas assinaturas e torna difícil adicionar um novo campo sem alterar todos os chamadores.

---

## 2. Solução

Agrupe os parâmetros relacionados em uma struct. Substitua os parâmetros individuais por um único parâmetro do tipo da nova struct.

---

## 3. Quando aplicar

- Os mesmos 3 ou mais parâmetros aparecem juntos em várias funções
- Parâmetros representam um conceito coeso (ex: intervalo de datas, critérios de filtro)
- Adicionar um novo parâmetro exigiria alterar todas as assinaturas
- Os parâmetros têm validação ou comportamento que poderia ficar encapsulado

---

## 4. Passos de refatoração

1. Crie uma nova struct com um nome que expresse o conceito agrupado
2. Adicione os campos correspondentes aos parâmetros a serem agrupados
3. Atualize a assinatura da função para receber a nova struct
4. Atualize os chamadores para criar e passar a struct
5. Execute os testes
6. Considere mover validação e comportamento para métodos da nova struct

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func (r *Relatorio) Gerar(dataInicio, dataFim time.Time, valorMinimo float64) []Transacao {
    // ...
}

func (r *Relatorio) Resumir(dataInicio, dataFim time.Time, valorMinimo float64) Resumo {
    // ...
}
```

**DEPOIS — esperado:**
```go
type FiltroRelatorio struct {
    DataInicio  time.Time
    DataFim     time.Time
    ValorMinimo float64
}

func (f FiltroRelatorio) EhValido() bool {
    return f.DataFim.After(f.DataInicio) && f.ValorMinimo >= 0
}

func (r *Relatorio) Gerar(filtro FiltroRelatorio) []Transacao {
    if !filtro.EhValido() {
        return nil
    }
    // ...
}

func (r *Relatorio) Resumir(filtro FiltroRelatorio) Resumo {
    // ...
}
```

**Por que esse padrão:**
- `FiltroRelatorio` encapsula os critérios de filtragem como um conceito coeso
- Adicionar um novo critério exige alterar apenas a struct, não todas as assinaturas
- A validação fica encapsulada onde pertence

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Criar uma struct que agrupa parâmetros sem relação entre si**
```go
// Não aceito — ParametrosMisturados não representa um conceito coeso
type ParametrosMisturados struct {
    DataInicio time.Time
    NomeUsuario string
    LimitePagina int
    Debug bool
}
```

**Erro 2: Usar um map genérico em vez de uma struct tipada**
```go
// Não aceito — perde type safety e autocompletar da IDE
func (r *Relatorio) Gerar(params map[string]interface{}) []Transacao { ... }
```

---

## 7. Benefícios

- **Assinaturas limpas:** Funções recebem um único parâmetro expressivo em vez de vários
- **Extensibilidade:** Adicionar campos não altera assinaturas existentes
- **Encapsulamento:** Validação e comportamento ficam na struct, não espalhados nos chamadores
