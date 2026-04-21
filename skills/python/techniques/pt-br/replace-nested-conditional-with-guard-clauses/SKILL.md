# TÉCNICA: Replace Nested Conditional with Guard Clauses — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/replace-nested-conditional-with-guard-clauses

---

## 1. Problema

Uma função tem condicionais profundamente aninhados que obscurecem o caminho feliz principal. Leitores precisam rastrear mentalmente cada nível de aninhamento para entender o que a função normalmente faz.

---

## 2. Solução

Substitua condições de casos especiais por retornos antecipados (guard clauses) no topo da função. A lógica principal executa sem indentação no final.

---

## 3. Quando aplicar

- A função tem 2+ níveis de aninhamento que codificam casos extremos ou pré-condições
- O branch else de um `if` de nível superior contém a lógica principal
- Cada condição aninhada verifica uma pré-condição ou caso excepcional antes do trabalho real
- A função termina com um único retorno enterrado dentro de múltiplos blocos else

---

## 4. Passos de refatoração

1. Identifique cada caso especial (verificação de None, condição de erro, saída antecipada)
2. Para cada caso especial, mova sua condição para o topo da função
3. Retorne antecipadamente (ou lance exceção) dentro desse guard clause
4. Remova o aninhamento else agora desnecessário
5. Verifique que a lógica principal executa sem indentação
6. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
def calcular_pagamento(funcionario: Funcionario) -> float:
    if funcionario.esta_morto:
        resultado = valor_morto()
    else:
        if funcionario.esta_demitido:
            resultado = valor_demitido()
        else:
            if funcionario.esta_aposentado:
                resultado = valor_aposentado()
            else:
                resultado = pagamento_normal()
    return resultado
```

**DEPOIS — esperado:**
```python
def calcular_pagamento(funcionario: Funcionario) -> float:
    if funcionario.esta_morto:      return valor_morto()
    if funcionario.esta_demitido:   return valor_demitido()
    if funcionario.esta_aposentado: return valor_aposentado()
    return pagamento_normal()
```

**Por que este padrão:**
- Os casos excepcionais são despachados no topo — leitores os ignoram imediatamente
- `pagamento_normal()` se destaca como o padrão: sem aninhamento, sem else, sem acumulação de variável

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Usar guard clauses para o caso principal, não as exceções**
```python
# Não aceito — guard clauses devem tratar as exceções; fluxo normal deve estar no final
def calcular_pagamento(funcionario: Funcionario) -> float:
    if not funcionario.esta_morto and not funcionario.esta_demitido and not funcionario.esta_aposentado:
        return pagamento_normal()  # invertido — verificar normalidade é o guard
    # casos extremos abaixo...
```

**Erro 2: Adicionar guard clause que duplica uma garantia existente**
```python
# Não aceito — contrato do chamador garante que funcionario nunca é None
if funcionario is None:
    return 0.0  # adiciona ruído sem adicionar segurança
if funcionario.esta_morto:
    return valor_morto()
```

**Erro 3: Achatar condições aninhadas que não são guard clauses independentes**
```python
# Não aceito — essas condições não são guards; representam branches de lógica de negócio
# Use Decompose Conditional em vez de guard clauses aqui
if plano == "verao":
    return cobrar_verao()  # não é uma exceção — é um caminho principal válido
```

---

## 7. Benefícios

- **Legibilidade:** O caminho de execução normal da função é imediatamente visível
- **Aninhamento reduzido:** Cada guard clause remove um nível de indentação
- **Clareza de intenção:** Retornos antecipados sinalizam "este caso é excepcional — pare aqui"
