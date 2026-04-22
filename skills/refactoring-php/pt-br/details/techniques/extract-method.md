# TÉCNICA: Extract Method — PHP

## Fonte
Baseado em: https://refactoring.guru/extract-method

---

## 1. Problema

Você tem um fragmento de código que pode ser agrupado em uma unidade com sentido próprio,
mas ele está embutido em um método maior junto com outras responsabilidades.

---

## 2. Solução

Mova esse fragmento para um novo método `private` com nome que descreva a intenção.
Substitua o trecho original por uma chamada ao novo método.

---

## 3. Quando aplicar

- O trecho de código mereceria um comentário explicativo
- O mesmo bloco de lógica aparece em mais de um lugar
- O método atual faz mais de uma coisa claramente distinta
- Existe dificuldade em dar um nome curto e preciso ao método atual

---

## 4. Passos de refatoração

1. Crie um novo método `private` com nome que expresse a intenção do trecho
2. Copie o fragmento de código para o novo método
3. Identifique variáveis locais usadas pelo trecho:
   - Usadas apenas dentro do trecho → tornam-se variáveis locais do novo método
   - Declaradas antes e lidas dentro → tornam-se parâmetros
   - Modificadas dentro e usadas depois → o novo método deve retorná-las
4. Substitua o trecho original pela chamada ao novo método
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
public function imprimirFatura(Fatura $fatura): void
{
    // imprimir cabeçalho
    echo "***********************\n";
    echo "***   FATURA #{$fatura->getNumero()}   ***\n";
    echo "***********************\n";

    // imprimir itens
    foreach ($fatura->getItens() as $item) {
        echo "{$item->getNome()}\t{$item->getQuantidade()}\t{$item->getPreco()}\n";
    }

    // imprimir total
    $total = array_reduce($fatura->getItens(), function (float $carry, ItemFatura $item) {
        return $carry + $item->getQuantidade() * $item->getPreco();
    }, 0.0);
    echo "TOTAL: R$ {$total}\n";
}
```

**DEPOIS — esperado:**
```php
public function imprimirFatura(Fatura $fatura): void
{
    $this->imprimirCabecalho($fatura);
    $this->imprimirItens($fatura);
    $this->imprimirTotal($fatura);
}

private function imprimirCabecalho(Fatura $fatura): void
{
    echo "***********************\n";
    echo "***   FATURA #{$fatura->getNumero()}   ***\n";
    echo "***********************\n";
}

private function imprimirItens(Fatura $fatura): void
{
    foreach ($fatura->getItens() as $item) {
        echo "{$item->getNome()}\t{$item->getQuantidade()}\t{$item->getPreco()}\n";
    }
}

private function imprimirTotal(Fatura $fatura): void
{
    $total = array_reduce($fatura->getItens(), function (float $carry, ItemFatura $item) {
        return $carry + $item->getQuantidade() * $item->getPreco();
    }, 0.0);
    echo "TOTAL: R$ {$total}\n";
}
```

**Variante — método com retorno:**
```php
// ANTES — variável temporária calculada e usada depois
public function calcularDesconto(Pedido $pedido): float
{
    $subtotal = 0.0;
    foreach ($pedido->getItens() as $item) {
        $subtotal += $item->getPreco() * $item->getQuantidade();
    }
    // ... outras coisas ...
    return $subtotal * $pedido->getTaxaDesconto();
}

// DEPOIS — extração com retorno
public function calcularDesconto(Pedido $pedido): float
{
    $subtotal = $this->calcularSubtotal($pedido);
    return $subtotal * $pedido->getTaxaDesconto();
}

private function calcularSubtotal(Pedido $pedido): float
{
    return array_reduce($pedido->getItens(), function (float $carry, Item $item) {
        return $carry + $item->getPreco() * $item->getQuantidade();
    }, 0.0);
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Nome vago que não expressa intenção**
```php
// Não aceito
private function processarParte1(Fatura $fatura): void { ... }
private function helper(Fatura $fatura): void { ... }
```

**Erro 2: Extrair trecho muito pequeno sem ganho de legibilidade**
```php
// Não aceito — uma linha não precisa virar método
private function imprimirNovaLinha(): void
{
    echo "\n";
}
```

**Erro 3: Deixar o método original ainda grande após extração**
```php
// Não aceito — extraiu um trecho mas o método ainda tem 50 linhas
public function imprimirFatura(Fatura $fatura): void
{
    $this->imprimirCabecalho($fatura);
    // ... 40 linhas restantes sem extração ...
}
```

---

## 7. Benefícios

- **Legibilidade:** O método principal vira uma narrativa de alto nível
- **Reuso:** O método extraído pode ser reutilizado em outros contextos
- **Testabilidade:** Métodos menores são mais fáceis de testar isoladamente
- **Isolamento de erros:** Mudanças afetam apenas o método que as contém
