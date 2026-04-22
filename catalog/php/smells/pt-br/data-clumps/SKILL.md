# SKILL: Detectando e Refatorando Data Clumps — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/data-clumps

---

## 1. O que é Data Clumps

Partes diferentes do código contêm grupos idênticos de variáveis (um "clump") — os mesmos campos em múltiplas classes, ou os mesmos parâmetros aparecendo juntos em muitas assinaturas de método. Se você removesse um item do grupo e os demais perdessem sentido, você tem um data clump.

**Por que isso acontece:**
- O relacionamento entre os itens de dados nunca foi formalizado como um value object
- Copy-paste propagou o grupo por lugares não relacionados
- O conceito existia informalmente na cabeça dos desenvolvedores, mas não no código

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Três ou mais campos que aparecem juntos em múltiplas classes
- [ ] Os mesmos 2–3 parâmetros consistentemente passados juntos para métodos
- [ ] Variáveis como `$dataInicio`/`$dataFim`, `$latitude`/`$longitude`, `$rua`/`$cidade`/`$cep` como primitivos separados
- [ ] Você precisa atualizar o mesmo grupo de campos em vários lugares para uma única mudança lógica

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                      | Técnica recomendada           |
|----------------------------------------------------------|-------------------------------|
| Clump aparece como campos em uma classe                  | Extract Class                 |
| Clump aparece em listas de parâmetros de métodos         | Introduce Parameter Object    |
| Um método recebe um objeto mas usa apenas parte dele     | Preserve Whole Object         |

**Teste chave:** remova um item do clump. Se os outros perderem significado, o grupo merece sua própria classe.

---

## 4. Exemplo

**ANTES — não aceito:**
```php
class Pedido
{
    private string $rua;
    private string $cidade;
    private string $cep;
    private string $pais;
    private string $nomeCliente;
    private string $emailCliente;
}

class Fatura
{
    private string $rua;     // mesmo clump novamente
    private string $cidade;
    private string $cep;
    private string $pais;
}

public function enviar(string $rua, string $cidade, string $cep, string $pais): void { ... }
```

**DEPOIS — esperado:**
```php
class Endereco
{
    public function __construct(
        public readonly string $rua,
        public readonly string $cidade,
        public readonly string $cep,
        public readonly string $pais,
    ) {}

    public function formatar(): string
    {
        return "{$this->rua}, {$this->cidade} {$this->cep}, {$this->pais}";
    }
}

class Pedido
{
    public function __construct(
        private Endereco $enderecoEntrega,
        private string $nomeCliente,
        private string $emailCliente,
    ) {}
}

class Fatura
{
    public function __construct(private Endereco $enderecoCobranca) {}
}

public function enviar(Endereco $destino): void { ... }
```

**Por que este padrão:**
- `Endereco` é um conceito nomeado — sua validação, formatação e comparação agora vivem em um único lugar
- Toda classe que possui um endereço se beneficia de qualquer melhoria em `Endereco`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Agrupar o clump em um array ou contêiner genérico**
```php
// Não aceito — perde segurança de tipos e intenção
$endereco = ['rua' => 'Rua Principal', 'cidade' => 'São Paulo'];
```

**Erro 2: Criar a classe mas manter os campos separados antigos também**
```php
// Não aceito — agora há duas representações dos mesmos dados
private Endereco $enderecoEntrega;
private string $rua;    // duplicado
private string $cidade; // duplicado
```

**Erro 3: Agrupar dados sem coesão só porque aparecem juntos**
```php
// Não aceito — PreferenciasCliente agrupa campos não relacionados em uma classe
// só porque apareceram juntos na mesma assinatura de método
```

---

## 6. Benefícios

- **Fonte única da verdade:** A lógica do clump (validação, formatação) é centralizada
- **Legibilidade:** `$pedido->getEnderecoEntrega()` é mais expressivo do que quatro campos separados
- **Extensibilidade:** Adicionar um novo campo de endereço requer mudança apenas na classe `Endereco`
