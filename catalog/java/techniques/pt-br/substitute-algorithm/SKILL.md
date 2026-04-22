# TÉCNICA: Substitute Algorithm — Java

## Fonte
Baseado em: https://refactoring.guru/substitute-algorithm

---

## 1. Problema

Você quer substituir um algoritmo existente por um mais limpo, simples ou eficiente. O algoritmo funciona corretamente, mas é difícil de entender, difícil de estender ou pode agora ser substituído por um método de biblioteca.

---

## 2. Solução

Substitua o corpo do método que implementa o algoritmo antigo pelo novo algoritmo. Mantenha a assinatura do método igual para que os chamadores não sejam afetados.

---

## 3. Quando aplicar

- Existe um algoritmo mais simples que produz os mesmos resultados
- Uma biblioteca ou recurso da linguagem agora cobre o que você implementou manualmente
- O algoritmo atual é difícil de estender incrementalmente — é mais fácil reescrever do que modificar
- O algoritmo já foi simplificado ao máximo possível, mas ainda está obscuro

---

## 4. Passos de refatoração

1. Certifique-se de que o algoritmo existente possui testes abrangentes — você precisará deles para verificar a substituição
2. Escreva o novo algoritmo em um método separado (ou em um branch)
3. Execute os testes com o novo algoritmo; corrija quaisquer falhas
4. Quando os testes passarem, substitua o corpo do algoritmo antigo pelo novo
5. Execute os testes uma última vez
6. Remova qualquer código auxiliar que era necessário apenas pelo algoritmo antigo

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public String encontrarPessoa(List<String> pessoas) {
    for (String pessoa : pessoas) {
        if (pessoa.equals("Carlos")) return "Carlos";
        if (pessoa.equals("João")) return "João";
        if (pessoa.equals("Maria")) return "Maria";
    }
    return "";
}
```

**DEPOIS — esperado:**
```java
public String encontrarPessoa(List<String> pessoas) {
    List<String> candidatos = Arrays.asList("Carlos", "João", "Maria");
    for (String pessoa : pessoas) {
        if (candidatos.contains(pessoa)) return pessoa;
    }
    return "";
}
```

**Por que esse padrão:**
- O novo algoritmo é mais fácil de estender: para adicionar um novo candidato, adicione uma entrada à lista
- O algoritmo antigo exigia adicionar um novo bloco `if` para cada candidato

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Substituir sem uma suíte de testes**
```java
// Não aceito — sem testes, você não pode verificar se o novo algoritmo é equivalente;
// escreva os testes primeiro antes de substituir qualquer coisa
```

**Erro 2: Corrigir o algoritmo antigo incrementalmente ao invés de substituí-lo**
```java
// Não aceito — se o algoritmo é fundamentalmente falho, correções pontuais pioram a situação;
// substitua-o inteiramente
```

---

## 7. Benefícios

- **Simplicidade:** O novo algoritmo é mais fácil de ler e manter
- **Extensibilidade:** Algoritmos bem estruturados são mais fáceis de modificar
- **Aproveita bibliotecas:** Evita reinventar o que a biblioteca padrão já oferece
