# TÉCNICA: Replace Conditional with Polymorphism — PHP

## Fonte
Baseado em: https://refactoring.guru/replace-conditional-with-polymorphism

---

## 1. Problema

Você tem um `switch` ou cadeia de `if/elseif/else` que executa comportamentos diferentes
dependendo do tipo do objeto ou de uma propriedade que simula um tipo.

---

## 2. Solução

Crie subclasses correspondentes a cada ramo do condicional. Em cada subclasse,
implemente um método que contém o comportamento daquele ramo. Substitua o condicional
por uma chamada polimórfica ao método.

---

## 3. Quando aplicar

- O condicional verifica um campo tipo, string mágica ou constante de classe
- O mesmo condicional aparece em múltiplos métodos
- Adicionar um novo tipo exigiria modificar todos os condicionais existentes (violação do Open/Closed Principle)
- Cada ramo do condicional representa um comportamento coeso e distinto

---

## 4. Passos de refatoração

1. Se o condicional está dentro de um método com outras responsabilidades, aplique **Extract Method** primeiro
2. Crie uma classe base `abstract` (ou interface) com o método a ser polimorfizado
3. Para cada ramo do condicional, crie uma subclasse que sobrescreve o método
4. Mova a lógica do ramo para o método da subclasse correspondente
5. Remova o ramo do condicional original
6. Repita até o condicional estar vazio, então declare o método como `abstract` na classe base
7. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
class Funcionario
{
    public function __construct(
        private string $tipo,        // "engenheiro", "gerente", "vendedor"
        private float $salarioBase,
        private float $vendas = 0,
    ) {}

    public function calcularBonus(): float
    {
        return match ($this->tipo) {
            'engenheiro' => $this->salarioBase * 0.10,
            'gerente'    => $this->salarioBase * 0.20 + 1000,
            'vendedor'   => $this->vendas * 0.05,
            default      => throw new \InvalidArgumentException("Tipo desconhecido: {$this->tipo}"),
        };
    }

    public function gerarRelatorio(): string
    {
        return match ($this->tipo) {
            'engenheiro' => "Engenheiro: projetos entregues",
            'gerente'    => "Gerente: equipes lideradas",
            'vendedor'   => "Vendedor: R$ {$this->vendas} em vendas",
            default      => throw new \InvalidArgumentException("Tipo desconhecido: {$this->tipo}"),
        };
    }
}
```

**DEPOIS — esperado:**
```php
abstract class Funcionario
{
    public function __construct(protected float $salarioBase) {}

    abstract public function calcularBonus(): float;
    abstract public function gerarRelatorio(): string;
}

class Engenheiro extends Funcionario
{
    public function calcularBonus(): float
    {
        return $this->salarioBase * 0.10;
    }

    public function gerarRelatorio(): string
    {
        return "Engenheiro: projetos entregues";
    }
}

class Gerente extends Funcionario
{
    public function calcularBonus(): float
    {
        return $this->salarioBase * 0.20 + 1000;
    }

    public function gerarRelatorio(): string
    {
        return "Gerente: equipes lideradas";
    }
}

class Vendedor extends Funcionario
{
    public function __construct(float $salarioBase, private float $vendas)
    {
        parent::__construct($salarioBase);
    }

    public function calcularBonus(): float
    {
        return $this->vendas * 0.05;
    }

    public function gerarRelatorio(): string
    {
        return "Vendedor: R$ {$this->vendas} em vendas";
    }
}
```

**Por que esse padrão:**
- Adicionar um novo tipo de funcionário → criar nova subclasse, sem tocar nas existentes
- Sem `match`/`switch` duplicado em `calcularBonus` e `gerarRelatorio`
- `abstract` garante em tempo de definição que nenhuma subclasse esqueça de implementar os métodos

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover o switch para uma factory e achar que resolveu**
```php
// Não aceito — o condicional continua existindo, só mudou de lugar
class FuncionarioFactory
{
    public static function criar(string $tipo, float $salario): Funcionario
    {
        return match ($tipo) {
            'engenheiro' => new Funcionario('engenheiro', $salario), // mesma classe
        };
    }
}
```

**Erro 2: Criar subclasses mas manter condicional nos métodos**
```php
// Não aceito — criou herança mas não polimorfizou o comportamento
class Engenheiro extends Funcionario
{
    public function calcularBonus(): float
    {
        if ($this->tipo === 'engenheiro') {  // condicional desnecessário
            return $this->salarioBase * 0.10;
        }
        return 0;
    }
}
```

**Erro 3: Usar polimorfismo para condicionais triviais**
```php
// Não aceito — over-engineering para um if simples
// Ex: if ($ativo) { ... } else { ... }
```

---

## 7. Benefícios

- **Open/Closed Principle:** Novos tipos não exigem modificação do código existente
- **Eliminação de duplicação:** O mesmo condicional não precisa ser repetido
- **Legibilidade:** Cada classe expressa claramente seu comportamento
- **Testabilidade:** Cada subclasse pode ser testada isoladamente
