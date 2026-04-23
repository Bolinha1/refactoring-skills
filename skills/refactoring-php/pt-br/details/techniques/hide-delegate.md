# TÉCNICA: Hide Delegate — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/hide-delegate

---

## 1. Problema

Um cliente acessa um objeto delegado navegando por uma cadeia através do objeto servidor:
`cliente → servidor → delegado`. Quando o delegado muda, o cliente também precisa mudar.

---

## 2. Solução

Crie um método no servidor que esconda o delegado. O cliente agora só fala com o servidor.

---

## 3. Quando aplicar

- Um cliente chama `$servidor->getDelegado()->fazerAlgo()`
- Uma mudança na classe delegada requer mudanças em todos os clientes que navegam até ela
- A cadeia de delegação cria o smell Message Chains
- Você quer que clientes dependam apenas da interface pública do servidor, não de sua estrutura interna

---

## 4. Passos de refatoração

1. Para cada método do delegado que clientes chamam através do servidor:
   - Crie um método delegante na classe servidor
2. Atualize o cliente para chamar o novo método do servidor em vez de navegar pela cadeia
3. Se nenhum cliente mais acessa o delegado diretamente, remova o getter do delegado
4. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```php
class Pessoa
{
    public function __construct(private Departamento $departamento) {}

    public function getDepartamento(): Departamento
    {
        return $this->departamento; // exposto — clientes navegam por ele
    }
}

class Departamento
{
    public function __construct(private Pessoa $gerente) {}

    public function getGerente(): Pessoa
    {
        return $this->gerente;
    }
}

// Código do cliente — precisa conhecer Pessoa e Departamento
$gerente = $pessoa->getDepartamento()->getGerente();
```

**DEPOIS — esperado:**
```php
class Pessoa
{
    public function __construct(private Departamento $departamento) {}

    public function getGerente(): Pessoa
    {
        return $this->departamento->getGerente(); // delega, esconde Departamento do cliente
    }
    // getDepartamento() pode ser removido se ninguém mais precisar dele
}

// Código do cliente — só conhece Pessoa
$gerente = $pessoa->getGerente();
```

**Por que este padrão:**
- Clientes são desacoplados da existência e estrutura de `Departamento`
- Mudar como `Pessoa` encontra seu gerente é agora uma mudança em um único lugar

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Adicionar o método delegante mas manter o getter também**
```php
// Não aceito — getDepartamento() ainda expõe o delegado; clientes usarão ambos os caminhos
public function getDepartamento(): Departamento { return $this->departamento; }  // deveria ser removido
public function getGerente(): Pessoa { return $this->departamento->getGerente(); }
```

**Erro 2: Criar muitos métodos pass-through para tudo no delegado**
```php
// Não aceito — se clientes precisam de muitas partes diferentes de Departamento,
// 10 métodos pass-through apenas deslocam a complexidade sem reduzi-la
// (considere Remove Middle Man em vez disso)
```

**Erro 3: Não remover a cadeia após adicionar o wrapper**
```php
// Não aceito — o método existe mas chamadores ainda usam a cadeia antiga
$pessoa->getDepartamento()->getGerente(); // ainda compila — velhos hábitos persistem
```

---

## 7. Benefícios

- **Encapsulamento:** A estrutura interna do servidor é escondida dos clientes
- **Resiliência:** Mudar a API do delegado requer apenas mudar a classe servidor
- **Simplicidade:** Clientes expressam intenção (`getGerente()`) em vez de navegar pela estrutura
