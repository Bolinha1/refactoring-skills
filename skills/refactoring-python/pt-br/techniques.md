# Técnicas de Refatoração — Python

Escolha a técnica adequada ao smell identificado em `smells.md`.
Para detalhes completos sobre uma técnica, leia `details/techniques/{name}.md`.

---

## Decompose Conditional

Uma expressão condicional complexa (e o código em seus branches) torna difícil entender o que está sendo testado e o que acontece em cada caso.

→ `details/techniques/decompose-conditional.md`

---

## Extract Class

Uma classe faz o trabalho de duas. Um subconjunto de seus atributos e métodos forma um conceito coeso que faria sentido como sua própria classe.

→ `details/techniques/extract-class.md`

---

## Extract Method

Você tem um fragmento de código que pode ser agrupado em uma unidade com sentido próprio, mas ele está embutido em um método maior junto com outras responsabilidades.

→ `details/techniques/extract-method.md`

---

## Extract Variable

Uma expressão complexa é difícil de entender de relance. Resultados intermediários são calculados inline, tornando difícil ver o que cada parte significa ou depurar o cálculo.

→ `details/techniques/extract-variable.md`

---

## Hide Delegate

Um cliente acessa um objeto delegado navegando por uma cadeia através do objeto servidor: `cliente → servidor → delegado`. Quando o delegado muda, o cliente também precisa mudar.

→ `details/techniques/hide-delegate.md`

---

## Inline Class

Uma classe não faz mais o suficiente para justificar sua existência. Ela tem responsabilidades demais poucas e é apenas uma indireção desnecessária.

→ `details/techniques/inline-class.md`

---

## Inline Method

O corpo de um método é tão óbvio quanto seu nome, ou o método é usado apenas uma vez e a indireção não agrega valor. Manter um wrapper trivial em torno de uma única expressão torna o código mais difícil de seguir, não mais fácil.

→ `details/techniques/inline-method.md`

---

## Inline Temp

Uma variável local armazena o resultado de uma expressão simples, e o nome da variável não acrescenta nenhuma informação que a própria expressão já não comunique claramente.

→ `details/techniques/inline-temp.md`

---

## Introduce Parameter Object

Vários parâmetros que naturalmente pertencem juntos são sempre passados como grupo para as mesmas funções. O grupo representa um conceito ainda não formalizado na base de código.

→ `details/techniques/introduce-parameter-object.md`

---

## Move Field

Um atributo é usado mais por outra classe do que pela própria classe — outras classes leem ou escrevem nele com mais frequência do que a classe que o contém.

→ `details/techniques/move-field.md`

---

## Move Method

Um método é mais usado por outra classe do que pela própria classe onde está definido. Isso cria acoplamento desnecessário e viola o princípio de coesão.

→ `details/techniques/move-method.md`

---

## Remove Assignments To Parameters

Um parâmetro de método é reatribuído dentro do corpo do método. Isso esconde o valor original e torna o método mais difícil de entender. Em Python, todos os argumentos são passados por atribuição, então reatribuir um parâmetro rebinda o nome local — não afeta a variável do chamador — mas a confusão que isso cria ainda é um code smell.

→ `details/techniques/remove-assignments-to-parameters.md`

---

## Remove Middle Man

Uma classe tem muitas propriedades ou métodos de delegação simples que não fazem nada além de encaminhar chamadas para outro objeto. A classe é um Middle Man — existe apenas para passar mensagens adiante.

→ `details/techniques/remove-middle-man.md`

---

## Replace Conditional With Polymorphism

Você tem uma cadeia de `if/elif/else` que executa comportamentos diferentes dependendo do tipo do objeto ou de uma propriedade que simula um tipo.

→ `details/techniques/replace-conditional-with-polymorphism.md`

---

## Replace Method With Method Object

Um método é tão grande e complexo que extrair submétodos fica difícil porque todos compartilhariam muitas variáveis locais. As variáveis locais estão tão entrelaçadas que não podem ser facilmente passadas como parâmetros.

→ `details/techniques/replace-method-with-method-object.md`

---

## Replace Nested Conditional With Guard Clauses

Uma função tem condicionais profundamente aninhados que obscurecem o caminho feliz principal. Leitores precisam rastrear mentalmente cada nível de aninhamento para entender o que a função normalmente faz.

→ `details/techniques/replace-nested-conditional-with-guard-clauses.md`

---

## Replace Temp With Query

Uma variável local armazena o resultado de uma expressão. Esse valor é usado mais adiante no método, mas a variável impede que a expressão seja reutilizada em outros métodos ou sobrescrita em subclasses. O método cresce porque carrega estado que poderia ser movido para um método de consulta reutilizável.

→ `details/techniques/replace-temp-with-query.md`

---

## Split Temporary Variable

Uma variável local é atribuída mais de uma vez, servindo a propósitos diferentes em momentos distintos do método. Reutilizar o mesmo nome para conceitos diferentes torna o código difícil de acompanhar.

→ `details/techniques/split-temporary-variable.md`

---

## Substitute Algorithm

Você quer substituir um algoritmo existente por um mais limpo, simples ou eficiente. O algoritmo funciona corretamente, mas é difícil de entender, difícil de estender ou pode agora ser substituído por uma função nativa do Python ou da biblioteca padrão.

→ `details/techniques/substitute-algorithm.md`

---
