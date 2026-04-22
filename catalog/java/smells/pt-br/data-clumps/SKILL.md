# SKILL: Detectando e Refatorando Data Clumps — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/data-clumps

---

## 1. O que é Data Clumps

Partes diferentes do código contêm grupos idênticos de variáveis (um "clump") — os mesmos campos em múltiplas classes, ou os mesmos parâmetros aparecendo juntos em muitas assinaturas de método. Se você removesse um item do grupo e os demais perdessem sentido, você tem um data clump.

**Por que isso acontece:**
- O relacionamento entre os itens de dados nunca foi formalizado como uma classe
- Copy-paste propagou o grupo por lugares não relacionados
- O conceito existia informalmente na cabeça dos desenvolvedores, mas não no código

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Três ou mais campos que aparecem juntos em múltiplas classes
- [ ] Os mesmos 2–3 parâmetros consistentemente passados juntos para métodos
- [ ] Variáveis como `dataInicio`/`dataFim`, `latitude`/`longitude`, `rua`/`cidade`/`cep` aparecendo como primitivos separados
- [ ] Você precisa atualizar o mesmo grupo de campos em vários lugares para uma única mudança lógica

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                      | Técnica recomendada           |
|----------------------------------------------------------|-------------------------------|
| Clump aparece como campos em uma classe                  | Extract Class                 |
| Clump aparece em listas de parâmetros de métodos         | Introduce Parameter Object    |
| Um método recebe um objeto mas usa apenas parte dele     | Preserve Whole Object         |

**Teste chave:** remova um item do clump. Se os outros perderem significado, o grupo merece sua própria classe.

---

## 4. Exemplo

**ANTES — não aceito:**
```java
public class Pedido {
    private String rua;
    private String cidade;
    private String cep;
    private String pais;
    private String nomeCliente;
    private String emailCliente;
    // ...
}

public class Fatura {
    private String rua;      // mesmo clump novamente
    private String cidade;
    private String cep;
    private String pais;
    // ...
}

public void enviar(String rua, String cidade, String cep, String pais) { ... }
```

**DEPOIS — esperado:**
```java
public class Endereco {
    private final String rua;
    private final String cidade;
    private final String cep;
    private final String pais;

    public Endereco(String rua, String cidade, String cep, String pais) {
        this.rua = rua;
        this.cidade = cidade;
        this.cep = cep;
        this.pais = pais;
    }
    // getters, equals, hashCode
}

public class Pedido {
    private Endereco enderecoEntrega;
    private String nomeCliente;
    private String emailCliente;
}

public class Fatura {
    private Endereco enderecoCobranca;
}

public void enviar(Endereco destino) { ... }
```

**Por que este padrão:**
- `Endereco` é um conceito nomeado — sua validação, formatação e comparação agora vivem em um único lugar
- Toda classe que possui um endereço se beneficia de qualquer melhoria em `Endereco`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Agrupar o clump em um Map ou contêiner genérico**
```java
// Não aceito — perde segurança de tipos e intenção
Map<String, String> endereco = new HashMap<>();
endereco.put("rua", "Rua Principal");
```

**Erro 2: Criar a classe mas manter os campos separados antigos também**
```java
// Não aceito — agora há duas representações dos mesmos dados
private Endereco enderecoEntrega;
private String rua;    // duplicado
private String cidade; // duplicado
```

**Erro 3: Agrupar dados sem coesão só porque aparecem juntos**
```java
// Não aceito — PreferenciasCliente agrupa campos não relacionados em uma classe
// só porque apareceram juntos na mesma assinatura de método
```

---

## 6. Benefícios

- **Fonte única da verdade:** A lógica do clump (validação, formatação) é centralizada
- **Legibilidade:** `pedido.getEnderecoEntrega()` é mais expressivo do que quatro campos separados
- **Extensibilidade:** Adicionar um novo campo de endereço requer mudança apenas na classe `Endereco`
