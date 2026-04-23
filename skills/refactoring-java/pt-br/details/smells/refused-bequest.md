# SMELL: Refused Bequest — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/refused-bequest

---

## 1. O que é?

Uma subclasse herda métodos ou dados de sua classe pai mas não os utiliza ou os sobrescreve ativamente lançando exceções. O filho rejeita parte do que o pai lhe dá, o que viola o Princípio de Substituição de Liskov: uma subclasse deve ser utilizável onde quer que seu pai seja esperado.

---

## 2. Sinais de alerta

- [ ] Uma subclasse sobrescreve um método pai apenas para lançar `UnsupportedOperationException` ou similar
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
```java
public class Ave {
    public String getNome() { return nome; }
    public void voar() { /* bater asas */ }
    public void pousar() { /* aterrissar */ }
}

// Pinguim É-UMA Ave no papel, mas não consegue voar
public class Pinguim extends Ave {
    @Override
    public void voar() {
        throw new UnsupportedOperationException("Pinguins não podem voar");
    }
}

// Chamador quebra em tempo de execução
Ave a = new Pinguim();
a.voar(); // lança exceção!
```

**DEPOIS — esperado:**
```java
// Divida a hierarquia ao longo das capacidades reais
public abstract class Ave {
    public abstract String getNome();
}

public interface Voavel {
    void voar();
    void pousar();
}

public class Pardal extends Ave implements Voavel {
    @Override public String getNome() { return "Pardal"; }
    @Override public void voar() { /* bater asas */ }
    @Override public void pousar() { /* aterrissar */ }
}

public class Pinguim extends Ave {
    @Override public String getNome() { return "Pinguim"; }
    public void nadar() { /* nadar */ }
}
```

**Por que este padrão:**
- `Ave` agora contém apenas comportamento compartilhado por TODAS as aves
- `Voavel` é uma capacidade opcional — pinguins simplesmente não a implementam
- Código que chama `voar()` deve ter um `Voavel`, não apenas qualquer `Ave`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter a exceção e documentá-la**
```java
// Não aceito — o LSP ainda é violado; chamadores não podem confiar no contrato
@Override
public void voar() {
    // Pinguins não voam — documentado mas ainda quebrado
    throw new UnsupportedOperationException();
}
```

**Erro 2: Retornar um no-op em vez de lançar exceção**
```java
// Não aceito — falhas silenciosas escondem bugs; chamadores assumem que a ação aconteceu
@Override
public void voar() { /* não faz nada */ }
```

---

## 6. Benefícios

- **Conformidade com LSP:** Cada subclasse honra o contrato completo de seu pai
- **Polimorfismo mais seguro:** Código que itera uma coleção de objetos `Ave` não irá quebrar
- **Hierarquia mais limpa:** Capacidades se tornam interfaces explícitas em vez de exceções ocultas
