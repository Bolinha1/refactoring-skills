# TÉCNICA: Extrair Struct — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/extract-class

---

## 1. Problema

Uma struct faz o trabalho de duas. Um subconjunto de seus campos e métodos forma um conceito coeso que faria sentido como sua própria struct. Em Go não há herança, mas composição e interfaces permitem extrair responsabilidades com clareza.

---

## 2. Solução

Crie uma nova struct e mova os campos e métodos que pertencem a esse conceito para ela. Substitua os campos originais por um campo do tipo da nova struct na struct original.

---

## 3. Quando aplicar

- A struct tem campos que são sempre usados juntos (Data Clumps)
- Existem métodos que operam apenas sobre um subconjunto dos campos
- A struct cresceu para lidar com mais de uma responsabilidade distinta
- Um subconjunto poderia ser reutilizado independentemente em outro contexto
- O nome da struct não descreve o que ela faz sem usar "e"

---

## 4. Passos de refatoração

1. Identifique o grupo de campos e métodos a extrair — eles devem formar um conceito coeso
2. Crie uma nova struct com um nome que expresse esse conceito
3. Adicione um campo do novo tipo na struct original
4. Mova os campos identificados para a nova struct (use Move Field para cada um)
5. Mova os métodos identificados para a nova struct (use Move Method para cada um)
6. Atualize a struct original para delegar ao campo embutido ou acessar via referência
7. Compile e execute os testes após cada movimentação

---

## 5. Exemplo

**ANTES — não aceito:**
```go
type Pessoa struct {
    Nome              string
    CodigoAreaEscritorio string
    NumeroEscritorio  string
}

func (p *Pessoa) NumeroTelefone() string {
    return "(" + p.CodigoAreaEscritorio + ") " + p.NumeroEscritorio
}
```

**DEPOIS — esperado:**
```go
type TelefoneEscritorio struct {
    CodigoArea string
    Numero     string
}

func (t *TelefoneEscritorio) Formatar() string {
    return "(" + t.CodigoArea + ") " + t.Numero
}

type Pessoa struct {
    Nome      string
    Telefone  TelefoneEscritorio
}

func (p *Pessoa) NumeroTelefone() string {
    return p.Telefone.Formatar()
}
```

**Por que esse padrão:**
- `TelefoneEscritorio` é um conceito independente e reutilizável
- A lógica de formatação fica encapsulada onde pertence

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair uma struct sem coesão**
```go
// Não aceito — DadosPessoa agrupa campos sem relacionamento natural
type DadosPessoa struct {
    Nome           string
    CodigoEscritorio string
    Salario        float64
    DataContratacao time.Time
}
```

**Erro 2: Mover métodos sem mover os campos que eles precisam**
```go
// Não aceito — Formatar ainda recebe codigoArea de fora, acoplado à Pessoa
func (t *TelefoneEscritorio) Formatar(codigoArea, numero string) string { ... }
```

**Erro 3: Extrair mas deixar os campos originais no lugar**
```go
// Não aceito — duas fontes da verdade para dados de telefone
type Pessoa struct {
    Telefone            TelefoneEscritorio // novo
    CodigoAreaEscritorio string            // antigo — deve ser removido
    NumeroEscritorio    string            // antigo — deve ser removido
}
```

---

## 7. Benefícios

- **Responsabilidade Única:** Cada struct tem um conceito claro para representar
- **Reutilização:** A struct extraída pode ser usada por outras structs independentemente
- **Encapsulamento:** Lógica de validação e formatação do conceito vive em um único lugar
