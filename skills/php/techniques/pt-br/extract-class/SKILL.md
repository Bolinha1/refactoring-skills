# TÉCNICA: Extract Class — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/extract-class

---

## 1. Problema

Uma classe faz o trabalho de duas. Um subconjunto de seus campos e métodos forma um conceito coeso que faria sentido como sua própria classe.

---

## 2. Solução

Crie uma nova classe e mova os campos e métodos que pertencem a esse conceito para ela. Substitua os campos originais por uma referência à nova classe.

---

## 3. Quando aplicar

- A classe tem campos que são sempre usados juntos (Data Clumps)
- A classe tem métodos que tocam apenas um subconjunto de seus campos
- A classe cresceu para lidar com mais de uma responsabilidade distinta (Divergent Change)
- Um subconjunto da classe poderia ser reutilizado independentemente em outro contexto
- O nome da classe não consegue descrever o que ela faz sem usar "e"

---

## 4. Passos de refatoração

1. Identifique o grupo de campos e métodos a extrair — eles devem formar um conceito coeso
2. Crie uma nova classe com um nome que expresse esse conceito
3. Crie uma instância da nova classe como campo na classe original
4. Mova os campos identificados para a nova classe (use Move Field para cada um)
5. Mova os métodos identificados para a nova classe (use Move Method para cada um)
6. Decida sobre a visibilidade:
   - Torne-a `public` se pode ser usada independentemente ou reutilizada
   - Mantenha interna se é um detalhe privado da classe original
7. Compile e execute os testes após cada movimentação

---

## 5. Exemplo

**ANTES — não aceito:**
```php
class Pessoa
{
    private string $nome;
    private string $codigoAreaEscritorio;
    private string $numeroEscritorio;

    public function getNumeroTelefone(): string
    {
        return "({$this->codigoAreaEscritorio}) {$this->numeroEscritorio}";
    }
}
```

**DEPOIS — esperado:**
```php
final class NumeroTelefone
{
    public function __construct(
        private readonly string $codigoArea,
        private readonly string $numero,
    ) {}

    public function formatar(): string
    {
        return "({$this->codigoArea}) {$this->numero}";
    }
}

class Pessoa
{
    public function __construct(
        private string $nome,
        private NumeroTelefone $telefoneEscritorio,
    ) {}

    public function getNumeroTelefone(): string
    {
        return $this->telefoneEscritorio->formatar();
    }
}
```

**Variante — extraindo um data clump:**
```php
// ANTES — campos de endereço espalhados em Pessoa e Fatura
class Pessoa
{
    private string $rua;
    private string $cidade;
    private string $cep;
}

// DEPOIS — Endereco é um conceito independente e reutilizável
final class Endereco
{
    public function __construct(
        public readonly string $rua,
        public readonly string $cidade,
        public readonly string $cep,
    ) {}

    public function formatar(): string
    {
        return "{$this->rua}, {$this->cidade} {$this->cep}";
    }
}

class Pessoa
{
    public function __construct(private Endereco $enderecoResidencial) {}
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair uma classe sem coesão**
```php
// Não aceito — DadosPessoa agrupa campos sem relacionamento natural
class DadosPessoa
{
    public string $nome;
    public string $codigoEscritorio;
    public float $salario;
    public \DateTime $dataContratacao;
}
```

**Erro 2: Mover métodos sem mover os campos que eles precisam**
```php
// Não aceito — NumeroTelefone::formatar() ainda recebe campos brutos de Pessoa
class NumeroTelefone
{
    public function formatar(string $codigoArea, string $numero): string { ... } // ainda acoplado
}
```

**Erro 3: Extrair mas deixar os campos originais no lugar**
```php
// Não aceito — agora há duas fontes da verdade para dados de telefone
class Pessoa
{
    private NumeroTelefone $telefone;       // novo
    private string $codigoAreaEscritorio;   // antigo — deve ser deletado
    private string $numeroEscritorio;       // antigo — deve ser deletado
}
```

---

## 7. Benefícios

- **Responsabilidade Única:** Cada classe tem um conceito claro para representar
- **Reutilização:** A classe extraída pode ser usada por outras classes independentemente
- **Encapsulamento:** Lógica de validação e formatação do conceito vive em um único lugar
