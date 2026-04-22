# TÉCNICA: Replace Nested Conditional with Guard Clauses — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/replace-nested-conditional-with-guard-clauses

---

## 1. Problema

Um método tem condicionais profundamente aninhados que obscurecem o caminho feliz principal. Leitores precisam rastrear mentalmente cada nível de aninhamento para entender o que o método normalmente faz.

---

## 2. Solução

Substitua condições de casos especiais por retornos antecipados (guard clauses) no topo do método. A lógica principal executa sem indentação no final.

---

## 3. Quando aplicar

- O método tem 2+ níveis de aninhamento que codificam casos extremos ou pré-condições
- O branch else de um `if` de nível superior contém a lógica principal
- Cada condição aninhada verifica uma pré-condição ou caso excepcional antes do trabalho real
- O método termina com um único retorno enterrado dentro de múltiplos blocos else

---

## 4. Passos de refatoração

1. Identifique cada caso especial (verificação de null, condição de erro, saída antecipada)
2. Para cada caso especial, mova sua condição para o topo do método
3. Retorne antecipadamente (ou lance exceção) dentro desse guard clause
4. Remova o aninhamento else agora desnecessário
5. Verifique que a lógica principal executa sem indentação
6. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
public function calcularPagamento(Funcionario $funcionario): float
{
    if ($funcionario->estaMorto()) {
        $resultado = $this->valorMorto();
    } else {
        if ($funcionario->estaDemitido()) {
            $resultado = $this->valorDemitido();
        } else {
            if ($funcionario->estaAposentado()) {
                $resultado = $this->valorAposentado();
            } else {
                $resultado = $this->pagamentoNormal();
            }
        }
    }
    return $resultado;
}
```

**DEPOIS — esperado:**
```php
public function calcularPagamento(Funcionario $funcionario): float
{
    if ($funcionario->estaMorto())      return $this->valorMorto();
    if ($funcionario->estaDemitido())   return $this->valorDemitido();
    if ($funcionario->estaAposentado()) return $this->valorAposentado();
    return $this->pagamentoNormal();
}
```

**Por que este padrão:**
- Os casos excepcionais são despachados no topo — leitores os ignoram imediatamente
- `pagamentoNormal()` se destaca como o padrão: sem aninhamento, sem else, sem acumulação de variável

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Usar guard clauses para o caso principal, não as exceções**
```php
// Não aceito — guard clauses devem tratar as exceções; fluxo normal deve estar no final
public function calcularPagamento(Funcionario $funcionario): float
{
    if (!$funcionario->estaMorto() && !$funcionario->estaDemitido() && !$funcionario->estaAposentado()) {
        return $this->pagamentoNormal(); // invertido — verificar normalidade é o guard
    }
    // casos extremos abaixo...
}
```

**Erro 2: Adicionar guard clause que duplica uma garantia existente**
```php
// Não aceito — type hint já garante que $funcionario nunca é null
if ($funcionario === null) return 0.0; // adiciona ruído, não segurança
if ($funcionario->estaMorto()) return $this->valorMorto();
```

**Erro 3: Achatar condições aninhadas que não são guard clauses independentes**
```php
// Não aceito — essas condições não são guards; representam branches de lógica de negócio
// Use Decompose Conditional em vez de guard clauses aqui
if ($plano === 'verao') {
    return $this->cobrarVerao(); // não é uma exceção — é um caminho principal válido
}
```

---

## 7. Benefícios

- **Legibilidade:** O caminho de execução normal do método é imediatamente visível
- **Aninhamento reduzido:** Cada guard clause remove um nível de indentação
- **Clareza de intenção:** Retornos antecipados sinalizam "este caso é excepcional — pare aqui"
