# SKILL: Detectando e Refatorando Shotgun Surgery — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/shotgun-surgery

---

## 1. O que é Shotgun Surgery

Uma única mudança lógica requer modificar muitas classes diferentes ao mesmo tempo. Fazer uma mudança conceitual "dispara" edições pela base de código como um tiro de espingarda — tocando muitos arquivos para o que deveria ser uma correção localizada.

O inverso do Divergent Change: muitas classes, uma razão para mudar.

**Por que isso acontece:**
- Uma responsabilidade foi fragmentada em muitas classes em vez de viver em um único lugar
- Lógica que deveria ser centralizada foi duplicada ou distribuída por "flexibilidade"
- Preocupações transversais (logging, auditoria, notificações) foram tratadas inline em todo lugar

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Adicionar uma nova funcionalidade força você a editar 5+ classes não relacionadas
- [ ] Renomear um conceito requer tocar dezenas de arquivos
- [ ] Uma única regra de negócio é aplicada em múltiplos lugares
- [ ] Você sempre precisa fazer a mesma mudança em vários lugares simultaneamente
- [ ] Falhas de teste em cascata por muitas classes de teste para uma única mudança conceitual

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                            | Técnica recomendada     |
|----------------------------------------------------------------|-------------------------|
| Comportamento espalhado todo relacionado a um conceito         | Move Method + Move Field |
| Múltiplas classes pequenas contribuindo para um conceito       | Inline Class            |
| Preocupação transversal repetida em todo lugar                 | Extract Class           |
| Regra de negócio duplicada aplicada em muitos lugares          | Move Method para um dono |

---

## 4. Exemplo

**ANTES — não aceito:**
```php
// Adicionar o conceito de "moeda" requer mudar TODAS essas classes:

class Produto
{
    private float $preco; // deve adicionar campo moeda aqui
}

class Pedido
{
    public function getTotal(): float { ... } // deve formatar com moeda
}

class Fatura
{
    public function formatarValor(float $valor): string
    {
        return 'R$' . number_format($valor, 2); // deve usar moeda
    }
}

class Recibo
{
    public function imprimirTotal(float $valor): void
    {
        echo $valor; // deve usar moeda
    }
}
```

**DEPOIS — esperado:**
```php
final class Dinheiro
{
    public function __construct(
        private readonly float $valor,
        private readonly string $moeda,
    ) {}

    public function somar(Dinheiro $outro): Dinheiro
    {
        if ($this->moeda !== $outro->moeda) {
            throw new \InvalidArgumentException('Moedas diferentes');
        }
        return new Dinheiro($this->valor + $outro->valor, $this->moeda);
    }

    public function formatar(): string
    {
        $simbolos = ['BRL' => 'R$', 'USD' => '$', 'EUR' => '€'];
        $simbolo = $simbolos[$this->moeda] ?? $this->moeda;
        return $simbolo . number_format($this->valor, 2);
    }
}

// Agora Produto, Pedido, Fatura, Recibo usam Dinheiro — um conceito, um lugar
class Produto
{
    public function __construct(private Dinheiro $preco) {}
}
```

**Por que este padrão:**
- `Dinheiro` centraliza todo comportamento de moeda — adicionar um novo formato de moeda toca uma classe
- Todos os chamadores se beneficiam automaticamente

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Centralizar os dados mas não o comportamento**
```php
// Não aceito — DinheiroDto agrupa os campos mas formatação/aritmética ficam espalhadas
class DinheiroDto
{
    public float $valor;
    public string $codigoMoeda;
}
```

**Erro 2: Usar uma classe utilitária estática como ponto único**
```php
// Não aceito — DinheiroHelper é um smell escondido; lógica ainda fragmentada entre chamadores
class DinheiroHelper
{
    public static function formatar(float $valor, string $moeda): string { ... }
}
```

**Erro 3: Fazer inline de classes agressivamente, criando uma nova god class**
```php
// Não aceito — fazer inline de todas as classes pequenas em uma megaclasse troca
// Shotgun Surgery por Divergent Change
```

---

## 6. Benefícios

- **Localidade:** Um conceito de negócio = uma classe para mudar
- **Segurança:** Mudanças são mais fáceis de raciocinar quando o blast radius é pequeno
- **Descobribilidade:** Todo comportamento de um conceito está em um lugar óbvio
