# TÉCNICA: Remove Middle Man — Java

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
3. Atualize cada chamador para chamar o delegado diretamente (através do acessor que o fornece)
4. Remova cada método de delegação do middle man
5. Se o middle man agora não tem comportamento restante, delete-o ou mescle-o com o chamador
6. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
// Pessoa tornou-se um middle man puro — apenas encaminha para Departamento
public class Pessoa {
    private Departamento departamento;

    public Pessoa getGerente()              { return departamento.getGerente(); }
    public List<Pessoa> getFuncionarios()   { return departamento.getFuncionarios(); }
    public String getNomeDepartamento()     { return departamento.getNome(); }
    public int getOrcamentoDepartamento()   { return departamento.getOrcamento(); }
    public Localizacao getLocalizacaoDepartamento() { return departamento.getLocalizacao(); }
}
```

**DEPOIS — esperado:**
```java
public class Pessoa {
    private Departamento departamento;

    public Departamento getDepartamento() { return departamento; } // expõe o delegado diretamente
    // Todos os métodos pass-through removidos
}

// Chamadores acessam Departamento diretamente onde precisam
Departamento dept = pessoa.getDepartamento();
Pessoa gerente = dept.getGerente();
List<Pessoa> equipe = dept.getFuncionarios();
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Remover o middle man mas expor objetos internos que deveriam ser escondidos**
```java
// Não aceito — se Departamento é um detalhe interno, expô-lo quebra o encapsulamento
// Remova o middle man apenas se acesso direto for apropriado para a arquitetura
```

**Erro 2: Remover métodos de delegação que têm lógica real neles**
```java
// Não aceito — isso não é delegação pura, tem uma guarda
public Pessoa getGerente() {
    if (departamento == null) return null;  // lógica real — não remova
    return departamento.getGerente();
}
```

**Erro 3: Aplicar Remove Middle Man quando Hide Delegate é na verdade o correto**
```java
// Não aceito — se clientes não deveriam conhecer Departamento de forma alguma,
// Hide Delegate é a escolha certa; Remove Middle Man vai na direção oposta
```

---

## 7. Benefícios

- **Simplicidade:** Remove indireção desnecessária
- **Transparência:** Chamadores têm um caminho direto para o que precisam
- **Manutenibilidade:** Adicionar recursos ao delegado não requer mais tocar o middle man
