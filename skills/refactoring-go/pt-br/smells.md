# Code Smells — Go

Identifique qual smell está presente e escolha uma técnica em `techniques.md`.
Para detalhes completos sobre um smell, leia `details/smells/{name}.md`.

---

## Alternative Classes With Different Interfaces

Duas ou mais structs (ou interfaces) fazem a mesma coisa mas possuem nomes de métodos ou assinaturas diferentes. Chamadores precisam tratar cada uma de forma especial mesmo que o comportamento seja equivalente, criando código duplicado e acoplamento desnecessário.

→ `details/smells/alternative-classes-with-different-interfaces.md`

---

## Comments

Comentários que explicam *o que* o código faz em vez de *por que* uma decisão foi tomada. Quando o código precisa de um comentário para ser entendido, geralmente significa que ele precisa ser refatorado para ser autoexplicativo. Comentários compensatórios mascaram código confuso em vez de corrigi-lo.

→ `details/smells/comments.md`

---

## Data Class

Uma struct que contém apenas campos e nenhum comportamento significativo. Outros pacotes ou structs manipulam seus dados diretamente em vez de delegar para ela. A struct é um recipiente passivo — existe apenas para carregar dados de um lado para o outro.

→ `details/smells/data-class.md`

---

## Data Clumps

Partes diferentes do código contêm grupos idênticos de campos — os mesmos campos em múltiplas structs, ou os mesmos parâmetros aparecendo juntos em muitas assinaturas de função. Se você removesse um item do grupo e os demais perdessem sentido, você tem um data clump.

→ `details/smells/data-clumps.md`

---

## Dead Code

Variáveis, funções, structs, imports ou blocos de código que não são mais usados por nenhuma parte do sistema. Esse código é mantido por medo ou por descuido, mas só polui o código-base, confunde os leitores e aumenta o custo de manutenção.

→ `details/smells/dead-code.md`

---

## Divergent Change

Uma única struct ou arquivo precisa ser modificada por razões diferentes e não relacionadas. Você abre o mesmo arquivo para corrigir um bug de banco de dados, para ajustar uma regra de negócio e para mudar o formato de resposta da API — tudo no mesmo lugar. Isso viola o Princípio da Responsabilidade Única (SRP).

→ `details/smells/divergent-change.md`

---

## Duplicate Code

Dois ou mais fragmentos de código são quase idênticos ou estruturalmente similares em partes diferentes da base de código. Qualquer mudança na lógica precisa ser replicada em cada cópia, e esquecer uma delas cria um bug silencioso.

→ `details/smells/duplicate-code.md`

---

## Feature Envy

Uma função ou método acessa os dados de outra struct mais do que acessa os dados de sua própria struct. O método tem "inveja" da outra struct — parece querer viver lá.

→ `details/smells/feature-envy.md`

---

## Inappropriate Intimacy

Duas structs ou pacotes conhecem detalhes internos um do outro em excesso — acessam campos não exportados via reflexão, dependem de comportamento interno indocumentado ou criam dependências circulares. Esse acoplamento bidirecional torna qualquer mudança em um perigosa para o outro.

→ `details/smells/inappropriate-intimacy.md`

---

## Incomplete Library Class

Uma biblioteca ou pacote externo não oferece a funcionalidade necessária, levando desenvolvedores a duplicar código, usar reflexão hacky ou espalhar workarounds pela base de código. Em vez de adaptar a biblioteca de forma estruturada, soluções pontuais se multiplicam.

→ `details/smells/incomplete-library-class.md`

---

## Large Class

Uma struct que acumula campos, métodos e responsabilidades demais. Em Go isso se manifesta como um arquivo com uma única struct enorme ou um conjunto de funções com o mesmo receiver que cobre domínios completamente diferentes.

→ `details/smells/large-class.md`

---

## Lazy Class

Uma struct, arquivo ou pacote que não faz o suficiente para justificar sua existência. Pode ser um wrapper trivial que apenas delega, uma struct criada para uma funcionalidade que nunca foi implementada, ou um pacote com um único arquivo de uma linha. Toda abstração tem um custo cognitivo — se ela não agrega valor, deve ser eliminada.

→ `details/smells/lazy-class.md`

---

## Long Method

Uma função ou método que contém linhas de código em excesso. Como regra geral: qualquer função com mais de 10 linhas já merece atenção.

→ `details/smells/long-method.md`

---

## Long Parameter List

Uma função ou método que recebe mais parâmetros do que é possível entender e lembrar facilmente. Mais de três ou quatro parâmetros já tornam a assinatura difícil de usar corretamente.

→ `details/smells/long-parameter-list.md`

---

## Message Chains

Uma cadeia de chamadas encadeadas onde o código precisa navegar por vários objetos para obter o que precisa: `a.GetB().GetC().GetD()`. Cada chamada cria dependência do código chamador em toda a estrutura interna da cadeia.

→ `details/smells/message-chains.md`

---

## Middle Man

Uma struct ou função que existe apenas para delegar chamadas a outra struct sem adicionar nenhum valor. Se a maior parte dos métodos de uma struct apenas repassam chamadas para outra, ela é um intermediário desnecessário.

→ `details/smells/middle-man.md`

---

## Parallel Inheritance Hierarchies

Toda vez que você cria um novo tipo em um conjunto de structs/interfaces, você precisa criar um tipo correspondente em outro conjunto. As duas hierarquias crescem em paralelo e estão acopladas: uma não pode evoluir sem a outra.

→ `details/smells/parallel-inheritance-hierarchies.md`

---

## Primitive Obsession

Uso excessivo de tipos primitivos (`string`, `int`, `float64`, `bool`) para representar conceitos de domínio que mereceriam tipos próprios. Em Go, isso também inclui usar `string` como tipo enumerado sem restrição de valores válidos.

→ `details/smells/primitive-obsession.md`

---

## Refused Bequest

Uma struct embute outra (embedding) ou implementa uma interface, mas não usa a maioria dos métodos promovidos — ou os sobrescreve com `panic`. Em Go, herança é substituída por embedding e interfaces, portanto esse smell se manifesta como abuso de embedding:

→ `details/smells/refused-bequest.md`

---

## Shotgun Surgery

Uma única mudança lógica exige modificar muitos pacotes ou structs diferentes ao mesmo tempo. Fazer uma mudança conceitual "dispara" edições por todo o código como uma escopeta — tocando muitos arquivos para o que deveria ser uma correção localizada.

→ `details/smells/shotgun-surgery.md`

---

## Speculative Generality

Código escrito para lidar com requisitos que ainda não existem — e talvez nunca existam. Interfaces com uma única implementação, funções com parâmetros que sempre recebem `nil` ou zero, e structs que existem apenas como "pontos de extensão futuros"

→ `details/smells/speculative-generality.md`

---

## Switch Statements

Cadeias complexas de `switch` ou `if/else` que ramificam com base no tipo ou estado de um valor. A mesma lógica de switch tende a ser duplicada por todo o código — quando um novo caso é adicionado, cada switch deve ser encontrado e atualizado.

→ `details/smells/switch-statements.md`

---

## Temporary Field

Uma struct contém um campo que só é definido e usado durante uma chamada de método ou algoritmo. No restante do tempo, ele tem valor zero ou não tem significado. Outro código que lê o campo não consegue saber quando ele contém um valor válido e quando não.

→ `details/smells/temporary-field.md`

---
