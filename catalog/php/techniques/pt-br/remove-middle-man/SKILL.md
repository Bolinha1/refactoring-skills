# TÉCNICA: Remove Middle Man — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/remove-middle-man

---

## 1. Problema

Uma classe tem muitos métodos de delegação simples que não fazem nada além de encaminhar chamadas para outro objeto. A classe é um Middle Man — existe apenas para passar mensagens adiante.

---

## 2. Solução

Remova os métodos de delegação. Deixe o cliente acessar o delegado diretamente.

---

## 3. Quando aplicar

- A classe servidor cresceu com um grande número de métodos delegantes de uma linha
- A classe servidor delega tanto que não tem comportamento real próprio
- Adicionar um novo recurso ao delegado requer adicionar um novo pass-through no servidor
- O smell Middle Man foi identificado na classe servidor

**Nota:** Este é o inverso do Hide Delegate. Aplique Hide Delegate quando quiser encapsulamento; aplique Remove Middle Man quando o encapsulamento se tornou um obstáculo.

---

## 4. Passos de refatoração

1. Identifique todos os métodos de delegação na classe middle man
2. Para cada método de delegação, encontre seus chamadores
3. Atualize cada chamador para chamar o delegado diretamente
4. Remova cada método de delegação do middle man
5. Se o middle man agora não tem comportamento restante, delete-o ou mescle-o com o chamador
6. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
// Pessoa tornou-se um middle man puro — apenas encaminha para Departamento
class Pessoa
{
    public function __construct(private Departamento $departamento) {}

    public function getGerente(): Pessoa             { return $this->departamento->getGerente(); }
    public function getFuncionarios(): array          { return $this->departamento->getFuncionarios(); }
    public function getNomeDepartamento(): string     { return $this->departamento->getNome(); }
    public function getOrcamentoDepartamento(): int   { return $this->departamento->getOrcamento(); }
}
```

**DEPOIS — esperado:**
```php
class Pessoa
{
    public function __construct(private Departamento $departamento) {}

    public function getDepartamento(): Departamento
    {
        return $this->departamento; // expõe o delegado diretamente
    }
    // Todos os métodos pass-through removidos
}

// Chamadores acessam Departamento diretamente onde precisam
$dept = $pessoa->getDepartamento();
$gerente = $dept->getGerente();
$equipe = $dept->getFuncionarios();
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Remover o middle man mas expor objetos internos que deveriam ser escondidos**
```php
// Não aceito — se Departamento é um detalhe interno, expô-lo quebra o encapsulamento
// Remova o middle man apenas se acesso direto for apropriado para a arquitetura
```

**Erro 2: Remover métodos de delegação que têm lógica real neles**
```php
// Não aceito — isso não é delegação pura; tem uma guarda
public function getGerente(): ?Pessoa
{
    if ($this->departamento === null) {  // lógica real — não remova
        return null;
    }
    return $this->departamento->getGerente();
}
```

**Erro 3: Aplicar Remove Middle Man quando Hide Delegate é na verdade o correto**
```php
// Não aceito — se clientes não deveriam conhecer Departamento de forma alguma,
// Hide Delegate é a escolha certa; Remove Middle Man vai na direção oposta
```

---

## 7. Benefícios

- **Simplicidade:** Remove indireção desnecessária
- **Transparência:** Chamadores têm um caminho direto para o que precisam
- **Manutenibilidade:** Adicionar recursos ao delegado não requer mais tocar o middle man
