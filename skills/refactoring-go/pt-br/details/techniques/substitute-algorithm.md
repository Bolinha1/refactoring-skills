# TÉCNICA: Substituir Algoritmo — Go

## Fonte
Baseado em: https://refactoring.guru/substitute-algorithm

---

## 1. Problema

Você quer substituir um algoritmo existente por um mais limpo, simples ou eficiente.
O algoritmo funciona corretamente, mas é difícil de entender, difícil de estender ou
pode agora ser substituído por uma função da biblioteca padrão do Go.

---

## 2. Solução

Substitua o corpo da função ou método que implementa o algoritmo antigo pelo novo
algoritmo. Mantenha a assinatura igual para que os chamadores não sejam afetados.

---

## 3. Quando aplicar

- Existe um algoritmo mais simples que produz os mesmos resultados
- Uma função da stdlib Go (`strings.Contains`, lookup em map, `slices.Contains`) agora
  cobre o que você implementou manualmente
- O algoritmo atual é difícil de estender incrementalmente — é mais fácil reescrever do que modificar
- O algoritmo já foi simplificado ao máximo possível, mas ainda está obscuro

---

## 4. Passos de refatoração

1. Certifique-se de que o algoritmo existente possui testes abrangentes — você precisará deles para verificar a substituição
2. Escreva o novo algoritmo em uma função separada (ou em um branch)
3. Execute os testes com o novo algoritmo; corrija quaisquer falhas
4. Quando os testes passarem, substitua o corpo do algoritmo antigo pelo novo
5. Execute os testes uma última vez
6. Remova qualquer código auxiliar que era necessário apenas pelo algoritmo antigo

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func encontrarPessoa(pessoas []string) string {
	for _, pessoa := range pessoas {
		if pessoa == "Carlos" {
			return "Carlos"
		}
		if pessoa == "Joao" {
			return "Joao"
		}
		if pessoa == "Maria" {
			return "Maria"
		}
	}
	return ""
}
```

**DEPOIS — esperado:**
```go
func encontrarPessoa(pessoas []string) string {
	candidatos := map[string]bool{"Carlos": true, "Joao": true, "Maria": true}
	for _, pessoa := range pessoas {
		if candidatos[pessoa] {
			return pessoa
		}
	}
	return ""
}
```

**Por que esse padrão:**
- Lookup em map é O(1) vs. comparações encadeadas O(k) por elemento
- Adicionar um novo candidato requer apenas uma mudança (adicionar ao map), não um novo bloco `if`
- A intenção — "primeiro elemento do conjunto" — fica imediatamente clara

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Substituir sem uma suíte de testes**
```go
// Não aceito — sem testes, você não pode verificar se o novo algoritmo é equivalente;
// escreva os testes primeiro antes de substituir qualquer coisa
```

**Erro 2: Corrigir o algoritmo antigo incrementalmente ao invés de substituí-lo**
```go
// Não aceito — se o algoritmo é fundamentalmente falho, correções pontuais pioram a situação;
// substitua-o inteiramente
```

---

## 7. Benefícios

- **Simplicidade:** O novo algoritmo é mais fácil de ler e manter
- **Extensibilidade:** Algoritmos bem estruturados são mais fáceis de modificar
- **Idiomático Go:** Usa maps e range loops ao invés de cadeias de if verbosas
