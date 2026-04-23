# SKILL: Detectando e Refatorando Classes Alternativas com Interfaces Diferentes — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/alternative-classes-with-different-interfaces

---

## 1. O que é Classes Alternativas com Interfaces Diferentes

Duas ou mais structs (ou interfaces) fazem a mesma coisa mas possuem nomes de métodos ou assinaturas diferentes. Chamadores precisam tratar cada uma de forma especial mesmo que o comportamento seja equivalente, criando código duplicado e acoplamento desnecessário.

**Por que isso acontece:**
- Duas partes do sistema resolveram o mesmo problema de forma independente
- Um struct foi criado sem verificar se já existia algo equivalente
- Código foi migrado de fontes diferentes sem harmonização das interfaces

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Dois structs têm métodos que fazem a mesma coisa com nomes diferentes
- [ ] Código cliente usa `if/switch` para escolher entre dois tipos que deveriam ser intercambiáveis
- [ ] Duas interfaces definem o mesmo contrato com vocabulário diferente
- [ ] Você duplicou lógica porque os dois tipos não compartilham uma interface comum

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica recomendada |
|---|---|
| Métodos fazem o mesmo mas têm nomes diferentes | Rename Method para alinhar assinaturas |
| Structs compartilham comportamento mas não interface | Extract Interface comum |
| Um dos structs tem funcionalidade extra útil | Move Method para consolidar |
| Structs são quase idênticos | Unir em um único struct com interface clara |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
// Dois "serviços de notificação" com interfaces incompatíveis
type NotificadorEmail struct{}

func (n *NotificadorEmail) EnviarEmail(destinatario, mensagem string) error {
    // envia email
    return nil
}

type AlertaSMS struct{}

func (a *AlertaSMS) DispararAlerta(numero, texto string) error {
    // envia SMS
    return nil
}

// Chamador precisa tratar cada um separadamente
func notificar(tipo string, email *NotificadorEmail, sms *AlertaSMS, msg string) {
    if tipo == "email" {
        email.EnviarEmail("usuario@exemplo.com", msg)
    } else {
        sms.DispararAlerta("+5511999999999", msg)
    }
}
```

**DEPOIS — esperado:**
```go
// Interface comum extraída
type Notificador interface {
    Notificar(destinatario, mensagem string) error
}

type NotificadorEmail struct{}

func (n *NotificadorEmail) Notificar(destinatario, mensagem string) error {
    // envia email
    return nil
}

type NotificadorSMS struct{}

func (n *NotificadorSMS) Notificar(destinatario, mensagem string) error {
    // envia SMS
    return nil
}

// Chamador usa a interface — sem switch, sem acoplamento
func notificar(n Notificador, destinatario, msg string) {
    n.Notificar(destinatario, msg)
}
```

**Por que este padrão:**
- A interface `Notificador` é o contrato único — adicionar um novo canal não muda `notificar`
- Cada implementação é testável de forma isolada com um mock simples

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Criar um wrapper que converte uma interface na outra**
```go
// Não aceito — adaptador esconde o smell em vez de corrigir o design
func wrapSMS(a *AlertaSMS) Notificador {
    return &adaptadorSMS{a}
}
```

**Erro 2: Usar interface vazia para unificar os tipos**
```go
// Não aceito — perde segurança de tipos; o switch volta via type assertion
func notificar(n interface{}, msg string) { ... }
```

---

## 6. Benefícios

- **Polimorfismo real:** Código cliente depende de abstrações, não de implementações concretas
- **Extensibilidade:** Novos canais de notificação são adicionados implementando a interface, sem tocar no código existente
- **Testabilidade:** Mocks e stubs são triviais de criar contra uma interface bem definida
