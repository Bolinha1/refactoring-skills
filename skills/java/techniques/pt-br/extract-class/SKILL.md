# TÉCNICA: Extract Class — Java

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
6. Decida sobre a visibilidade da nova classe:
   - Torne-a pública se pode ser usada independentemente ou reutilizada
   - Mantenha como package-private se é um detalhe interno
7. Compile e execute os testes após cada movimentação

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public class Pessoa {
    private String nome;
    private String codigoAreaEscritorio;
    private String numeroEscritorio;

    public String getNumeroTelefone() {
        return "(" + codigoAreaEscritorio + ") " + numeroEscritorio;
    }
}
```

**DEPOIS — esperado:**
```java
public class NumeroTelefone {
    private final String codigoArea;
    private final String numero;

    public NumeroTelefone(String codigoArea, String numero) {
        this.codigoArea = codigoArea;
        this.numero = numero;
    }

    public String formatar() {
        return "(" + codigoArea + ") " + numero;
    }
}

public class Pessoa {
    private String nome;
    private NumeroTelefone telefoneEscritorio;

    public String getNumeroTelefone() {
        return telefoneEscritorio.formatar();
    }
}
```

**Variante — extraindo um data clump:**
```java
// ANTES — campos de endereço espalhados em Pessoa e Fatura
public class Pessoa {
    private String rua;
    private String cidade;
    private String cep;
}

// DEPOIS — Endereco é um conceito independente e reutilizável
public class Endereco {
    private final String rua;
    private final String cidade;
    private final String cep;
    // construtor, getters, validação, formatação
}

public class Pessoa {
    private Endereco enderecoResidencial;
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair uma classe sem coesão**
```java
// Não aceito — DadosPessoa agrupa campos sem relacionamento natural
public class DadosPessoa {
    private String nome;
    private String codigoEscritorio;
    private double salario;
    private Date dataContratacao;
}
```

**Erro 2: Mover métodos sem mover os campos que eles precisam**
```java
// Não aceito — NumeroTelefone.formatar() ainda lê codigoArea de Pessoa
public class NumeroTelefone {
    public String formatar(String codigoArea, String numero) { ... } // ainda acoplado
}
```

**Erro 3: Extrair mas deixar os campos originais no lugar**
```java
// Não aceito — agora há duas fontes da verdade para dados de telefone
public class Pessoa {
    private NumeroTelefone telefone;      // novo
    private String codigoAreaEscritorio;  // antigo — deve ser deletado
    private String numeroEscritorio;      // antigo — deve ser deletado
}
```

---

## 7. Benefícios

- **Responsabilidade Única:** Cada classe tem um conceito claro para representar
- **Reutilização:** A classe extraída pode ser usada por outras classes independentemente
- **Encapsulamento:** Lógica de validação e formatação do conceito vive em um único lugar
