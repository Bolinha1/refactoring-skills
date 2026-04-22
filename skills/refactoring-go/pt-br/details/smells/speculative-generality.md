# SKILL: Detectando e Refatorando Speculative Generality — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/speculative-generality

---

## 1. O que é?

Código escrito para lidar com requisitos que ainda não existem — e talvez nunca
existam. Interfaces com uma única implementação, funções com parâmetros que sempre
recebem `nil` ou zero, e structs que existem apenas como "pontos de extensão futuros"
são sinais de que alguém construiu flexibilidade "por via das dúvidas".

---

## 2. Sinais de alerta

- [ ] Uma interface tem apenas uma implementação concreta e nenhum plano para uma segunda
- [ ] Um parâmetro de função é sempre passado como `nil`, zero ou a mesma constante em todos os pontos de chamada
- [ ] Uma struct ou função existe apenas para suportar requisitos futuros mencionados em comentários
- [ ] Um ponto de extensão ou hook nunca foi chamado desde que foi criado
- [ ] Testes são os únicos chamadores de certas funções exportadas

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| Struct existe apenas como ponto de extensão futuro | Inline Class |
| Interface tem apenas uma implementação | Collapse Hierarchy (fundir interface e implementação) |
| Parâmetro é sempre passado com o mesmo valor | Remove Parameter |
| Nomes abstratos escolhidos para variedade que nunca veio | Renomear para algo concreto |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// Interface Notificador existe apenas porque alguém assumiu que outros notificadores viriam
type Notificador interface {
	Enviar(mensagem, canal string) error
}

type NotificadorEmail struct {
	cliente ClienteEmail
}

// canal é sempre "email" em todos os pontos de chamada — generalidade não usada
func (n *NotificadorEmail) Enviar(mensagem, canal string) error {
	return n.cliente.Enviar(mensagem)
}

// Todo ponto de chamada:
notificador.Enviar("Olá", "email")
```

**DEPOIS — esperado:**
```go
type NotificadorEmail struct {
	cliente ClienteEmail
}

func (n *NotificadorEmail) Enviar(mensagem string) error {
	return n.cliente.Enviar(mensagem)
}

notificador.Enviar("Olá")
```

**Por que esse padrão:**
- A interface e o parâmetro `canal` não utilizado eram complexidade especulativa
- Se um segundo tipo de notificador for necessário no futuro, a interface pode ser
  introduzida então, guiada por requisitos reais

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Remover interfaces usadas em testes**
```go
// Cuidado — interfaces usadas para injetar test doubles têm propósito real;
// verifique se não há mocks de teste antes de remover
```

**Erro 2: Remover interface porque hoje tem apenas uma implementação**
```go
// Cuidado — se uma segunda implementação já está no roadmap,
// a abstração não é especulativa; mantenha-a
```

---

## 6. Benefícios

- **Complexidade reduzida:** O código só contém o que serve às necessidades atuais
- **Onboarding mais fácil:** Novos desenvolvedores não precisam entender hooks que não fazem nada
- **Refatoração mais simples:** Quando requisitos reais chegarem, as abstrações serão informadas por casos de uso reais
