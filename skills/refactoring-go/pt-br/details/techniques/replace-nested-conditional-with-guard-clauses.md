# TÉCNICA: Substituir Condicional Aninhado por Guard Clauses — Go

## Fonte
Baseado em: https://refactoring.guru/replace-nested-conditional-with-guard-clauses

---

## 1. Problema

Um método tem condicionais aninhadas que tornam o fluxo normal difícil de identificar.
O código "principal" fica enterrado dentro de vários níveis de indentação, obscurecendo
a intenção.

---

## 2. Solução

Separe todos os casos especiais em verificações separadas de retorno antecipado
(guard clauses). O caminho feliz fica no final do método, sem indentação extra.
Retornos antecipados são idiomáticos em Go e amplamente preferidos.

---

## 3. Quando aplicar

- O método tem múltiplos níveis de condicionais aninhados
- O caso "normal" fica enterrado dentro dos condicionais
- As verificações de pré-condições ou casos de erro dominam a leitura do método
- A função de erro de Go (`if err != nil`) cria pirâmides de indentação

---

## 4. Passos de refatoração

1. Identifique os casos especiais ou condições de erro que podem retornar cedo
2. Para cada caso especial, crie uma verificação no início do método com retorno antecipado
3. Mova o código do caminho normal para fora dos condicionais
4. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func calcularPagamento(pedido *Pedido) (float64, error) {
	if pedido != nil {
		if pedido.EstaAtivo() {
			if pedido.Total > 0 {
				if pedido.Cliente != nil {
					desconto := 0.0
					if pedido.Cliente.EhPremium() {
						desconto = 0.1
					}
					return pedido.Total * (1 - desconto), nil
				}
			}
		}
	}
	return 0, fmt.Errorf("pedido inválido")
}
```

**DEPOIS — esperado:**
```go
func calcularPagamento(pedido *Pedido) (float64, error) {
	if pedido == nil {
		return 0, fmt.Errorf("pedido não pode ser nulo")
	}
	if !pedido.EstaAtivo() {
		return 0, fmt.Errorf("pedido inativo")
	}
	if pedido.Total <= 0 {
		return 0, fmt.Errorf("total do pedido deve ser positivo")
	}
	if pedido.Cliente == nil {
		return 0, fmt.Errorf("pedido sem cliente")
	}

	desconto := 0.0
	if pedido.Cliente.EhPremium() {
		desconto = 0.1
	}
	return pedido.Total * (1 - desconto), nil
}
```

**Por que esse padrão:**
- O caminho feliz fica no final, sem indentação extra
- Cada guard clause tem uma mensagem de erro específica
- É o padrão idiomático Go para tratamento de erros

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Usar flag booleana em vez de retorno antecipado**
```go
// Não aceito — rastrear a flag é mais difícil que leitura linear com retornos
valido := true
if pedido == nil { valido = false }
if valido && !pedido.EstaAtivo() { valido = false }
if valido { ... }
```

**Erro 2: Inverter condições desnecessariamente**
```go
// Não aceito — condições duplas negativas são difíceis de ler
if !(pedido == nil || !pedido.EstaAtivo()) {
    // código principal
}
```

---

## 7. Benefícios

- **Legibilidade:** O caminho feliz fica no final, sem indentação extra
- **Erros explícitos:** Cada condição de falha tem sua própria mensagem de erro
- **Idiomático Go:** Segue o padrão de tratamento de erros da linguagem
- **Testabilidade:** Cada condição de guard pode ser testada independentemente
