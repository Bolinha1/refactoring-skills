# SKILL: Detectando e Refatorando Código Morto — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/dead-code

---

## 1. O que é Código Morto

Variáveis, funções, structs, imports ou blocos de código que não são mais usados por nenhuma parte do sistema. Esse código é mantido por medo ou por descuido, mas só polui o código-base, confunde os leitores e aumenta o custo de manutenção.

**Por que isso acontece:**
- Requisitos mudaram e o código antigo não foi removido
- Refatorações incompletas deixaram artefatos para trás
- Código foi comentado "temporariamente" e nunca deletado
- Flags de feature nunca removidas após o lançamento

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Funções ou métodos que nunca são chamados
- [ ] Variáveis declaradas mas nunca lidas (o compilador Go já avisa para variáveis locais)
- [ ] Imports não utilizados (o compilador Go já bloqueia isso)
- [ ] Structs ou tipos que nunca são instanciados
- [ ] Código dentro de condição que nunca pode ser verdadeira (`if false`, `if versao == 0`)
- [ ] Blocos de código comentados que existem há sprints

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Função exportada sem uso interno | Verificar uso externo; remover se não usada |
| Código comentado | Deletar — o histórico vive no git |
| Condição sempre falsa | Delete Dead Code |
| Flag de feature obsoleta | Remover a flag e o código do caminho inativo |
| Campo de struct nunca lido | Remover o campo |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
package pedido

// calcularFreteAntigo foi substituído pelo ServicoFrete externo
// func calcularFreteAntigo(peso float64) float64 {
//     return peso * 2.5
// }

type Pedido struct {
    ID       int
    Total    float64
    Desconto float64 // nunca usado desde v2
    // CodLegado string // campo legado removido mas comentário permaneceu
}

func (p *Pedido) Processar() error {
    // TODO: remover após migração completa (sprint 12)
    if false {
        fmt.Println("modo debug ativo")
    }
    return p.validar()
}

// gerarHashLegado nunca chamada após migração para JWT
func gerarHashLegado(id int) string {
    return fmt.Sprintf("hash-%d", id)
}
```

**DEPOIS — esperado:**
```go
package pedido

type Pedido struct {
    ID    int
    Total float64
}

func (p *Pedido) Processar() error {
    return p.validar()
}
```

**Por que este padrão:**
- O compilador Go já força a remoção de imports e variáveis não utilizadas
- Remover código morto explícito (funções, campos, comentários) reduz ruído cognitivo

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter código morto "por precaução"**
```go
// Não aceito — o git preserva o histórico; não há motivo para manter código morto no código-fonte
// func calcularFreteAntigo(peso float64) float64 { ... }
```

**Erro 2: Adicionar tag de build para desativar em vez de deletar**
```go
// Não aceito — cria código zumbi que ninguém mantém
//go:build ignore
func rotinaDesnecessaria() {}
```

---

## 6. Benefícios

- **Clareza:** Menos código para ler e entender ao navegar pela base de código
- **Confiança:** Desenvolvedores não desperdiçam tempo tentando entender código que não faz nada
- **Velocidade:** Compilações e análises estáticas são mais rápidas com menos código
