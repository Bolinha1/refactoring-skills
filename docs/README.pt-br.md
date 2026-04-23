# Refactoring Skills

Uma base de conhecimento estruturada para identificar code smells e aplicar técnicas de refatoração,
organizada por linguagem de programação e idioma. Baseada no catálogo do [refactoring.guru](https://refactoring.guru/pt-br/refactoring).

---

## Instalar via skills CLI

Use com qualquer agente de IA que suporte o [skills.sh](https://skills.sh):

```bash
# Instalar todas as linguagens
npx skills add Bolinha1/refactoring-skills

# Instalar uma linguagem específica
npx skills add Bolinha1/refactoring-skills --skill refactoring-java
npx skills add Bolinha1/refactoring-skills --skill refactoring-go
npx skills add Bolinha1/refactoring-skills --skill refactoring-python
npx skills add Bolinha1/refactoring-skills --skill refactoring-php

# Instalar apenas os templates de prompt
npx skills add Bolinha1/refactoring-skills --skill refactoring-templates
```

Após instalado, o agente identifica automaticamente os smells e aplica a técnica de refatoração correta ao revisar código.

---

## Estrutura do repositório

```text
skills/
├── refactoring-go/
│   ├── SKILL.md                     ← entry point (com suporte a locale)
│   ├── en/
│   │   ├── smells.md                ← índice dos 23 smells
│   │   ├── techniques.md            ← índice das 19 técnicas
│   │   └── details/
│   │       ├── smells/              ← conteúdo completo por smell
│   │       └── techniques/          ← conteúdo completo por técnica
│   └── pt-br/                       ← mesma estrutura em português
├── refactoring-java/                ← mesma estrutura
├── refactoring-python/              ← mesma estrutura
├── refactoring-php/                 ← mesma estrutura
└── refactoring-templates/           ← prompts de code review e refatoração
```

Cada `SKILL.md` segue um formato consistente: definição do problema, quando aplicar, passo a passo da refatoração,
exemplos de código antes/depois, exemplos negativos (o que NÃO fazer) e benefícios.

---

## Clonar o repositório

### Opção 1 — Clonar tudo (mais simples)

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
```

### Opção 2 — Clonar apenas a sua linguagem (recomendado)

Evita baixar pacotes que você não vai usar:

```bash
git clone --filter=blob:none --sparse https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
git sparse-checkout set skills/refactoring-java   # substitua por: refactoring-go, refactoring-python ou refactoring-php
```

### Opção 3 — Clonar tudo e remover o que não precisa

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
rm -rf skills/refactoring-go skills/refactoring-python skills/refactoring-php   # mantenha apenas o que precisar
```

---

## Conteúdo disponível

| Linguagem | Smells | Técnicas | Templates | Idiomas           |
|-----------|--------|----------|-----------|-------------------|
| Go        | 23     | 19       | 2         | `en`, `pt-br`     |
| Java      | 23     | 19       | 2         | `en`, `pt-br`     |
| Python    | 23     | 19       | 2         | `en`, `pt-br`     |
| PHP       | 23     | 19       | 2         | `en`, `pt-br`     |

**Smells (23):** Alternative Classes with Different Interfaces · Comments · Data Class · Data Clumps · Dead Code · Divergent Change · Duplicate Code · Feature Envy · Inappropriate Intimacy · Incomplete Library Class · Large Class · Lazy Class · Long Method · Long Parameter List · Message Chains · Middle Man · Parallel Inheritance Hierarchies · Primitive Obsession · Refused Bequest · Shotgun Surgery · Speculative Generality · Switch Statements · Temporary Field

**Técnicas (19):** Decompose Conditional · Extract Class · Extract Method · Extract Variable · Hide Delegate · Inline Class · Inline Method · Inline Temp · Introduce Parameter Object · Move Field · Move Method · Remove Assignments to Parameters · Remove Middle Man · Replace Conditional with Polymorphism · Replace Method with Method Object · Replace Nested Conditional with Guard Clauses · Replace Temp with Query · Split Temporary Variable · Substitute Algorithm

**Templates:** Code Review Instruction · Refactoring Task Prompt

> Todo o conteúdo está disponível em inglês (`en/`) e português brasileiro (`pt-br/`).

---

## Contribuindo

### Adicionando uma nova linguagem

1. Crie a estrutura dentro de `skills/refactoring-rust/`:

   ```bash
   mkdir -p skills/refactoring-rust/{en,pt-br}/details/{smells,techniques}
   ```

2. Adicione um `SKILL.md` de entrada seguindo o padrão de qualquer linguagem existente
3. Use `skills/refactoring-go/en/details/smells/long-method.md` como modelo para o formato e as seções

### Adicionando uma nova skill

1. Adicione o arquivo de detalhe: `skills/refactoring-{lang}/{locale}/details/smells/{nome}.md`
2. Adicione a técnica correspondente: `skills/refactoring-{lang}/{locale}/details/techniques/{nome}.md`
3. Atualize os arquivos de índice `skills/refactoring-{lang}/{locale}/smells.md` e `techniques.md`
4. Mantenha os nomes das seções consistentes: Problema, Solução, Quando aplicar, Passos, Exemplo, Exemplos negativos, Benefícios
