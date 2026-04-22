# TÉCNICA: Split Temporary Variable — Python

## Fonte
Baseado em: https://refactoring.guru/split-temporary-variable

---

## 1. Problema

Uma variável local é atribuída mais de uma vez, servindo a propósitos diferentes em momentos distintos do método. Reutilizar o mesmo nome para conceitos diferentes torna o código difícil de acompanhar.

---

## 2. Solução

Crie uma variável separada para cada atribuição. Dê a cada variável um nome que descreva seu propósito específico.

---

## 3. Quando aplicar

- Uma variável é atribuída em um lugar e depois reatribuída com um significado completamente diferente no mesmo método
- O nome da variável é genérico (ex.: `temp`, `resultado`, `valor`) porque cobre múltiplos conceitos
- O duplo propósito da variável dificulta a leitura ou o teste do método

---

## 4. Passos de refatoração

1. Identifique a variável que é atribuída mais de uma vez para propósitos diferentes
2. Renomeie a primeira atribuição com um nome que descreva o primeiro propósito
3. Renomeie as atribuições subsequentes com novas variáveis cujos nomes descrevam seus propósitos
4. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
def calcular(self, altura: float, largura: float) -> float:
    temp = 2 * (altura + largura)   # perímetro
    print(f"Perímetro: {temp}")
    temp = altura * largura          # área — mesma variável, conceito diferente
    print(f"Área: {temp}")
    return temp
```

**DEPOIS — esperado:**
```python
def calcular(self, altura: float, largura: float) -> float:
    perimetro = 2 * (altura + largura)
    print(f"Perímetro: {perimetro}")
    area = altura * largura
    print(f"Área: {area}")
    return area
```

**Por que esse padrão:**
- `perimetro` e `area` são conceitos distintos que merecem nomes distintos
- Cada variável é atribuída apenas uma vez, tornando o código mais fácil de seguir

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Dividir um acumulador de loop**
```python
# Cuidado — um acumulador de loop é reatribuído intencionalmente; não divida
total = 0
for item in itens:
    total += item.preco  # esta reatribuição é intencional
```

**Erro 2: Manter sufixo genérico ao invés de nomear pelo propósito**
```python
# Não aceito — temp1 e temp2 são tão sem sentido quanto temp
temp1 = 2 * (altura + largura)
temp2 = altura * largura
```

---

## 7. Benefícios

- **Clareza:** Cada nome de variável torna seu propósito único óbvio
- **Depuração mais fácil:** Cada etapa do cálculo tem um valor nomeado distinto
- **Legibilidade:** Os leitores não precisam rastrear mentalmente o que `temp` significa em cada ponto
