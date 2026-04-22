# SKILL: Detectando e Refatorando Feature Envy — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/feature-envy

---

## 1. O que é Feature Envy

Um método acessa os dados de outro objeto mais do que acessa os dados da sua própria classe. O método tem "inveja" da outra classe — parece querer viver lá.

**Por que isso acontece:**
- Campos foram movidos para uma nova classe mas os métodos que os usam não foram
- Lógica de negócio foi colocada em uma classe de serviço ou utilitário que opera nos internos de um objeto de domínio
- Um método foi gradualmente estendido para depender cada vez mais dos getters de outro objeto

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Um método chama vários getters de outro objeto em sequência
- [ ] Um método recebe um objeto como parâmetro e usa principalmente seus campos/métodos
- [ ] Um método ignora completamente o `$this` e só manipula o estado de outro objeto
- [ ] O nome do método naturalmente pertence à outra classe ("calcularTotalPedido" em ServiçoRelatorio)

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                     | Técnica recomendada               |
|---------------------------------------------------------|-----------------------------------|
| O método inteiro pertence à outra classe                | Move Method                       |
| Apenas parte do método tem inveja de outra classe       | Extract Method, depois Move Method |
| O método usa dados de várias classes                    | Mover para a classe com mais dados |
| O método é um cálculo que pertence ao modelo            | Move Method para objeto de domínio |

---

## 4. Exemplo

**ANTES — não aceito:**
```php
class ImpressoraFatura
{
    public function calcularTotalFatura(Fatura $fatura): float
    {
        $subtotal = 0;
        foreach ($fatura->getItens() as $item) {
            $subtotal += $item->getPreco() * $item->getQuantidade();
        }
        $imposto = $subtotal * $fatura->getTaxaImposto();
        $desconto = $fatura->temDesconto() ? $subtotal * $fatura->getTaxaDesconto() : 0;
        return $subtotal + $imposto - $desconto;
    }
}
```

**DEPOIS — esperado:**
```php
class Fatura
{
    public function calcularTotal(): float
    {
        $subtotal = array_sum(
            array_map(fn($i) => $i->getPreco() * $i->getQuantidade(), $this->itens)
        );
        $imposto = $subtotal * $this->taxaImposto;
        $desconto = $this->temDesconto() ? $subtotal * $this->taxaDesconto : 0;
        return $subtotal + $imposto - $desconto;
    }
}

class ImpressoraFatura
{
    public function imprimir(Fatura $fatura): void
    {
        $total = $fatura->calcularTotal(); // delega ao dono
        // ... lógica de impressão
    }
}
```

**Por que este padrão:**
- `calcularTotal` naturalmente pertence à `Fatura` — só usa dados da Fatura
- `ImpressoraFatura` é liberada de conhecer os internos da Fatura

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter o método invejoso como um wrapper fino**
```php
// Não aceito — apenas um wrapper; delete o método invejoso completamente
public function calcularTotalFatura(Fatura $fatura): float
{
    return $fatura->calcularTotal();
}
```

**Erro 2: Mover o método mas manter parâmetros de campo brutos**
```php
// Não aceito — movido mas ainda recebendo valores de campo brutos
public function calcularTotal(float $subtotal, float $taxaImposto, float $taxaDesconto, bool $temDesconto): float { ... }
```

**Erro 3: Mover um método que usa dados de muitas classes igualmente**
```php
// Não aceito — se o método usa Pedido, Cliente e Cupom igualmente,
// movê-lo para qualquer uma das classes apenas desloca a inveja; Extract Class é melhor
```

---

## 6. Benefícios

- **Coesão:** Cada classe possui o comportamento que opera sobre seus próprios dados
- **Encapsulamento:** O objeto de domínio expõe intenção, não campos brutos
- **Manutenibilidade:** As regras de negócio são colocalizadas com os dados que governam
