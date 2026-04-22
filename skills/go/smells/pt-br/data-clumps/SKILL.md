# SKILL: Detectando e Refatorando Data Clumps — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/data-clumps

---

## 1. O que é Data Clumps

Partes diferentes do código contêm grupos idênticos de campos — os mesmos campos em múltiplas structs, ou os mesmos parâmetros aparecendo juntos em muitas assinaturas de função. Se você removesse um item do grupo e os demais perdessem sentido, você tem um data clump.

**Por que isso acontece:**
- O relacionamento entre os dados nunca foi formalizado como uma struct
- Copy-paste propagou o grupo por lugares não relacionados
- O conceito existia informalmente na cabeça dos desenvolvedores, mas não no código

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Três ou mais campos que aparecem juntos em múltiplas structs
- [ ] Os mesmos 2–3 parâmetros consistentemente passados juntos para funções
- [ ] Variáveis como `dataInicio`/`dataFim`, `latitude`/`longitude`, `rua`/`cidade`/`cep` como primitivos separados
- [ ] Você precisa atualizar o mesmo grupo de campos em vários lugares para uma única mudança lógica

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Clump aparece como campos em uma struct | Extract Class (nova struct) |
| Clump aparece em listas de parâmetros de funções | Introduce Parameter Object |
| Uma função recebe uma struct mas usa apenas parte dela | Preserve Whole Object |

**Teste chave:** remova um item do clump. Se os outros perderem significado, o grupo merece sua própria struct.

---

## 4. Exemplo

**ANTES — não aceito:**
```go
type Pedido struct {
    Rua        string
    Cidade     string
    CEP        string
    Pais       string
    NomeCliente  string
    EmailCliente string
}

type Fatura struct {
    Rua    string // mesmo clump novamente
    Cidade string
    CEP    string
    Pais   string
}

func enviar(rua, cidade, cep, pais string) error {
    // lógica de envio
    return nil
}
```

**DEPOIS — esperado:**
```go
type Endereco struct {
    Rua    string
    Cidade string
    CEP    string
    Pais   string
}

func (e Endereco) Valido() bool {
    return e.Rua != "" && e.Cidade != "" && e.CEP != ""
}

type Pedido struct {
    EnderecoEntrega Endereco
    NomeCliente     string
    EmailCliente    string
}

type Fatura struct {
    EnderecoCobranca Endereco
}

func enviar(destino Endereco) error {
    // lógica de envio
    return nil
}
```

**Por que este padrão:**
- `Endereco` é um conceito nomeado — sua validação e formatação vivem em um único lugar
- Toda struct que possui um endereço se beneficia de qualquer melhoria em `Endereco`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Agrupar o clump em um map genérico**
```go
// Não aceito — perde segurança de tipos e intenção
endereco := map[string]string{
    "rua":    "Rua Principal",
    "cidade": "São Paulo",
}
```

**Erro 2: Criar a struct mas manter os campos separados antigos também**
```go
// Não aceito — agora há duas representações dos mesmos dados
type Pedido struct {
    EnderecoEntrega Endereco
    Rua             string // duplicado
    Cidade          string // duplicado
}
```

---

## 6. Benefícios

- **Fonte única da verdade:** A lógica do clump (validação, formatação) é centralizada
- **Legibilidade:** `pedido.EnderecoEntrega` é mais expressivo do que quatro campos separados
- **Extensibilidade:** Adicionar um novo campo de endereço requer mudança apenas na struct `Endereco`
