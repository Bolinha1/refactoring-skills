# TÉCNICA: Introduce Parameter Object — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/introduce-parameter-object

---

## 1. Problema

Vários parâmetros que naturalmente pertencem juntos são sempre passados como grupo para os mesmos métodos. O grupo representa um conceito ainda não formalizado na base de código.

---

## 2. Solução

Substitua o grupo de parâmetros por um único value object (DTO ou classe readonly) que representa o conceito. O objeto se torna um cidadão de primeira classe no modelo de domínio.

---

## 3. Quando aplicar

- Três ou mais parâmetros que sempre aparecem juntos em assinaturas de método
- O mesmo grupo aparece em múltiplos métodos pela base de código
- O grupo representa um conceito de domínio reconhecível (intervalo de datas, endereço, coordenadas)
- Você quer adicionar comportamento (validação, formatação) ao grupo de dados

---

## 4. Passos de refatoração

1. Crie uma nova classe para representar o grupo de parâmetros
2. Adicione os parâmetros como campos readonly
3. Adicione validação de invariantes no construtor
4. Atualize cada método que recebe o grupo de parâmetros para aceitar a nova classe
5. Atualize todos os locais de chamada para construir o novo objeto e passá-lo
6. Procure por comportamento (verificações, cálculos) nos chamadores que poderiam ser movidos para a nova classe
7. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
class ServiçoRelatorio
{
    public function buscarVendasPorPeriodo(\DateTimeImmutable $dataInicio, \DateTimeImmutable $dataFim): array { ... }
    public function somarReceitaPorPeriodo(\DateTimeImmutable $dataInicio, \DateTimeImmutable $dataFim): float { ... }
    public function faturasPorPeriodo(\DateTimeImmutable $dataInicio, \DateTimeImmutable $dataFim): array { ... }
}
```

**DEPOIS — esperado:**
```php
final class Periodo
{
    public function __construct(
        public readonly \DateTimeImmutable $inicio,
        public readonly \DateTimeImmutable $fim,
    ) {
        if ($fim < $inicio) {
            throw new \InvalidArgumentException('fim deve ser após início');
        }
    }

    public function inclui(\DateTimeImmutable $data): bool
    {
        return $data >= $this->inicio && $data <= $this->fim;
    }
}

class ServiçoRelatorio
{
    public function buscarVendasPorPeriodo(Periodo $periodo): array { ... }
    public function somarReceitaPorPeriodo(Periodo $periodo): float { ... }
    public function faturasPorPeriodo(Periodo $periodo): array { ... }
}
```

**Por que este padrão:**
- `Periodo` valida seu próprio invariante uma vez — chamadores não podem passar um período inválido
- O método `inclui()` captura lógica de consulta que antes era duplicada em cada chamador

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Criar o objeto mas não mover comportamento para ele**
```php
// Não aceito — Periodo ainda é apenas um recipiente de duas datas sem comportamento
class Periodo
{
    public \DateTimeImmutable $inicio;
    public \DateTimeImmutable $fim;
}
```

**Erro 2: Tornar o objeto mutável**
```php
// Não aceito — um Periodo mutável pode ser alterado pelos chamadores após passá-lo
class Periodo
{
    public function setInicio(\DateTimeImmutable $inicio): void { $this->inicio = $inicio; }
    public function setFim(\DateTimeImmutable $fim): void { $this->fim = $fim; }
}
```

**Erro 3: Agrupar parâmetros não relacionados só para reduzir a contagem**
```php
// Não aceito — ContextoRequisicao agrupa idCliente, idSessao e locale
// sem razão além de serem três
```

---

## 7. Benefícios

- **Expressividade:** O grupo de parâmetros recebe um nome significativo no domínio
- **Validação:** O objeto valida seus próprios invariantes uma vez, no momento da construção
- **Migração de comportamento:** Lógica que trabalha com o grupo pode ser movida para dentro do objeto
