# TÉCNICA: Replace Temp with Query — Go

## Fonte
Baseado em: https://refactoring.guru/replace-temp-with-query

---

## 1. Problema

Você usa uma variável temporária para armazenar o resultado de uma expressão e depois
referencia essa variável no mesmo método. A expressão é recalculada toda vez que o
método é executado, mas nunca é exposta como lógica reutilizável.

---

## 2. Solução

Extraia a expressão para um método (ou função auxiliar privada). Substitua todas as
referências à variável temporária por chamadas a esse método.

---

## 3. Quando aplicar

- A mesma expressão aparece em múltiplos métodos e você quer evitar duplicação
- A variável temporária obscurece o que o valor representa
- O método é longo e extrair sub-cálculos melhora a legibilidade
- A expressão tem um significado nomeável no domínio

---

## 4. Passos de refatoração

1. Certifique-se de que a expressão é livre de efeitos colaterais (cálculo puro)
2. Extraia a expressão para um novo método com nome descritivo
3. Substitua cada referência à variável temporária por uma chamada ao novo método
4. Remova a declaração da variável temporária
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func (p *Pedido) PrecoComDesconto() float64 {
	precoBase := float64(p.Quantidade) * p.PrecoItem
	var fatorDesconto float64
	if precoBase > 1000 {
		fatorDesconto = 0.95
	} else {
		fatorDesconto = 0.98
	}
	return precoBase * fatorDesconto
}
```

**DEPOIS — esperado:**
```go
func (p *Pedido) PrecoComDesconto() float64 {
	return p.precoBase() * p.fatorDesconto()
}

func (p *Pedido) precoBase() float64 {
	return float64(p.Quantidade) * p.PrecoItem
}

func (p *Pedido) fatorDesconto() float64 {
	if p.precoBase() > 1000 {
		return 0.95
	}
	return 0.98
}
```

**Por que esse padrão:**
- `precoBase()` e `fatorDesconto()` podem ser reutilizados em outros métodos de `Pedido`
- O método principal lê como uma fórmula, não uma sequência imperativa
- Cada auxiliar pode ser testado independentemente

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair método com efeitos colaterais**
```go
// Não aceito — chamar duas vezes produz resultados diferentes
func (p *Pedido) proximoID() int {
	p.contador++
	return p.contador
}
```

**Erro 2: Extrair expressão de uma linha que não adiciona clareza**
```go
// Não aceito — a variável já tinha nome claro; envolve-la não acrescenta nada
func (p *Pedido) qtd() int { return p.Quantidade }
```

---

## 7. Benefícios

- **Reutilizabilidade:** O método extraído fica disponível para toda a struct, não apenas um método
- **Legibilidade:** O método chamador descreve o que calcula, não como
- **Testabilidade:** Métodos pequenos e puros são fáceis de testar isoladamente
