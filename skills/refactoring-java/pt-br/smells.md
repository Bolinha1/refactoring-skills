# Code Smells — Java

Identifique qual smell está presente e escolha uma técnica em `techniques.md`.
Para detalhes completos sobre um smell, leia `details/smells/{name}.md`.

---

## Alternative Classes With Different Interfaces

Duas ou mais classes executam funções similares ou idênticas, mas possuem nomes e assinaturas de métodos diferentes. Como as interfaces diferem, o código cliente não pode tratá-las de forma intercambiável e precisa saber com qual classe concreta está lidando.

→ `details/smells/alternative-classes-with-different-interfaces.md`

---

## Comments

Um método está cheio de comentários explicativos porque o código em si não é claro o suficiente para ser compreendido sem eles. O comentário é um sintoma: ele marca um lugar onde o código deveria falar por si mesmo, mas não consegue. **Importante:** Nem todo comentário é um smell. Comentários que explicam *por que* uma decisão não óbvia foi tomada são valiosos. Comentários que explicam *o que* o código faz são o smell — eles deveriam ser substituídos por nomes melhores e métodos menores.

→ `details/smells/comments.md`

---

## Data Class

Uma classe que contém apenas campos, getters e setters — mas nenhum comportamento real. Toda a lógica que opera sobre esses dados vive em outras classes. A classe é essencialmente um contêiner de dados passivo e age como um registro sem responsabilidade.

→ `details/smells/data-class.md`

---

## Data Clumps

Partes diferentes do código contêm grupos idênticos de variáveis (um "clump") — os mesmos campos em múltiplas classes, ou os mesmos parâmetros aparecendo juntos em muitas assinaturas de método. Se você removesse um item do grupo e os demais perdessem sentido, você tem um data clump.

→ `details/smells/data-clumps.md`

---

## Dead Code

Variáveis, parâmetros, campos, métodos ou classes inteiras que não são mais usados em lugar nenhum. O dead code polui a base de código, engana futuros leitores fazendo-os pensar que é relevante, e ainda precisa ser lido e compreendido mesmo não fazendo nada.

→ `details/smells/dead-code.md`

---

## Divergent Change

Uma única classe é alterada por muitas razões diferentes. Quando você se encontra modificando a mesma classe toda vez que uma nova regra de negócio, uma nova fonte de dados ou uma nova funcionalidade de preocupação não relacionada é adicionada, essa classe tem divergent change. O inverso do Shotgun Surgery: uma classe, muitas razões para mudar.

→ `details/smells/divergent-change.md`

---

## Duplicate Code

Dois ou mais fragmentos de código são quase idênticos ou estruturalmente similares em partes diferentes da base de código. Qualquer mudança na lógica precisa ser replicada em cada cópia, e esquecer uma delas cria um bug silencioso.

→ `details/smells/duplicate-code.md`

---

## Feature Envy

Um método acessa os dados de outro objeto mais do que acessa os dados da sua própria classe. O método tem "inveja" da outra classe — parece querer viver lá.

→ `details/smells/feature-envy.md`

---

## Inappropriate Intimacy

Uma classe acessa os campos privados, coleções internas ou detalhes de implementação de outra classe de forma excessiva. As duas classes estão fortemente acopladas de um modo que dificulta a mudança de uma sem afetar a outra.

→ `details/smells/inappropriate-intimacy.md`

---

## Incomplete Library Class

Uma biblioteca ou classe de framework não possui um método que você precisa e, como não é possível modificar a biblioteca, você acaba colocando o comportamento ausente em uma classe utilitária, um helper estático ou uma subclasse. A lógica que pertence à biblioteca vaza para seu próprio código.

→ `details/smells/incomplete-library-class.md`

---

## Large Class

Uma classe que contém muitos campos, métodos ou linhas de código. Quando uma classe tenta fazer coisas demais, ela acumula responsabilidades que deveriam estar distribuídas.

→ `details/smells/large-class.md`

