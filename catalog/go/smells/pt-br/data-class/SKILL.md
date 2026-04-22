# SKILL: Detectando e Refatorando Data Class — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/data-class

---

## 1. O que é Data Class

Uma struct que contém apenas campos e nenhum comportamento significativo. Outros pacotes ou structs manipulam seus dados diretamente em vez de delegar para ela. A struct é um recipiente passivo — existe apenas para carregar dados de um lado para o outro.

**Por que isso acontece:**
- Structs criadas como DTOs (Data Transfer Objects) e nunca evoluídas para objetos de domínio reais
- Comportamento relacionado foi colocado em serviços ou funções externas em vez de na própria struct
- Desenvolvedor evitou adicionar métodos por não reconhecer que a lógica pertencia ali

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Struct com vários campos mas nenhum método com lógica de negócio
- [ ] Funções externas que acessam campos da struct para calcular ou validar algo
- [ ] Código cliente que obtém dados da struct, realiza uma operação e devolve o resultado
- [ ] Validações da struct espalhadas por todo o código em vez de centralizadas
- [ ] Struct que só existe para ser passada entre chamadas de função

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Cálculo externo usa apenas dados da struct | Move Method para a struct |
| Validação espalhada por múltiplos locais | Move Method — adicionar método `Validar()` |
| Campos públicos sem encapsulamento | Encapsulate Field (usar métodos acessores ou construtor validado) |
| Apenas alguns campos são usados por cliente | Extract Interface para expor subconjunto |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
type Produto struct {
    Nome     string
    Preco    float64
    Estoque  int
    Categoria string
}

// Lógica espalhada fora da struct
func calcularPrecoComDesconto(p Produto, percentual float64) float64 {
    return p.Preco * (1 - percentual/100)
}

func produtoDisponivel(p Produto) bool {
    return p.Estoque > 0
}

func validarProduto(p Produto) error {
    if p.Nome == "" {
        return fmt.Errorf("nome é obrigatório")
    }
    if p.Preco <= 0 {
        return fmt.Errorf("preço deve ser positivo")
    }
    return nil
}
```

**DEPOIS — esperado:**
```go
type Produto struct {
    nome      string
    preco     float64
    estoque   int
    categoria string
}

func NovoProduto(nome string, preco float64, estoque int, categoria string) (*Produto, error) {
    p := &Produto{nome: nome, preco: preco, estoque: estoque, categoria: categoria}
    if err := p.Validar(); err != nil {
        return nil, err
    }
    return p, nil
}

func (p *Produto) Validar() error {
    if p.nome == "" {
        return fmt.Errorf("nome é obrigatório")
    }
    if p.preco <= 0 {
        return fmt.Errorf("preço deve ser positivo")
    }
    return nil
}

func (p *Produto) PrecoComDesconto(percentual float64) float64 {
    return p.preco * (1 - percentual/100)
}

func (p *Produto) Disponivel() bool {
    return p.estoque > 0
}
```

**Por que este padrão:**
- Toda lógica relacionada ao `Produto` vive dentro dele — encapsulamento real
- O construtor `NovoProduto` garante invariantes desde a criação

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Adicionar métodos triviais que apenas retornam campos**
```go
// Não aceito — getter sem lógica não resolve o smell
func (p *Produto) GetPreco() float64 { return p.Preco }
```

**Erro 2: Mover toda a lógica para um único método gigante**
```go
// Não aceito — cria Long Method em vez de distribuir responsabilidades
func (p *Produto) Processar(desconto float64, verificarEstoque bool) float64 { ... }
```

---

## 6. Benefícios

- **Encapsulamento:** A struct controla seus próprios dados e invariantes
- **Coesão:** Comportamento e dados relacionados vivem no mesmo lugar
- **Testabilidade:** Métodos da struct são testados diretamente, sem dependências externas
