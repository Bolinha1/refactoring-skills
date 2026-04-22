# Técnicas de Refatoração — Go

Escolha a técnica adequada ao smell identificado em `smells.md`.
Para detalhes completos sobre uma técnica, leia `details/techniques/{name}.md`.

---

## Decompose Conditional

Uma expressão condicional complexa (e o código em seus branches) torna difícil entender o que está sendo testado e o que acontece em cada caso. Ler a condição exige conhecimento de regras de negócio que não estão nomeadas em lugar algum.

→ `details/techniques/decompose-conditional.md`

---

## Extract Class

Uma struct faz o trabalho de duas. Um subconjunto de seus campos e métodos forma um conceito coeso que faria sentido como sua própria struct. Em Go não há herança, mas composição e interfaces permitem extrair responsabilidades com clareza.

→ `details/techniques/extract-class.md`

---

## Extract Method

Você tem um fragmento de código que pode ser agrupado em uma unidade com sentido próprio, mas ele está embutido em uma função maior junto com outras responsabilidades. O resultado é uma função longa difícil de ler e testar.

→ `details/techniques/extract-method.md`

---

## Extract Variable

Uma expressão complexa ou difícil de ler está embutida diretamente em uma condição, retorno ou cálculo. Sem um nome, o leitor precisa decodificar a expressão para entender a intenção do código.

→ `details/techniques/extract-variable.md`

---

## Hide Delegate

O cliente precisa conhecer o objeto intermediário (delegado) para obter um serviço. Isso cria acoplamento: se o delegado mudar, o cliente precisa ser atualizado. O cliente navega por uma cadeia de objetos para chegar ao que precisa.

→ `details/techniques/hide-delegate.md`

---

## Inline Class

Uma struct não faz mais nada de útil por conta própria. Ela contém poucos campos e métodos, e sua responsabilidade foi absorvida por outras structs ao longo de refatorações anteriores. Manter essa struct em separado adiciona complexidade desnecessária.

→ `details/techniques/inline-class.md`

---

## Inline Method

O corpo de uma função é tão óbvio quanto seu próprio nome. A função não agrega legibilidade — ela apenas adiciona uma camada de indireção sem valor, tornando o código mais difícil de navegar.

→ `details/techniques/inline-method.md`

---

## Inline Temp

Uma variável temporária armazena o resultado de uma expressão simples e é usada apenas uma vez. A variável não agrega legibilidade — o nome dela não é mais claro do que a própria expressão, e ela ocupa espaço desnecessário no código.

→ `details/techniques/inline-temp.md`

---

## Introduce Parameter Object

Um grupo de parâmetros sempre aparece junto em várias funções. Passar esses parâmetros individualmente cria ruído nas assinaturas e torna difícil adicionar um novo campo sem alterar todos os chamadores.

→ `details/techniques/introduce-parameter-object.md`

---

## Move Field

Um campo de uma struct é mais usado por outra struct do que pela que o contém. Isso indica que o campo está na struct errada, criando acoplamento desnecessário entre elas.

→ `details/techniques/move-field.md`

---

## Move Method

Um método usa mais dados e comportamentos de outra struct do que da struct em que está definido. Isso cria Feature Envy e acopla a struct errada a detalhes que não lhe pertencem.

→ `details/techniques/move-method.md`

---

## Remove Assignments To Parameters

Em Go, parâmetros são passados por valor — reatribuir um parâmetro dentro de uma função não afeta o chamador, mas confunde o leitor que pode pensar que há um efeito externo. Além disso, reatribuir parâmetros mistura dois conceitos distintos: o valor de entrada e o valor de trabalho.

→ `details/techniques/remove-assignments-to-parameters.md`

---

## Remove Middle Man

Uma struct tem muitos métodos de encaminhamento que apenas delegam chamadas a outro objeto. A quantidade de delegações triviais supera o benefício do encapsulamento — a struct virou um intermediário inútil que apenas adiciona indireção.

→ `details/techniques/remove-middle-man.md`

---

## Replace Conditional With Polymorphism

Um switch ou cadeia de if/else seleciona comportamentos diferentes com base no tipo de um objeto. Toda vez que um novo tipo é adicionado, o switch precisa ser atualizado em vários lugares do código. Em Go, polimorfismo é obtido via interfaces, não herança.

→ `details/techniques/replace-conditional-with-polymorphism.md`

---

## Replace Method With Method Object

Um método é tão grande e complexo que extrair sub-métodos fica difícil porque todos eles compartilhariam muitas variáveis locais. As variáveis locais estão muito interligadas para serem facilmente passadas como parâmetros.

→ `details/techniques/replace-method-with-method-object.md`

---

## Replace Nested Conditional With Guard Clauses

Um método tem condicionais aninhadas que tornam o fluxo normal difícil de identificar. O código "principal" fica enterrado dentro de vários níveis de indentação, obscurecendo a intenção.

→ `details/techniques/replace-nested-conditional-with-guard-clauses.md`

---

## Replace Temp With Query

Você usa uma variável temporária para armazenar o resultado de uma expressão e depois referencia essa variável no mesmo método. A expressão é recalculada toda vez que o método é executado, mas nunca é exposta como lógica reutilizável.

→ `details/techniques/replace-temp-with-query.md`

---

## Split Temporary Variable

Uma variável temporária é atribuída mais de uma vez dentro do mesmo método, mas cada atribuição serve a um propósito diferente. Reutilizar o mesmo nome para valores não relacionados torna o código difícil de seguir e impede que o compilador detecte

→ `details/techniques/split-temporary-variable.md`

---

## Substitute Algorithm

Você quer substituir um algoritmo existente por um mais limpo, simples ou eficiente. O algoritmo funciona corretamente, mas é difícil de entender, difícil de estender ou pode agora ser substituído por uma função da biblioteca padrão do Go.

→ `details/techniques/substitute-algorithm.md`

---
