# TÉCNICA: Remove Assignments to Parameters — Python

## Fonte
Baseado em: https://refactoring.guru/remove-assignments-to-parameters

---

## 1. Problema

Um parâmetro de método é reatribuído dentro do corpo do método. Isso esconde o valor original e torna o método mais difícil de entender. Em Python, todos os argumentos são passados por atribuição, então reatribuir um parâmetro rebinda o nome local — não afeta a variável do chamador — mas a confusão que isso cria ainda é um code smell.

---

## 2. Solução

Introduza uma variável local para armazenar o valor modificado. Use essa variável no lugar de reatribuir o parâmetro.

---

## 3. Quando aplicar

- Um parâmetro recebe um novo valor dentro do método
- A reatribuição não precisa ser visível ao chamador
- O valor original do parâmetro pode ser necessário mais adiante no método (ou a clareza melhora ao preservá-lo)

---

## 4. Passos de refatoração

1. Encontre a atribuição ao parâmetro
2. Declare uma nova variável local com um nome descritivo, inicializada com o valor do parâmetro (ou com o novo valor)
3. Substitua todos os usos do parâmetro após a atribuição pela nova variável local
4. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
def desconto(self, valor_entrada: int, quantidade: int) -> int:
    if valor_entrada > 50:
        valor_entrada -= 2    # reatribuindo o parâmetro
    if quantidade > 100:
        valor_entrada -= 1    # reatribuindo novamente
    return valor_entrada
```

**DEPOIS — esperado:**
```python
def desconto(self, valor_entrada: int, quantidade: int) -> int:
    resultado = valor_entrada
    if resultado > 50:
        resultado -= 2
    if quantidade > 100:
        resultado -= 1
    return resultado
```

**Por que esse padrão:**
- `valor_entrada` sempre se refere ao valor original
- `resultado` sinaliza claramente "este é o valor que estamos construindo para retornar"

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mutar o objeto referenciado pelo parâmetro**
```python
# Isso NÃO é atribuição a parâmetro — mutar o objeto É visível ao chamador
def adicionar_item(self, pedido, item) -> None:
    pedido.adicionar_item(item)  # muta o objeto Order; o chamador vê a mudança
```

**Erro 2: Manter a reatribuição do parâmetro e adicionar um comentário**
```python
# Não aceito — o comentário não resolve o problema de legibilidade
if valor_entrada > 50:
    valor_entrada -= 2  # modificando a entrada intencionalmente
```

---

## 7. Benefícios

- **Clareza:** O valor original do parâmetro é preservado e claramente separado do resultado calculado
- **Intenção:** O nome da variável de saída comunica o papel do valor final
- **Segurança:** Evita confundir a "entrada" com a "cópia de trabalho" do mesmo conceito
