# TÉCNICA: Replace Method with Method Object — Go

## Fonte
Baseado em: https://refactoring.guru/replace-method-with-method-object

---

## 1. Problema

Um método é tão grande e complexo que extrair sub-métodos fica difícil porque todos
eles compartilhariam muitas variáveis locais. As variáveis locais estão muito
interligadas para serem facilmente passadas como parâmetros.

---

## 2. Solução

Transforme o método em uma struct separada. Cada variável local vira um campo dessa
struct. O corpo do método vira o método principal `Calcular()`, que agora pode ser
livremente dividido em sub-métodos sem precisar passar parâmetros.

---

## 3. Quando aplicar

- O método tem muitas variáveis locais usadas em múltiplas fases lógicas
- Extract Method exigiria passar parâmetros demais entre os novos métodos
- O algoritmo é complexo o suficiente para merecer sua própria abstração
- Você quer testar o algoritmo isoladamente

---

## 4. Passos de refatoração

1. Crie uma nova struct com o nome derivado do método
2. Adicione um campo para o objeto que originalmente continha o método
3. Adicione um campo para cada variável local e parâmetro do método original
4. Crie um construtor (função `New...`) que recebe o objeto fonte e todos os parâmetros
5. Copie o corpo do método original para um método `Calcular()`; referências a variáveis locais viram campos
6. Substitua o corpo do método original por: `return NovaCalculadoraXxx(o, param1).Calcular()`
7. Aplique Extract Method livremente em `Calcular()` — sem parâmetros necessários pois o estado compartilhado está nos campos
8. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func (p *Pedido) Preco() float64 {
	var precoBasePrimario float64
	var precoBaseSecundario float64
	var precoBaseTerciario float64
	// cálculo longo e complexo envolvendo todas as três variáveis
	precoBasePrimario = float64(p.Quantidade) * p.TaxaPrimaria()
	precoBaseSecundario = precoBasePrimario * 0.7
	precoBaseTerciario = math.Max(precoBasePrimario, precoBaseSecundario)
	return precoBaseTerciario - p.desconto(precoBasePrimario, precoBaseSecundario)
}
```

**DEPOIS — esperado:**
```go
func (p *Pedido) Preco() float64 {
	return NovaCalculadoraPreco(p).Calcular()
}

type CalculadoraPreco struct {
	pedido             *Pedido
	precoBasePrimario  float64
	precoBaseSecundario float64
	precoBaseTerciario  float64
}

func NovaCalculadoraPreco(pedido *Pedido) *CalculadoraPreco {
	return &CalculadoraPreco{pedido: pedido}
}

func (c *CalculadoraPreco) Calcular() float64 {
	c.precoBasePrimario = float64(c.pedido.Quantidade) * c.pedido.TaxaPrimaria()
	c.precoBaseSecundario = c.precoBasePrimario * 0.7
	c.precoBaseTerciario = math.Max(c.precoBasePrimario, c.precoBaseSecundario)
	return c.precoBaseTerciario - c.desconto()
}

func (c *CalculadoraPreco) desconto() float64 {
	if c.pedido.EhPremium() {
		return c.precoBasePrimario * 0.1
	}
	return c.precoBaseSecundario * 0.05
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Aplicar quando Extract Method seria suficiente**
```go
// Não aceito — se o método tem apenas 2-3 variáveis locais,
// Extract Method com parâmetros explícitos é mais simples
```

**Erro 2: Nomear a struct de forma vaga**
```go
// Não aceito — Helper, Processador, Calculadora são genéricos demais
type HelperPedido struct { ... }
// Prefira: nomear pelo algoritmo específico
type CalculadoraPrecoPedido struct { ... }
```

---

## 7. Benefícios

- **Habilita Extract Method:** Sub-métodos compartilham estado via campos sem passar parâmetros
- **Testabilidade:** O objeto de cálculo pode ser instanciado e testado independentemente
- **Extensibilidade:** A struct calculadora pode incorporar variações do algoritmo via interfaces
