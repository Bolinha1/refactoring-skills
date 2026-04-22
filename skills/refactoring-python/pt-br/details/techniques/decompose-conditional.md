# TÉCNICA: Decompose Conditional — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/decompose-conditional

---

## 1. Problema

Uma expressão condicional complexa (e o código em seus branches) torna difícil entender o que está sendo testado e o que acontece em cada caso.

---

## 2. Solução

Extraia a condição e cada branch para funções bem nomeadas. Os nomes das funções explicam a intenção; os corpos explicam a implementação.

---

## 3. Quando aplicar

- A condição em si é uma expressão booleana complexa que requer reflexão para analisar
- O código no branch verdadeiro ou falso tem várias linhas e merece um nome
- O condicional aparece dentro de uma função já longa (combina com Long Method)
- Ler a condição requer conhecimento de regras de domínio, não apenas lógica de programação

---

## 4. Passos de refatoração

1. Extraia a condição para uma função nomeada conforme a regra de negócio que verifica
2. Extraia o branch verdadeiro para uma função nomeada conforme o que faz
3. Extraia o branch falso para uma função nomeada conforme o que faz
4. Substitua o `if` original por chamadas às funções extraídas
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
def calcular_cobranca(data: date, quantidade: int, preco_unitario: float) -> float:
    if INICIO_VERAO <= data <= FIM_VERAO:
        cobranca = quantidade * preco_unitario * TAXA_VERAO
    else:
        servico = COBRANCA_SERVICO_INVERNO if quantidade > LIMITE_INVERNO else 0
        cobranca = quantidade * preco_unitario * TAXA_INVERNO + servico
    return cobranca
```

**DEPOIS — esperado:**
```python
def calcular_cobranca(d: date, quantidade: int, preco_unitario: float) -> float:
    if _eh_verao(d):
        return _cobranca_verao(quantidade, preco_unitario)
    return _cobranca_inverno(quantidade, preco_unitario)


def _eh_verao(d: date) -> bool:
    return INICIO_VERAO <= d <= FIM_VERAO


def _cobranca_verao(quantidade: int, preco_unitario: float) -> float:
    return quantidade * preco_unitario * TAXA_VERAO


def _cobranca_inverno(quantidade: int, preco_unitario: float) -> float:
    servico = COBRANCA_SERVICO_INVERNO if quantidade > LIMITE_INVERNO else 0
    return quantidade * preco_unitario * TAXA_INVERNO + servico
```

**Por que este padrão:**
- `_eh_verao` nomeia a regra de negócio — leitores entendem sem analisar a lógica de datas
- `_cobranca_verao` e `_cobranca_inverno` expressam intenção de precificação, não implementação

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair a função mas dar um nome técnico**
```python
# Não aceito — o nome descreve o mecanismo, não a regra de negócio
def _verificar_intervalo_data(d: date) -> bool:
    return INICIO_VERAO <= d <= FIM_VERAO
```

**Erro 2: Extrair apenas a condição mas não os branches**
```python
# Não aceito — extração parcial; os branches ainda precisam de nomes
if _eh_verao(d):
    cobranca = quantidade * preco_unitario * TAXA_VERAO  # ainda inline
```

**Erro 3: Decompor uma condição trivial**
```python
# Não aceito — over-engineering para uma verificação simples
def _eh_none(obj) -> bool: return obj is None
if _eh_none(cliente): ...
```

---

## 7. Benefícios

- **Legibilidade:** O `if` lê como uma regra de negócio, não uma fórmula
- **Testabilidade:** Cada branch pode ser testado isoladamente
- **Documentação:** Nomes de funções substituem a necessidade de comentários
