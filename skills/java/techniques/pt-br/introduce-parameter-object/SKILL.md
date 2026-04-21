# TÉCNICA: Introduce Parameter Object — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/introduce-parameter-object

---

## 1. Problema

Vários parâmetros que naturalmente pertencem juntos são sempre passados como grupo para os mesmos métodos. O grupo representa um conceito ainda não formalizado na base de código.

---

## 2. Solução

Substitua o grupo de parâmetros por um único objeto que representa o conceito. O objeto se torna um cidadão de primeira classe no modelo de domínio.

---

## 3. Quando aplicar

- Três ou mais parâmetros que sempre aparecem juntos em assinaturas de método
- O mesmo grupo aparece em múltiplos métodos pela base de código
- O grupo representa um conceito de domínio reconhecível (intervalo de datas, endereço, coordenadas)
- Você quer adicionar comportamento (validação, formatação) ao grupo de dados

---

## 4. Passos de refatoração

1. Crie uma nova classe para representar o grupo de parâmetros
2. Adicione os parâmetros como campos na nova classe (torne-os imutáveis se representam um valor)
3. Adicione um construtor que receba os parâmetros originais
4. Atualize cada método que recebe o grupo de parâmetros para aceitar a nova classe
5. Atualize todos os locais de chamada para construir o novo objeto e passá-lo
6. Procure por comportamento (verificações, cálculos) nos chamadores que poderiam ser movidos para a nova classe
7. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public class ServiçoRelatorio {
    public List<Venda> buscarVendasPorPeriodo(LocalDate dataInicio, LocalDate dataFim) { ... }
    public double somarReceitaPorPeriodo(LocalDate dataInicio, LocalDate dataFim) { ... }
    public List<Fatura> faturasPorPeriodo(LocalDate dataInicio, LocalDate dataFim) { ... }
}
```

**DEPOIS — esperado:**
```java
public class Periodo {
    private final LocalDate inicio;
    private final LocalDate fim;

    public Periodo(LocalDate inicio, LocalDate fim) {
        if (fim.isBefore(inicio)) throw new IllegalArgumentException("fim deve ser após início");
        this.inicio = inicio;
        this.fim = fim;
    }

    public boolean inclui(LocalDate data) {
        return !data.isBefore(inicio) && !data.isAfter(fim);
    }

    public LocalDate getInicio() { return inicio; }
    public LocalDate getFim() { return fim; }
}

public class ServiçoRelatorio {
    public List<Venda> buscarVendasPorPeriodo(Periodo periodo) { ... }
    public double somarReceitaPorPeriodo(Periodo periodo) { ... }
    public List<Fatura> faturasPorPeriodo(Periodo periodo) { ... }
}
```

**Por que este padrão:**
- `Periodo` valida seu próprio invariante (fim ≥ início) uma vez — chamadores não podem passar um período inválido
- O método `inclui()` captura lógica de consulta que antes era duplicada em cada chamador

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Criar o objeto mas não mover comportamento para ele**
```java
// Não aceito — Periodo ainda é apenas um recipiente de duas datas sem comportamento
public class Periodo {
    public LocalDate inicio;
    public LocalDate fim;
}
```

**Erro 2: Tornar o objeto mutável**
```java
// Não aceito — um objeto de parâmetro mutável pode ser alterado pelos chamadores após passá-lo
public class Periodo {
    public void setInicio(LocalDate inicio) { this.inicio = inicio; }
    public void setFim(LocalDate fim) { this.fim = fim; }
}
```

**Erro 3: Agrupar parâmetros não relacionados só para reduzir a contagem**
```java
// Não aceito — ContextoRequisicao agrupa idCliente, idSessao e locale
// sem razão além de serem três
```

---

## 7. Benefícios

- **Expressividade:** O grupo de parâmetros recebe um nome significativo no domínio
- **Validação:** O objeto valida seus próprios invariantes uma vez, no momento da construção
- **Migração de comportamento:** Lógica que trabalha com o grupo pode ser movida para dentro do objeto
