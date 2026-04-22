# SKILL: Detectando e Refatorando Incomplete Library Class — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/incomplete-library-class

---

## 1. O que é Incomplete Library Class

Uma biblioteca ou pacote externo não oferece a funcionalidade necessária, levando desenvolvedores a duplicar código, usar reflexão hacky ou espalhar workarounds pela base de código. Em vez de adaptar a biblioteca de forma estruturada, soluções pontuais se multiplicam.

**Por que isso acontece:**
- A biblioteca foi projetada para casos de uso diferentes do seu
- O pacote externo não pode ser modificado diretamente (código de terceiros)
- Funcionalidades foram adicionadas ao redor da biblioteca sem criar uma abstração adequada

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] O mesmo workaround para uma limitação de biblioteca aparece em vários lugares
- [ ] Funções utilitárias espalhadas que "consertam" ou "extendem" um tipo externo
- [ ] Conversões repetitivas entre o tipo da biblioteca e o tipo interno do domínio
- [ ] Funções de extensão duplicadas porque não existe um lugar canônico para elas

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Precisa adicionar comportamento a um tipo externo | Introduce Local Extension (wrapper ou funções em pacote próprio) |
| Conversão repetitiva entre tipos externos e internos | Extract Function centralizada no pacote de adaptação |
| Comportamento faltante usado em poucos lugares | Introduce Foreign Method (função autônoma que recebe o tipo externo) |
| Biblioteca inadequada para o domínio | Wrap em interface própria (Anti-Corruption Layer) |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// Workarounds espalhados em vários arquivos para time.Time da stdlib

// arquivo pedido.go
func formatarDataPedido(t time.Time) string {
    return t.Format("02/01/2006")
}

// arquivo fatura.go
func formatarDataFatura(t time.Time) string {
    return t.Format("02/01/2006") // mesma lógica duplicada
}

// arquivo relatorio.go
func formatarDataRelatorio(t time.Time) string {
    return t.In(time.UTC).Format("02/01/2006") // variação não intencional
}
```

**DEPOIS — esperado:**
```go
// pacote datautil — extensão local centralizada para time.Time
package datautil

import "time"

// FormatarBR formata uma data no padrão brasileiro dd/mm/aaaa.
func FormatarBR(t time.Time) string {
    return t.Format("02/01/2006")
}

// InicioDoMes retorna o primeiro dia do mês da data fornecida.
func InicioDoMes(t time.Time) time.Time {
    return time.Date(t.Year(), t.Month(), 1, 0, 0, 0, 0, t.Location())
}

// pedido.go, fatura.go, relatorio.go — todos usam o pacote centralizado
import "meuprojeto/datautil"

datautil.FormatarBR(pedido.CriadoEm)
```

**Por que este padrão:**
- `datautil` é o único lugar para ajustar o formato — uma mudança se propaga para todos os chamadores
- Funções nomeadas comunicam intenção (`FormatarBR`) melhor do que strings de formato espalhadas

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Adicionar métodos ao tipo externo via type alias desnecessário**
```go
// Não aceito — cria um tipo novo que não é compatível com o original sem necessidade
type MinhaData time.Time

func (d MinhaData) FormatarBR() string { ... }
```

**Erro 2: Usar um arquivo `utils.go` global com funções não relacionadas**
```go
// Não aceito — utils.go vira uma gaveta de coisas sem coesão
func FormatarData(t time.Time) string { ... }
func ValidarCPF(cpf string) bool { ... }
func EnviarEmail(dest string) error { ... }
```

---

## 6. Benefícios

- **Centralização:** Adaptações e extensões de bibliotecas vivem em um único pacote
- **Consistência:** Todos os chamadores usam a mesma implementação, eliminando variações acidentais
- **Substituição:** Trocar a biblioteca subjacente requer mudança apenas no pacote de extensão
