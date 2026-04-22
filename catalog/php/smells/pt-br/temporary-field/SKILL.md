# SMELL: Temporary Field — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/temporary-field

---

## 1. O que é?

Uma classe contém um campo que só é definido e usado em certas situações — durante um algoritmo ou chamada de método. No restante do tempo ele é `null`, vazio ou sem significado. Outros códigos que leem o campo não conseguem saber quando ele tem um valor válido e quando não tem.

---

## 2. Sinais de alerta

- [ ] Um campo é `null` ou não inicializado na maior parte do tempo
- [ ] Um campo é definido em um método e consumido em outro, mas nunca mantido durante o tempo de vida do objeto
- [ ] Um campo existe apenas porque um algoritmo longo precisava de um lugar para colocar estado intermediário
- [ ] A classe tem um padrão "prepare então execute" onde campos são preenchidos logo antes do uso
- [ ] Verificações de null para campos de instância aparecem em toda a classe

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Extract Class** | Mova os campos temporários (e os métodos que os usam) para uma classe dedicada |
| **Introduce Null Object** | Substitua o estado null/vazio por um Null Object adequado para que chamadores não precisem verificar |

---

## 4. Exemplo

**ANTES — não aceito:**
```php
// CalculadoraRota mantém campos que só são válidos durante calcularRota()
class CalculadoraRota {
    private ?array $paradas = null;           // só definido durante o cálculo
    private int $distanciaTotal = 0;          // só válido após calcular()
    private ?string $caminhoMaisRapido = null; // null a menos que calcular() tenha executado

    public function setParadas(array $paradas): void {
        $this->paradas = $paradas;
    }

    public function calcular(): void {
        $this->distanciaTotal = $this->calcularDistancia($this->paradas);
        $this->caminhoMaisRapido = $this->encontrarCaminhoMaisRapido($this->paradas);
    }

    public function getCaminhoMaisRapido(): ?string {
        return $this->caminhoMaisRapido; // null se calcular() não foi chamado primeiro
    }
}
```

**DEPOIS — esperado:**
```php
// Extraia o estado temporário para um objeto de valor dedicado
class ResultadoRota {
    public function __construct(
        private readonly int $distanciaTotal,
        private readonly string $caminhoMaisRapido
    ) {}

    public function getDistanciaTotal(): int { return $this->distanciaTotal; }
    public function getCaminhoMaisRapido(): string { return $this->caminhoMaisRapido; }
}

class CalculadoraRota {
    public function calcular(array $paradas): ResultadoRota {
        $distancia = $this->calcularDistancia($paradas);
        $caminho = $this->encontrarCaminhoMaisRapido($paradas);
        return new ResultadoRota($distancia, $caminho);
    }
}
```

**Por que este padrão:**
- `ResultadoRota` só existe quando contém dados válidos — não há estado null
- `CalculadoraRota` é sem estado: fácil de reutilizar e testar em paralelo

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Adicionar uma flag `pronto` em vez de corrigir o design**
```php
// Não aceito — disciplina de flag é frágil; Extract Class remove a necessidade dela
private bool $calculado = false;
public function getCaminhoMaisRapido(): string {
    if (!$this->calculado) {
        throw new \RuntimeException('Chame calcular() primeiro');
    }
    return $this->caminhoMaisRapido;
}
```

**Erro 2: Mover o campo temporário para uma subclasse**
```php
// Não aceito — o campo ainda viaja com o objeto; o smell permanece
class CalculadoraRotaCalculando extends CalculadoraRota {
    private ?string $caminhoMaisRapido = null; // mesmo problema, apenas renomeado
}
```

---

## 6. Benefícios

- **Correção:** Sem mais estado null se passando por valor válido
- **Clareza:** Os campos da classe são sempre significativos independente de quando você os observa
- **Testabilidade:** A classe extraída pode ser testada com dados válidos e totalmente inicializados
