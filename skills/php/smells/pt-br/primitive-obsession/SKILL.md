# SKILL: Detecção e Refatoração de Primitive Obsession — PHP

## Fonte
Baseado em: https://refactoring.guru/smells/primitive-obsession

---

## 1. O que é Primitive Obsession

Uso excessivo de tipos básicos (`string`, `int`, `float`, `bool`, `array`)
para representar conceitos de domínio que mereceriam classes ou value objects próprios.

**Por que isso acontece:**
- Criar uma nova classe parece exagero para algo "simples"
- Arrays associativos são convenientes e rápidos de usar
- O código cresce e as chaves do array viram strings mágicas difíceis de rastrear

---

## 2. Sinais de alerta (gatilhos para acionar este SKILL)

- [ ] `string` representando CPF, telefone, CEP, e-mail, código de produto
- [ ] Constante `int` ou `string` simulando tipo (ex: `const ROLE_ADMIN = 1`)
- [ ] Array associativo com chaves mágicas para estruturar dados de domínio
- [ ] Múltiplos parâmetros primitivos que sempre aparecem juntos (ex: `float $valor, string $moeda`)
- [ ] Validação do mesmo primitivo repetida em vários lugares

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                  | Técnica indicada               |
|------------------------------------------------------|-------------------------------|
| Primitivo com regras de validação próprias            | Replace Data Value with Object |
| Múltiplos primitivos que andam juntos                | Introduce Parameter Object     |
| Primitivo passado como grupo                         | Preserve Whole Object          |
| `int`/`string` simulando tipo enumerado              | Replace Type Code with Class   |
| Array associativo como estrutura de dados            | Replace Array with Object      |

---

## 4. Exemplo

**ANTES — não aceito:**
```php
class Cliente
{
    public function __construct(
        private string $nome,
        private string $cpf,       // string crua, sem validação centralizada
        private string $telefone,
        private string $email,
    ) {}
}

class PedidoService
{
    public function criar(
        string $cpfCliente,
        float $valor,
        string $moeda,
        int $tipoPagamento   // 1 = cartão, 2 = boleto, 3 = pix — constante mágica
    ): void {
        if ($tipoPagamento === 1) {
            // ...
        }
        // validação de CPF duplicada em vários pontos
        if (empty($cpfCliente) || strlen($cpfCliente) !== 11) {
            throw new \InvalidArgumentException("CPF inválido");
        }
    }

    public function buscarEndereco(array $dados): void
    {
        // chaves mágicas no array
        $rua    = $dados['rua'];
        $numero = $dados['numero'];
        $cidade = $dados['cidade'];
        $cep    = $dados['cep'];
    }
}
```

**DEPOIS — esperado:**
```php
// Value Object para CPF
final class Cpf
{
    private readonly string $valor;

    public function __construct(string $valor)
    {
        if (!preg_match('/^\d{11}$/', $valor)) {
            throw new \InvalidArgumentException("CPF inválido");
        }
        $this->valor = $valor;
    }

    public function getValor(): string
    {
        return $this->valor;
    }
}

// Value Object para Email
final class Email
{
    private readonly string $endereco;

    public function __construct(string $endereco)
    {
        if (!filter_var($endereco, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException("E-mail inválido");
        }
        $this->endereco = $endereco;
    }

    public function getEndereco(): string
    {
        return $this->endereco;
    }
}

// Parameter Object para dados que andam juntos
final class Dinheiro
{
    public function __construct(
        private readonly float $valor,
        private readonly string $moeda,
    ) {}

    public function getValor(): float { return $this->valor; }
    public function getMoeda(): string { return $this->moeda; }
}

// Enum no lugar de constante inteira (PHP 8.1+)
enum TipoPagamento: int
{
    case Cartao = 1;
    case Boleto = 2;
    case Pix    = 3;
}

// Replace Array with Object
final class Endereco
{
    public function __construct(
        private readonly string $rua,
        private readonly string $numero,
        private readonly string $cidade,
        private readonly string $cep,
    ) {}

    public function getRua(): string    { return $this->rua; }
    public function getNumero(): string { return $this->numero; }
    public function getCidade(): string { return $this->cidade; }
    public function getCep(): string    { return $this->cep; }
}

class Cliente
{
    public function __construct(
        private string $nome,
        private Cpf $cpf,
        private Email $email,
    ) {}
}

class PedidoService
{
    public function criar(Cpf $cpfCliente, Dinheiro $valor, TipoPagamento $tipoPagamento): void
    {
        if ($tipoPagamento === TipoPagamento::Cartao) {
            // ...
        }
    }

    public function buscarEndereco(Endereco $endereco): void
    {
        $rua    = $endereco->getRua();
        $cidade = $endereco->getCidade();
    }
}
```

**Por que esse padrão:**
- A validação do CPF está centralizada em `Cpf` — não se repete
- `TipoPagamento` é autoexplicativo, sem inteiros mágicos
- `Dinheiro` agrupa valor e moeda — evita que andem separados
- `Endereco` substitui o array com propriedades tipadas e nomeadas

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: `string` para tudo**
```php
// Não aceito
public function processar(string $cpf, string $status, string $tipoPagamento): void
{
    if ($status === 'ativo' && $tipoPagamento === 'pix') { ... }
}
```

**Erro 2: Constantes inteiras no lugar de Enum**
```php
// Não aceito
const PAGAMENTO_CARTAO = 1;
const PAGAMENTO_BOLETO = 2;
if ($tipo === self::PAGAMENTO_CARTAO) { ... }
```

**Erro 3: Array com chaves mágicas**
```php
// Não aceito
$endereco = [
    'rua'    => 'Rua A',
    'numero' => '123',
    'cidade' => 'SP',
    'cep'    => '01310-100',
];
```

---

## 6. Benefícios

- **Flexibilidade:** Regras de negócio ficam encapsuladas no objeto correto
- **Legibilidade:** Parâmetros e campos expressam intenção de domínio
- **Manutenção:** Validação centralizada — muda em um lugar, vale em todos
- **Segurança:** Type hints estritos eliminam erros de tipo em tempo de análise estática
