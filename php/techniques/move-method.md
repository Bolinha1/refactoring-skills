# TÉCNICA: Move Method — PHP

## Fonte
Baseado em: https://refactoring.guru/move-method

---

## 1. Problema

Um método é mais usado por outra classe do que pela própria classe onde está definido.
Isso cria acoplamento desnecessário e viola o princípio de coesão.

---

## 2. Solução

Declare o método na classe que o usa com mais frequência. Mova o código original para lá.
No lugar original, substitua o corpo por uma delegação ao novo método — ou remova-o completamente
se não for mais necessário.

---

## 3. Quando aplicar

- O método acessa propriedades de outra classe mais do que as da própria classe
- O método seria mais útil na classe que o consome
- Mover o método reduziria ou eliminaria dependências entre classes
- O smell **Feature Envy** está presente (método "inveja" os dados de outra classe)

---

## 4. Passos de refatoração

1. Analise as dependências do método dentro da classe atual
2. Verifique se o método existe em classes pai ou filha (evite quebrar polimorfismo)
3. Declare o método na classe de destino com nome contextualmente adequado
4. Obtenha uma referência à classe de destino (via propriedade injetada, parâmetro ou instância local)
5. Transforme o método original em uma delegação — ou delete-o se não houver chamadores externos
6. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
class Conta
{
    public function __construct(
        private float $saldo,
        private TipoContrato $contrato,
    ) {}

    public function calcularEncargo(): float
    {
        // esse método usa quase só dados de TipoContrato — está no lugar errado
        if ($this->contrato->getTipo() === 'especial') {
            return $this->contrato->getTaxaEspecial() * $this->saldo * 30;
        }
        return $this->contrato->getTaxaPadrao() * $this->saldo * 30;
    }
}

class TipoContrato
{
    public function __construct(
        private string $tipo,
        private float $taxaPadrao,
        private float $taxaEspecial,
    ) {}

    public function getTipo(): string       { return $this->tipo; }
    public function getTaxaPadrao(): float  { return $this->taxaPadrao; }
    public function getTaxaEspecial(): float { return $this->taxaEspecial; }
}
```

**DEPOIS — esperado:**
```php
class Conta
{
    public function __construct(
        private float $saldo,
        private TipoContrato $contrato,
    ) {}

    public function calcularEncargo(): float
    {
        // delega para quem realmente tem os dados
        return $this->contrato->calcularEncargo($this->saldo);
    }
}

class TipoContrato
{
    public function __construct(
        private string $tipo,
        private float $taxaPadrao,
        private float $taxaEspecial,
    ) {}

    public function calcularEncargo(float $saldo): float
    {
        if ($this->tipo === 'especial') {
            return $this->taxaEspecial * $saldo * 30;
        }
        return $this->taxaPadrao * $saldo * 30;
    }
}
```

**Por que esse padrão:**
- `calcularEncargo` usava `getTipo()`, `getTaxaEspecial()` e `getTaxaPadrao()` de `TipoContrato`
- O método pertence a `TipoContrato` — é lá que estão os dados necessários
- `Conta` agora apenas coordena, sem precisar conhecer os detalhes de `TipoContrato`

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover e criar dependência mútua entre as classes**
```php
// Não aceito — cria acoplamento circular
class TipoContrato
{
    public function calcularEncargo(Conta $conta): float
    {
        return $this->taxa * $conta->getSaldo() * $conta->getDias(); // agora depende de Conta
    }
}
```

**Erro 2: Mover método sobrescrito em subclasses sem avaliar impacto**
```php
// Não aceito — se calcularEncargo() é sobrescrito em ContaEspecial,
// mover sem cuidado quebra o contrato polimórfico
```

**Erro 3: Manter a versão original com lógica duplicada**
```php
// Não aceito — dois métodos com o mesmo comportamento em classes diferentes
```

---

## 7. Benefícios

- **Coesão:** Cada classe contém os métodos que pertencem aos seus dados
- **Acoplamento reduzido:** Elimina dependências desnecessárias entre classes
- **Manutenção:** Mudanças na lógica afetam apenas a classe correta
