# TÉCNICA: Internalizar Variável Temporária — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/inline-temp

---

## 1. Problema

Uma variável temporária armazena o resultado de uma expressão simples e é usada apenas uma vez. A variável não agrega legibilidade — o nome dela não é mais claro do que a própria expressão, e ela ocupa espaço desnecessário no código.

---

## 2. Solução

Substitua todas as referências à variável temporária pela expressão que ela armazena. Remova a declaração da variável.

---

## 3. Quando aplicar

- A variável é atribuída apenas uma vez e usada apenas uma vez
- O nome da variável não é mais informativo que a expressão em si
- A variável foi criada como passo intermediário antes de aplicar Replace Temp with Query
- A expressão é curta e clara o suficiente para ser lida inline

---

## 4. Passos de refatoração

1. Verifique que a variável é atribuída apenas uma vez
2. Verifique que a expressão não tem efeitos colaterais
3. Substitua cada uso da variável pela expressão original
4. Remova a declaração da variável
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```go
func eClienteEspecial(pedido *Pedido) bool {
    precoBase := pedido.Quantidade * pedido.PrecoUnitario
    return precoBase > 1000
}
```

**DEPOIS — esperado:**
```go
func eClienteEspecial(pedido *Pedido) bool {
    return pedido.Quantidade*pedido.PrecoUnitario > 1000
}
```

**Por que esse padrão:**
- `precoBase` era usada apenas uma vez e a expressão é clara por si só
- Remover a variável torna a função mais concisa sem perder legibilidade

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Internalizar uma variável cujo nome agrega significado**
```go
// Não aceito — precoComDesconto comunica intenção que a expressão pura não comunicaria
precoComDesconto := precoBase * (1 - taxaDesconto)
return precoComDesconto
```

**Erro 2: Internalizar uma expressão com efeito colateral**
```go
// Não aceito — chamar buscarPedido() inline pode ter efeitos colaterais
pedido := buscarPedido(id) // consulta ao banco — não deve ser internalizada
return pedido.Total > 0
```

**Erro 3: Internalizar uma variável usada mais de uma vez**
```go
// Não aceito — internalizar duplica a expressão e pode causar inconsistência
total := calcularTotal(itens)
fmt.Println(total)
return total > limiteAprovacao
```

---

## 7. Benefícios

- **Concisão:** Remove variáveis temporárias que não agregam clareza
- **Foco:** O leitor não precisa rastrear uma variável que existe por apenas uma linha
- **Preparação:** Limpa o código antes de aplicar outras refatorações
