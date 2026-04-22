# TÉCNICA: Remover Intermediário — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/remove-middle-man

---

## 1. Problema

Uma struct tem muitos métodos de encaminhamento que apenas delegam chamadas a outro objeto. A quantidade de delegações triviais supera o benefício do encapsulamento — a struct virou um intermediário inútil que apenas adiciona indireção.

---

## 2. Solução

Exponha o campo delegado diretamente (ou forneça um accessor) para que os clientes possam chamar o objeto delegado diretamente, eliminando os métodos de encaminhamento desnecessários.

---

## 3. Quando aplicar

- A maioria dos métodos de uma struct são apenas delegações para outro campo
- Cada novo método do delegado exige criar um novo encaminhamento
- O encapsulamento não está protegendo nenhum invariante real
- É a situação oposta de Hide Delegate — muita delegação sem benefício

---

## 4. Passos de refatoração

1. Crie um método accessor (getter) para o campo delegado, ou torne-o exported
2. Para cada método de encaminhamento que apenas delega:
   - Atualize os chamadores para acessar o delegado diretamente
   - Remova o método de encaminhamento
3. Execute os testes após cada remoção
4. Se o campo antes era unexported, avalie se deve continuar assim ou ser exportado

---

## 5. Exemplo

**ANTES — não aceito:**
```go
type Pessoa struct {
    departamento *Departamento
}

// todos esses métodos apenas delegam para departamento
func (p *Pessoa) Gerente() *Pessoa {
    return p.departamento.Gerente()
}

func (p *Pessoa) NomeDepartamento() string {
    return p.departamento.Nome()
}

func (p *Pessoa) OrcamentoDepartamento() float64 {
    return p.departamento.Orcamento()
}
```

**DEPOIS — esperado:**
```go
type Pessoa struct {
    Departamento *Departamento // exposto diretamente
}

// clientes acessam diretamente:
// pessoa.Departamento.Gerente()
// pessoa.Departamento.Nome()
// pessoa.Departamento.Orcamento()
```

**Por que esse padrão:**
- `Pessoa` não protegia nenhum invariante com os encaminhamentos
- Expor `Departamento` diretamente elimina boilerplate e simplifica a interface de `Pessoa`

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Remover delegações que protegem invariantes reais**
```go
// Não aceito — se o método de encaminhamento adiciona validação ou lógica,
// não é um simples intermediário
func (p *Pessoa) Gerente() *Pessoa {
    if p.departamento == nil {
        return nil // lógica real — não é apenas delegação
    }
    return p.departamento.Gerente()
}
```

**Erro 2: Expor o delegado quando isso viola encapsulamento importante**
```go
// Não aceito — expor o campo interno quando há lógica de negócio que depende
// do controle de acesso ao delegado
```

---

## 7. Benefícios

- **Simplicidade:** Elimina boilerplate de métodos que apenas repassam chamadas
- **Transparência:** O cliente acessa diretamente o que precisa sem camadas desnecessárias
- **Manutenibilidade:** Menos código para manter quando a interface do delegado muda
