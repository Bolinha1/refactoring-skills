# SMELL: Refused Bequest — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/refused-bequest

---

## 1. O que é?

Uma subclasse herda métodos ou dados de sua classe pai mas não os utiliza ou os sobrescreve ativamente lançando exceções. O filho rejeita parte do que o pai lhe dá, o que viola o Princípio de Substituição de Liskov: uma subclasse deve ser utilizável onde quer que seu pai seja esperado.

---

## 2. Sinais de alerta

- [ ] Uma subclasse sobrescreve um método pai apenas para lançar `\BadMethodCallException` ou similar
- [ ] Campos herdados nunca são lidos ou escritos pela subclasse
- [ ] Testes que esperam comportamento da classe base falham quando recebem a subclasse
- [ ] A subclasse usa apenas uma pequena fração da API pública do pai
- [ ] O relacionamento é "é-um" apenas no papel — comportamentalmente não é

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Replace Inheritance with Delegation** | A subclasse quer algum comportamento do pai mas não todos — envolva em vez de herdar |
| **Extract Superclass** | Puxe apenas o comportamento compartilhado para um novo pai; deixe o resto em classes irmãs |
| **Push Down Method / Push Down Field** | Mova membros herdados indesejados para baixo, fora do pai, para que a subclasse nunca os receba |

---

## 4. Exemplo

**ANTES — não aceito:**
```php
class Ave {
    public function getNome(): string { return $this->nome; }
    public function voar(): void { /* bater asas */ }
    public function pousar(): void { /* aterrissar */ }
}

// Pinguim É-UMA Ave no papel, mas não consegue voar
class Pinguim extends Ave {
    public function voar(): void {
        throw new \BadMethodCallException('Pinguins não podem voar');
    }
}

// Chamador quebra em tempo de execução
$a = new Pinguim();
$a->voar(); // lança exceção!
```

**DEPOIS — esperado:**
```php
// Divida a hierarquia ao longo das capacidades reais
abstract class Ave {
    abstract public function getNome(): string;
}

interface Voavel {
    public function voar(): void;
    public function pousar(): void;
}

class Pardal extends Ave implements Voavel {
    public function getNome(): string { return 'Pardal'; }
    public function voar(): void { /* bater asas */ }
    public function pousar(): void { /* aterrissar */ }
}

class Pinguim extends Ave {
    public function getNome(): string { return 'Pinguim'; }
    public function nadar(): void { /* nadar */ }
}
```

**Por que este padrão:**
- `Ave` agora contém apenas comportamento compartilhado por TODAS as aves
- `Voavel` é uma capacidade opcional — pinguins simplesmente não a implementam
- Código que chama `voar()` deve declarar `Voavel`, não apenas qualquer `Ave`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter a exceção e documentá-la**
```php
// Não aceito — o LSP ainda é violado; chamadores não podem confiar no contrato
public function voar(): void {
    // Pinguins não voam — documentado mas ainda quebrado
    throw new \BadMethodCallException();
}
```

**Erro 2: Retornar um no-op em vez de lançar exceção**
```php
// Não aceito — falhas silenciosas escondem bugs; chamadores assumem que a ação aconteceu
public function voar(): void { /* não faz nada */ }
```

---

## 6. Benefícios

- **Conformidade com LSP:** Cada subclasse honra o contrato completo de seu pai
- **Polimorfismo mais seguro:** Código que itera uma coleção de objetos `Ave` não irá quebrar
- **Hierarquia mais limpa:** Capacidades se tornam interfaces explícitas em vez de exceções ocultas
