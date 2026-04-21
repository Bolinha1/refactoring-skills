# TÉCNICA: Inline Class — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/inline-class

---

## 1. Problema

Uma classe não faz mais o suficiente para justificar sua existência. Ela tem responsabilidades demais poucas e é apenas uma indireção desnecessária.

---

## 2. Solução

Mova todas as funcionalidades da classe para outra classe e então delete-a.

---

## 3. Quando aplicar

- Após uma refatoração, uma classe ficou com apenas um ou dois métodos triviais
- Uma classe é usada em apenas um lugar e não adiciona abstração real
- Uma classe que foi dividida de forma muito agressiva precisa ser consolidada
- A classe é um wrapper fino que apenas delega tudo para outra classe

---

## 4. Passos de refatoração

1. Identifique a classe a fazer inline e a classe alvo (para onde as funcionalidades irão)
2. Para cada funcionalidade pública na classe a fazer inline:
   - Copie-a para a classe alvo
   - Atualize a classe alvo para delegar à classe inline (transição segura)
3. Atualize todos os chamadores para usar a classe alvo diretamente
4. Remova a delegação da classe alvo (métodos agora funcionam nativamente)
5. Delete a classe que foi inlined
6. Compile e execute os testes após cada passo

---

## 5. Exemplo

**ANTES — não aceito:**
```php
class NumeroTelefone
{
    public function __construct(private string $numero) {}

    public function getNumero(): string
    {
        return $this->numero;
    }
}

class Pessoa
{
    public function __construct(
        private string $nome,
        private NumeroTelefone $telefone,
    ) {}

    public function getNumeroTelefone(): string
    {
        return $this->telefone->getNumero(); // NumeroTelefone é apenas um wrapper
    }
}
```

**DEPOIS — esperado:**
```php
class Pessoa
{
    public function __construct(
        private string $nome,
        private string $numeroTelefone, // inlined diretamente
    ) {}

    public function getNumeroTelefone(): string
    {
        return $this->numeroTelefone;
    }
}
```

**Caso de uso: consolidando Shotgun Surgery**
```php
// ANTES — quatro classes de serviço minúsculas divididas de forma muito agressiva
class BuscadorPedido  { public function buscar(int $id): Pedido { ... } }
class SalvadorPedido  { public function salvar(Pedido $p): void { ... } }
class AtualizadorPedido { public function atualizar(Pedido $p): void { ... } }
class DeletadorPedido { public function deletar(int $id): void { ... } }

// DEPOIS — fazer inline de todos em um único repositório coeso
class RepositorioPedido
{
    public function buscar(int $id): Pedido { ... }
    public function salvar(Pedido $p): void { ... }
    public function atualizar(Pedido $p): void { ... }
    public function deletar(int $id): void { ... }
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Fazer inline de uma classe que ainda tem um conceito significativo**
```php
// Não aceito — Dinheiro tem comportamento (aritmética, formatação) que pertence junto
// Não faça inline de Dinheiro em Pedido só porque tem poucos campos
```

**Erro 2: Fazer inline sem remover a classe original**
```php
// Não aceito — tanto Pessoa.$numeroTelefone quanto NumeroTelefone existem simultaneamente
class Pessoa
{
    private string $numeroTelefone;      // cópia inlined
    private NumeroTelefone $telefone;    // ainda aqui — cria confusão
}
```

**Erro 3: Fazer inline de uma classe usada em muitos lugares**
```php
// Não aceito — se NumeroTelefone é usado por Pessoa, Funcionario e Contato,
// fazer inline exigiria duplicar a lógica em três lugares
```

---

## 7. Benefícios

- **Simplicidade:** Remove indireção desnecessária da base de código
- **Legibilidade:** Menos classes para navegar quando a abstração não adiciona valor
- **Preparação:** Frequentemente precede Extract Class — fazer inline primeiro, depois re-extrair corretamente