---

## Lazy Class

Uma classe que faz tão pouco que não vale o esforço de entendê-la e mantê-la. Pode ter sido útil no passado, existir em antecipação a crescimento futuro, ou ser a casca restante de uma refatoração que extraiu a maior parte de sua lógica.

→ `details/smells/lazy-class.md`

---

## Long Method

Um método que contém linhas de código em excesso. Como regra geral: qualquer método com mais de 10 linhas já merece atenção.

→ `details/smells/long-method.md`

---

## Long Parameter List

Um método tem parâmetros demais — tipicamente mais de três ou quatro. Listas longas de parâmetros são difíceis de entender, fáceis de confundir e dolorosas de chamar. Elas frequentemente indicam que dados relacionados deveriam ser agrupados em um objeto.

→ `details/smells/long-parameter-list.md`

---

## Message Chains

Um cliente pede a um objeto outro objeto, que pede a outro objeto ainda, formando uma cadeia: `a.getB().getC().getD().fazerAlgo()`. O cliente está acoplado a toda a cadeia de intermediários. Se qualquer passo mudar sua estrutura, o cliente quebra.

→ `details/smells/message-chains.md`

---

## Middle Man

Uma classe cujo único propósito é delegar cada chamada a outra classe. Ela adiciona uma camada de indireção sem agregar nenhum valor. Este é o inverso de Feature Envy: em vez de uma classe que alcança outras, esta é uma classe pela qual outros alcançam desnecessariamente.

→ `details/smells/middle-man.md`

---

## Parallel Inheritance Hierarchies

Toda vez que você adiciona uma subclasse a uma hierarquia de classes, é forçado a adicionar uma subclasse correspondente a outra hierarquia. As duas árvores espelham uma à outra: adicionar `UsuarioPremium` em uma árvore significa adicionar `RelatorioUsuarioPremium` em outra.

→ `details/smells/parallel-inheritance-hierarchies.md`

---

## Primitive Obsession

Uso excessivo de tipos primitivos (`String`, `int`, `double`, `boolean`) para representar conceitos de domínio que mereceriam classes próprias.

→ `details/smells/primitive-obsession.md`

---

## Refused Bequest

Uma subclasse herda métodos ou dados de sua classe pai mas não os utiliza ou os sobrescreve ativamente lançando exceções. O filho rejeita parte do que o pai lhe dá, o que viola o Princípio de Substituição de Liskov: uma subclasse deve ser utilizável onde quer que seu pai seja esperado.

→ `details/smells/refused-bequest.md`

---

## Shotgun Surgery

Uma única mudança lógica requer modificar muitas classes diferentes ao mesmo tempo. Fazer uma mudança conceitual "dispara" edições pela base de código como um tiro de espingarda — tocando muitos arquivos para o que deveria ser uma correção localizada. O inverso do Divergent Change: muitas classes, uma razão para mudar.

→ `details/smells/shotgun-surgery.md`

---

## Speculative Generality

Código escrito para lidar com requisitos que ainda não existem — e talvez nunca existam. Classes abstratas com apenas uma subclasse concreta, hooks que nunca são chamados, parâmetros que são sempre passados como `null` e interfaces com uma única implementação são todos sinais de que alguém construiu flexibilidade "por precaução".

→ `details/smells/speculative-generality.md`

---

## Switch Statements

Cadeias complexas de switch ou if/else que ramificam no tipo ou estado de um objeto. A mesma lógica de switch tende a ser duplicada pela base de código — quando um novo caso é adicionado, cada switch precisa ser encontrado e atualizado.

→ `details/smells/switch-statements.md`

---

## Temporary Field

Uma classe contém um campo que só é definido e usado em certas situações — durante um algoritmo ou chamada de método. No restante do tempo ele é `null`, vazio ou sem significado. Outros códigos que leem o campo não conseguem saber quando ele tem um valor válido e quando não tem.

→ `details/smells/temporary-field.md`

---
