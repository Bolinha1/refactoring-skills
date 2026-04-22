# TÉCNICA: Inline Class — Java

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
   - Atualize a classe alvo para delegar à classe inline (para uma transição segura, passo a passo)
3. Atualize todos os chamadores para usar a classe alvo diretamente
4. Remova a delegação da classe alvo (métodos agora funcionam nativamente)
5. Delete a classe que foi inlined
6. Compile e execute os testes após cada passo

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public class NumeroTelefone {
    private String numero;

    public String getNumero() { return numero; }
    public void setNumero(String numero) { this.numero = numero; }
}

public class Pessoa {
    private String nome;
    private NumeroTelefone telefone;

    public String getNumeroTelefone() {
        return telefone.getNumero(); // NumeroTelefone é apenas um wrapper
    }
}
```

**DEPOIS — esperado:**
```java
public class Pessoa {
    private String nome;
    private String numeroTelefone; // inlined diretamente

    public String getNumeroTelefone() {
        return numeroTelefone;
    }

    public void setNumeroTelefone(String numero) {
        this.numeroTelefone = numero;
    }
}
```

**Caso de uso: consolidando Shotgun Surgery**
```java
// ANTES — quatro classes de serviço minúsculas foram divididas de forma muito agressiva
public class BuscadorPedido  { Pedido buscar(long id) { ... } }
public class SalvadorPedido  { void salvar(Pedido p) { ... } }
public class AtualizadorPedido { void atualizar(Pedido p) { ... } }
public class DeletadorPedido { void deletar(long id) { ... } }

// DEPOIS — fazer inline de todos em um único repositório coeso
public class RepositorioPedido {
    public Pedido buscar(long id) { ... }
    public void salvar(Pedido p) { ... }
    public void atualizar(Pedido p) { ... }
    public void deletar(long id) { ... }
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Fazer inline de uma classe que ainda tem um conceito significativo**
```java
// Não aceito — Dinheiro tem comportamento (aritmética, formatação) que pertence junto
// Não faça inline de Dinheiro em Pedido só porque tem poucos campos
```

**Erro 2: Fazer inline sem remover a classe original**
```java
// Não aceito — tanto Pessoa.numeroTelefone quanto NumeroTelefone existem simultaneamente
public class Pessoa {
    private String numeroTelefone;  // cópia inlined
    private NumeroTelefone telefone; // ainda aqui — cria confusão
}
```

**Erro 3: Fazer inline de uma classe usada em muitos lugares**
```java
// Não aceito — se NumeroTelefone é usado por Pessoa, Funcionario e Contato,
// fazer inline exigiria duplicar a lógica em três lugares
```

---

## 7. Benefícios

- **Simplicidade:** Remove indireção desnecessária da base de código
- **Legibilidade:** Menos classes para navegar quando a abstração não adiciona valor
- **Preparação:** Frequentemente precede Extract Class — fazer inline primeiro, depois re-extrair corretamente
