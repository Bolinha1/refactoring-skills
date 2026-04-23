# TÉCNICA: Remove Assignments to Parameters — Java

## Fonte
Baseado em: https://refactoring.guru/remove-assignments-to-parameters

---

## 1. Problema

Um parâmetro de método é reatribuído dentro do corpo do método. Isso é confuso porque esconde o valor original, dificulta o entendimento do método e pode enganar quem lê o código: objetos são passados por referência em Java, portanto atribuir um novo objeto ao parâmetro NÃO afeta a variável do chamador, mas mutar o estado do objeto SIM.

---

## 2. Solução

Introduza uma variável local para armazenar o valor modificado. Use essa variável no lugar de reatribuir o parâmetro.

---

## 3. Quando aplicar

- Um parâmetro recebe um novo valor dentro do método
- A reatribuição não precisa ser visível ao chamador (semântica de passagem por valor)
- O valor original do parâmetro é necessário mais adiante no método (ou deve ser preservado para legibilidade)

---

## 4. Passos de refatoração

1. Encontre a atribuição ao parâmetro
2. Declare uma nova variável local com um nome descritivo, inicializada com o valor do parâmetro (ou com o novo valor)
3. Substitua todos os usos do parâmetro após a atribuição pela nova variável local
4. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public int desconto(int valorEntrada, int quantidade) {
    if (valorEntrada > 50) valorEntrada -= 2;    // reatribuindo o parâmetro
    if (quantidade > 100) valorEntrada -= 1;      // reatribuindo novamente
    return valorEntrada;
}
```

**DEPOIS — esperado:**
```java
public int desconto(int valorEntrada, int quantidade) {
    int resultado = valorEntrada;
    if (resultado > 50) resultado -= 2;
    if (quantidade > 100) resultado -= 1;
    return resultado;
}
```

**Por que esse padrão:**
- `valorEntrada` sempre se refere ao valor original — o método não pode passar acidentalmente um valor modificado de volta
- `resultado` sinaliza claramente "este é o valor que estamos construindo para retornar"

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mutar o estado do objeto referenciado pelo parâmetro**
```java
// Isso NÃO é atribuição a parâmetro — isso muta o objeto referenciado e É visível ao chamador
public void adicionarItem(Pedido pedido, Item item) {
    pedido.adicionarItem(item); // muta o objeto Pedido; o chamador vê a mudança
}
```

**Erro 2: Manter a reatribuição do parâmetro e adicionar um comentário**
```java
// Não aceito — o comentário não resolve o problema de legibilidade
if (valorEntrada > 50) valorEntrada -= 2; // modificando a entrada intencionalmente
```

---

## 7. Benefícios

- **Clareza:** O valor original do parâmetro é preservado e claramente separado do resultado calculado
- **Intenção:** O nome da variável de saída comunica o papel do valor final
- **Segurança:** Evita confundir a "entrada" com a "cópia de trabalho" do mesmo conceito
