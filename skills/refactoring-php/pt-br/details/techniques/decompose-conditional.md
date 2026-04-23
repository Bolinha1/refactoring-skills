# TÉCNICA: Decompose Conditional — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/decompose-conditional

---

## 1. Problema

Uma expressão condicional complexa (e o código em seus branches) torna difícil entender o que está sendo testado e o que acontece em cada caso.

---

## 2. Solução

Extraia a condição e cada branch para métodos bem nomeados. Os nomes dos métodos explicam a intenção; os corpos explicam a implementação.

---

## 3. Quando aplicar

- A condição em si é uma expressão booleana complexa que requer reflexão para analisar
- O código no branch verdadeiro ou falso tem várias linhas e merece um nome
- O condicional aparece dentro de um método já longo (combina com Long Method)
- Ler a condição requer conhecimento de regras de domínio, não apenas lógica de programação

---

## 4. Passos de refatoração

1. Extraia a condição para um método nomeado conforme a regra de negócio que verifica
2. Extraia o branch verdadeiro para um método nomeado conforme o que faz
3. Extraia o branch falso para um método nomeado conforme o que faz
4. Substitua o `if` original por chamadas aos métodos extraídos
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
public function calcularCobranca(DateTimeImmutable $data, int $quantidade, float $precoUnitario): float
{
    if ($data >= INICIO_VERAO && $data <= FIM_VERAO) {
        $cobranca = $quantidade * $precoUnitario * TAXA_VERAO;
    } else {
        $servicoExtra = $quantidade > LIMITE_INVERNO ? COBRANCA_SERVICO_INVERNO : 0;
        $cobranca = $quantidade * $precoUnitario * TAXA_INVERNO + $servicoExtra;
    }
    return $cobranca;
}
```

**DEPOIS — esperado:**
```php
public function calcularCobranca(DateTimeImmutable $data, int $quantidade, float $precoUnitario): float
{
    if ($this->ehVerao($data)) {
        return $this->cobrancaVerao($quantidade, $precoUnitario);
    }
    return $this->cobrancaInverno($quantidade, $precoUnitario);
}

private function ehVerao(DateTimeImmutable $data): bool
{
    return $data >= INICIO_VERAO && $data <= FIM_VERAO;
}

private function cobrancaVerao(int $quantidade, float $precoUnitario): float
{
    return $quantidade * $precoUnitario * TAXA_VERAO;
}

private function cobrancaInverno(int $quantidade, float $precoUnitario): float
{
    $servicoExtra = $quantidade > LIMITE_INVERNO ? COBRANCA_SERVICO_INVERNO : 0;
    return $quantidade * $precoUnitario * TAXA_INVERNO + $servicoExtra;
}
```

**Por que este padrão:**
- `ehVerao` nomeia a regra de negócio — leitores entendem sem analisar a lógica de datas
- `cobrancaVerao` e `cobrancaInverno` expressam intenção de precificação, não implementação

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair o método mas dar um nome técnico**
```php
// Não aceito — o nome descreve o mecanismo, não a regra de negócio
private function verificarIntervaloData(DateTimeImmutable $data): bool
{
    return $data >= INICIO_VERAO && $data <= FIM_VERAO;
}
```

**Erro 2: Extrair apenas a condição mas não os branches**
```php
// Não aceito — extração parcial; os branches ainda precisam de nomes
if ($this->ehVerao($data)) {
    $cobranca = $quantidade * $precoUnitario * TAXA_VERAO; // ainda inline
}
```

**Erro 3: Decompor uma condição trivial**
```php
// Não aceito — over-engineering para uma verificação simples
private function ehNulo(mixed $obj): bool { return $obj === null; }
if ($this->ehNulo($cliente)) { ... }
```

---

## 7. Benefícios

- **Legibilidade:** A declaração `if` lê como uma regra de negócio, não uma fórmula
- **Testabilidade:** Cada branch pode ser testado isoladamente
- **Documentação:** Nomes de métodos substituem a necessidade de comentários
