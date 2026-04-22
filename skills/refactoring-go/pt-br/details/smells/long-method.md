# SKILL: Detectando e Refatorando Long Method — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/long-method

---

## 1. O que é Long Method

Uma função ou método que contém linhas de código em excesso.
Como regra geral: qualquer função com mais de 10 linhas já merece atenção.

**Por que isso acontece:**
- Sempre se adiciona lógica à função, nunca se remove
- É mentalmente mais fácil adicionar duas linhas do que criar uma nova função
- O problema cresce silenciosamente até virar código espaguete

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Função com mais de 10 linhas
- [ ] Bloco de código que você sentiu vontade de comentar
- [ ] Condicional complexa (if/else aninhado)
- [ ] Loop com lógica não trivial dentro
- [ ] Variáveis temporárias espalhadas pela função
- [ ] Função que faz mais de uma coisa claramente distinta

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| Trecho de código comentado ou comentável | Extract Method |
| Variáveis locais impedem a extração | Replace Temp with Query |
| Muitos parâmetros na função extraída | Introduce Parameter Object |
| Condicional complexa | Decompose Conditional |
| Nenhuma das anteriores resolve | Replace Method with Method Object |

**Regra de ouro:** Se você sentiu vontade de escrever um comentário dentro da função,
esse trecho deve virar uma função com nome descritivo.

---

## 4. Exemplo

**ANTES — não aceito:**
```go
func processarPedido(pedido *Pedido, repo PedidoRepo, email EmailService) error {
	// validar pedido
	if len(pedido.Itens) == 0 {
		return fmt.Errorf("pedido sem itens")
	}
	if pedido.Cliente == nil {
		return fmt.Errorf("cliente não informado")
	}

	// calcular total
	total := 0.0
	for _, item := range pedido.Itens {
		total += item.Preco * float64(item.Quantidade)
	}
	if pedido.TemDesconto {
		total = total * (1 - pedido.Desconto)
	}
	pedido.Total = total

	// persistir
	if err := repo.Salvar(pedido); err != nil {
		return err
	}
	return email.EnviarConfirmacao(pedido.Cliente.Email, pedido)
}
```

**DEPOIS — esperado:**
```go
func processarPedido(pedido *Pedido, repo PedidoRepo, email EmailService) error {
	if err := validarPedido(pedido); err != nil {
		return err
	}
	pedido.Total = calcularTotal(pedido)
	return confirmarPedido(pedido, repo, email)
}

func validarPedido(pedido *Pedido) error {
	if len(pedido.Itens) == 0 {
		return fmt.Errorf("pedido sem itens")
	}
	if pedido.Cliente == nil {
		return fmt.Errorf("cliente não informado")
	}
	return nil
}

func calcularTotal(pedido *Pedido) float64 {
	total := 0.0
	for _, item := range pedido.Itens {
		total += item.Preco * float64(item.Quantidade)
	}
	if pedido.TemDesconto {
		return total * (1 - pedido.Desconto)
	}
	return total
}

func confirmarPedido(pedido *Pedido, repo PedidoRepo, email EmailService) error {
	if err := repo.Salvar(pedido); err != nil {
		return err
	}
	return email.EnviarConfirmacao(pedido.Cliente.Email, pedido)
}
```

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair sem nomear com intenção**
```go
// Não aceito — nomes não expressam intenção de negócio
func fazAlgumaCoisa(p *Pedido) error { ... }
func parte2(p *Pedido) float64 { ... }
```

**Erro 2: Comentar em vez de extrair**
```go
// Não aceito — comentário indica que devia ser uma função
// valida os dados do cliente
if pedido.Cliente == nil || pedido.Cliente.Email == "" { ... }
```

---

## 6. Benefícios

- **Manutenção:** Funções curtas são mais fáceis de entender e modificar
- **Testabilidade:** Funções pequenas são mais fáceis de testar isoladamente
- **Reutilização:** Funções extraídas podem ser usadas em outros contextos
- **Legibilidade:** A função principal vira uma narrativa de alto nível
