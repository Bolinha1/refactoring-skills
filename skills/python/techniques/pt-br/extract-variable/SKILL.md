# TECHNIQUE: Extract Variable — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/extract-variable

---

## 1. Problema

Uma expressão complexa é difícil de entender de relance. Resultados intermediários são calculados inline, tornando difícil ver o que cada parte significa ou depurar o cálculo.

---

## 2. Solução

Atribua a expressão (ou uma sub-expressão) a uma variável temporária com nome claro. Use o nome da variável para transmitir o significado do resultado.

---

## 3. Quando aplicar

- Uma expressão condicional tem múltiplas partes que cada uma merece um nome
- Uma longa expressão aritmética ou de string oculta o que está calculando
- A mesma sub-expressão aparece mais de uma vez no mesmo escopo
- Um ponto de interrupção de depuração é necessário em um resultado intermediário

---

## 4. Passos de refatoração

1. Identifique a expressão ou sub-expressão a nomear
2. Atribua a expressão a uma variável local descritiva
3. Substitua a expressão original (e todas as duplicatas no mesmo escopo) pela variável
4. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
def preco(pedido) -> float:
    return (pedido.quantidade * pedido.preco_item
            - max(0, pedido.quantidade - 500) * pedido.preco_item * 0.05
            + min(pedido.quantidade * pedido.preco_item * 0.1, 100.0))
```

**DEPOIS — esperado:**
```python
def preco(pedido) -> float:
    preco_base = pedido.quantidade * pedido.preco_item
    desconto_quantidade = max(0, pedido.quantidade - 500) * pedido.preco_item * 0.05
    frete = min(preco_base * 0.1, 100.0)
    return preco_base - desconto_quantidade + frete
```

**Por que este padrão:**
- Cada nome de variável explica a intenção de sua sub-expressão
- `preco_base` é calculado uma vez e reutilizado, removendo duplicação

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Nomear uma variável pelo tipo ou posição em vez do significado**
```python
# Não aceito — resultado, temp, valor não adicionam informação
resultado = pedido.quantidade * pedido.preco_item
```

**Erro 2: Extrair excessivamente cada sub-expressão individual**
```python
# Não aceito — muito granular; operações simples raramente precisam de variável
qtd = pedido.quantidade
preco = pedido.preco_item
produto = qtd * preco
```

**Erro 3: Mutar a variável após a extração**
```python
# Não aceito — uma variável extraída para clareza não deve ser reatribuída
preco_base = pedido.quantidade * pedido.preco_item
preco_base = preco_base * 1.1  # confuso — deve ser uma nova variável
```

---

## 7. Benefícios

- **Legibilidade:** Nomes tornam expressões complexas autodocumentadas
- **Depurabilidade:** Você pode definir um ponto de interrupção ou imprimir cada etapa nomeada
- **Duplicação reduzida:** Uma sub-expressão usada mais de uma vez é calculada uma vez e nomeada
