# TÉCNICA: Dividir Variável Temporária — Go

## Fonte
Baseado em: https://refactoring.guru/split-temporary-variable

---

## 1. Problema

Uma variável temporária é atribuída mais de uma vez dentro do mesmo método, mas cada
atribuição serve a um propósito diferente. Reutilizar o mesmo nome para valores não
relacionados torna o código difícil de seguir e impede que o compilador detecte
incompatibilidades de tipo.

---

## 2. Solução

Crie uma variável separada para cada uso distinto. Nomeie cada variável de acordo com
o valor que ela representa naquele ponto.

---

## 3. Quando aplicar

- Uma variável é atribuída em dois ou mais lugares com significados não relacionados
- Ler a variável no meio do método exige lembrar qual atribuição foi a última
- Variáveis de loop são a exceção — elas são intencionalmente reatribuídas

---

## 4. Passos de refatoração

1. Identifique todos os propósitos distintos que a variável serve
2. Renomeie o primeiro uso para refletir esse propósito específico
3. Declare uma nova variável para o segundo uso com seu próprio nome descritivo
4. Repita para eventuais outros reúsos
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func geometria(altura, largura float64) {
	temp := 2 * (altura + largura)
	fmt.Println("Perímetro:", temp)

	temp = altura * largura   // reutilizada para um valor completamente diferente
	fmt.Println("Área:", temp)
}
```

**DEPOIS — esperado:**
```go
func geometria(altura, largura float64) {
	perimetro := 2 * (altura + largura)
	fmt.Println("Perímetro:", perimetro)

	area := altura * largura
	fmt.Println("Área:", area)
}
```

**Por que esse padrão:**
- `perimetro` e `area` descrevem o que cada valor representa
- Nenhum leitor precisa rastrear qual atribuição está "ativa"
- O compilador pode detectar reuso acidental entre escopos não relacionados

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Renomear sem separar — ainda é uma variável**
```go
// Não aceito — ainda a mesma variável, apenas o comentário mudou
temp := 2 * (altura + largura) // perímetro
fmt.Println(temp)
temp = altura * largura // área — sobrescrevendo a mesma var
```

**Erro 2: Separar variáveis de loop**
```go
// Não aceito — variáveis de loop são intencionalmente reutilizadas
for i := 0; i < n; i++ { ... }
```

---

## 7. Benefícios

- **Clareza:** Cada variável tem um único significado durante toda sua vida
- **Segurança:** O compilador garante que cada variável é usada apenas para seu tipo declarado
- **Facilidade de refatoração:** Variáveis com propósito único são mais simples de extrair ou inlinear depois
