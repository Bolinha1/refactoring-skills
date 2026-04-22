# TÉCNICA: Internalizar Struct — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/inline-class

---

## 1. Problema

Uma struct não faz mais nada de útil por conta própria. Ela contém poucos campos e métodos, e sua responsabilidade foi absorvida por outras structs ao longo de refatorações anteriores. Manter essa struct em separado adiciona complexidade desnecessária.

---

## 2. Solução

Mova todos os campos e métodos da struct para outra struct que a utiliza. Delete a struct vazia.

---

## 3. Quando aplicar

- A struct tem pouquíssimos campos e métodos após refatorações
- A struct não tem uma responsabilidade clara e distinta
- A struct é usada em apenas um lugar
- A complexidade de manter a indireção supera o benefício

---

## 4. Passos de refatoração

1. Declare os campos da struct a ser eliminada diretamente na struct receptora
2. Copie os métodos da struct eliminada para a struct receptora, ajustando referências
3. Atualize todos os chamadores para usar a struct receptora diretamente
4. Execute os testes
5. Delete a struct original

---

## 5. Exemplo

**ANTES — não aceito:**
```go
type TelefoneSimples struct {
    Numero string
}

func (t *TelefoneSimples) Formatar() string {
    return t.Numero
}

type Pessoa struct {
    Nome     string
    Telefone TelefoneSimples
}

func (p *Pessoa) NumeroTelefone() string {
    return p.Telefone.Formatar()
}
```

**DEPOIS — esperado:**
```go
// TelefoneSimples foi eliminado — campo e lógica absorvidos por Pessoa
type Pessoa struct {
    Nome           string
    NumeroContato  string
}

func (p *Pessoa) NumeroTelefone() string {
    return p.NumeroContato
}
```

**Por que esse padrão:**
- `TelefoneSimples` era uma indireção sem valor — apenas armazenava uma string
- Eliminar a struct reduz a quantidade de tipos que o leitor precisa compreender

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Internalizar uma struct que ainda tem responsabilidades reais**
```go
// Não aceito — Endereco tem validação, formatação e lógica própria;
// internalizá-la espalha essa responsabilidade por outras structs
type Endereco struct {
    Rua    string
    Cidade string
    CEP    string
}
func (e *Endereco) Validar() error { ... }
func (e *Endereco) Formatar() string { ... }
```

**Erro 2: Internalizar uma struct usada em múltiplos lugares**
```go
// Não aceito — se Contato é usado por Pessoa, Empresa e Fornecedor,
// internalizá-lo duplica campos e lógica em todos os lugares
```

---

## 7. Benefícios

- **Simplicidade:** Menos tipos para compreender e manter
- **Clareza:** Elimina indireções que não agregam significado
- **Manutenibilidade:** Código mais direto e fácil de navegar
