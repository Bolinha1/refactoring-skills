# SMELL: Temporary Field — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/temporary-field

---

## 1. O que é?

Uma classe contém um campo que só é definido e usado em certas situações — durante um algoritmo ou chamada de método. No restante do tempo ele é `null`, vazio ou sem significado. Outros códigos que leem o campo não conseguem saber quando ele tem um valor válido e quando não tem.

---

## 2. Sinais de alerta

- [ ] Um campo é `null` ou não inicializado na maior parte do tempo
- [ ] Um campo é definido em um método e consumido em outro, mas nunca mantido durante o tempo de vida do objeto
- [ ] Um campo existe apenas porque um algoritmo longo precisava de um lugar para colocar estado intermediário
- [ ] A classe tem um padrão "prepare então execute" onde campos são preenchidos logo antes do uso
- [ ] Verificações de null para campos de instância aparecem em toda a classe

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Extract Class** | Mova os campos temporários (e os métodos que os usam) para uma classe dedicada |
| **Introduce Null Object** | Substitua o estado null/vazio por um Null Object adequado para que chamadores não precisem verificar |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
// CalculadoraRota mantém campos que só são válidos durante calcularRota()
public class CalculadoraRota {
    private String[] paradas;       // só definido durante o cálculo
    private int distanciaTotal;     // só válido após calcular()
    private String caminhoMaisRapido; // null a menos que calcular() tenha executado

    public void setParadas(String[] paradas) {
        this.paradas = paradas;
    }

    public void calcular() {
        this.distanciaTotal = calcularDistancia(this.paradas);
        this.caminhoMaisRapido = encontrarCaminhoMaisRapido(this.paradas);
    }

    public String getCaminhoMaisRapido() {
        return this.caminhoMaisRapido; // null se calcular() não foi chamado primeiro
    }
}
```

**DEPOIS — esperado:**
```java
// Extraia o estado temporário para um objeto de valor dedicado
public class ResultadoRota {
    private final int distanciaTotal;
    private final String caminhoMaisRapido;

    public ResultadoRota(int distanciaTotal, String caminhoMaisRapido) {
        this.distanciaTotal = distanciaTotal;
        this.caminhoMaisRapido = caminhoMaisRapido;
    }

    public int getDistanciaTotal() { return distanciaTotal; }
    public String getCaminhoMaisRapido() { return caminhoMaisRapido; }
}

public class CalculadoraRota {
    public ResultadoRota calcular(String[] paradas) {
        int distancia = calcularDistancia(paradas);
        String caminho = encontrarCaminhoMaisRapido(paradas);
        return new ResultadoRota(distancia, caminho);
    }
}
```

**Por que este padrão:**
- `ResultadoRota` só existe quando contém dados válidos — não há estado null
- `CalculadoraRota` é sem estado: fácil de reutilizar e testar em paralelo

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Adicionar uma flag `pronto` em vez de corrigir o design**
```java
// Não aceito — disciplina de flag é frágil; Extract Class remove a necessidade dela
private boolean calculado = false;
public String getCaminhoMaisRapido() {
    if (!calculado) throw new IllegalStateException("Chame calcular() primeiro");
    return this.caminhoMaisRapido;
}
```

**Erro 2: Mover o campo temporário para uma subclasse**
```java
// Não aceito — o campo ainda viaja com o objeto; o smell permanece
class CalculadoraRotaCalculando extends CalculadoraRota {
    private String caminhoMaisRapido; // mesmo problema, apenas renomeado
}
```

---

## 6. Benefícios

- **Correção:** Sem mais estado null se passando por valor válido
- **Clareza:** Os campos da classe são sempre significativos independente de quando você os observa
- **Testabilidade:** A classe extraída pode ser testada com dados válidos e totalmente inicializados
