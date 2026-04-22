# TÉCNICA: Inline Temp — PHP

## Fonte
Baseado em: https://refactoring.guru/inline-temp

---

## 1. Problema

Uma variável local armazena o resultado de uma expressão simples, e o nome da variável não acrescenta nenhuma informação que a própria expressão já não comunique claramente.

---

## 2. Solução

Substitua cada referência à variável pela expressão em si. Remova a declaração da variável.

---

## 3. Quando aplicar

- A variável é atribuída exatamente uma vez e a expressão é clara por conta própria
- O nome da variável não agrega significado além de repetir a expressão
- A variável é usada apenas como argumento de outro método ou em um `return`
- Extract Method ou Replace Temp with Query está sendo aplicado e a variável temporária está no caminho

---

## 4. Passos de refatoração

1. Verifique que a variável é atribuída apenas uma vez
2. Encontre todos os usos da variável
3. Substitua cada uso pela expressão do lado direito da atribuição
4. Remova a declaração da variável
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
public function isDescontoElegivel(Order $pedido): bool {
    $elegivel = $pedido->getQuantidadeItens() > 10;
    return $elegivel;
}
```

**DEPOIS — esperado:**
```php
public function isDescontoElegivel(Order $pedido): bool {
    return $pedido->getQuantidadeItens() > 10;
}
```

**Outro exemplo — temp usada como argumento intermediário:**
```php
// ANTES
$precoBase = $pedido->getQuantidade() * $pedido->getPrecoUnitario();
return $precoBase > 1000;

// DEPOIS
return $pedido->getQuantidade() * $pedido->getPrecoUnitario() > 1000;
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Inlining de variável que captura um conceito significativo**
```php
// Não aceito — $isElegivelParaDescontoEmVolume comunica intenção de negócio;
// fazer inline perde o termo do domínio
$isElegivelParaDescontoEmVolume = $pedido->getQuantidadeItens() > 50;
```

**Erro 2: Inlining quando a expressão é custosa e chamada múltiplas vezes**
```php
// Não aceito — fazer inline causa a chamada ao banco a cada referência
$cliente = $this->buscarClientePorId($pedido->getClienteId());
// Usado 3 vezes abaixo — fazer inline acessaria o banco 3 vezes
```

---

## 7. Benefícios

- **Remove ruído:** Elimina variáveis que não agregam valor semântico
- **Prepara outras refatorações:** Limpar temporárias desbloqueia Extract Method e outros movimentos
- **Código mais simples:** Menos para ler, nomear e manter
