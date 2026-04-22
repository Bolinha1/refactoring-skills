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
skills/                              ← entry points compatíveis com skills.sh
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

catalog/                             ← referência granular (estrutura original)
├── go/smells/{en,pt-br}/*/SKILL.md
├── java/...
├── python/...
└── php/...
```

Cada `SKILL.md` segue um formato consistente: definição do problema, quando aplicar, passo a passo da refatoração,
exemplos de código antes/depois, exemplos negativos (o que NÃO fazer) e benefícios.

---

## Clonar o catálogo completo

### Opção 1 — Clonar tudo (mais simples)

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
```

### Opção 2 — Clonar apenas a sua linguagem (recomendado)

Evita baixar pacotes que você não vai usar:

```bash
git clone --filter=blob:none --sparse https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
git sparse-checkout set catalog/java   # substitua por: catalog/go, catalog/python ou catalog/php
```

### Opção 3 — Clonar tudo e remover o que não precisa

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
rm -rf catalog/go catalog/python catalog/php   # mantenha apenas o que precisar
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

1. Crie a estrutura de diretórios:

   ```bash
   mkdir -p catalog/rust/{smells,techniques,templates}/{en,pt-br}
   ```

2. Adicione as pastas de skills seguindo a estrutura existente de qualquer linguagem como referência
3. Use `catalog/go/smells/en/long-method/SKILL.md` como modelo para o formato e as seções
4. Adicione uma entrada correspondente em `skills/refactoring-rust/` seguindo o padrão de qualquer linguagem existente

### Adicionando uma nova skill

1. Crie uma pasta em `catalog/{lang}/smells/{locale}/nome-do-smell/` e adicione `SKILL.md`
2. Crie a pasta correspondente em `catalog/{lang}/techniques/{locale}/nome-da-tecnica/` e adicione `SKILL.md`
3. Copie os arquivos de detalhe para `skills/refactoring-{lang}/{locale}/details/`
4. Atualize os arquivos de índice `skills/refactoring-{lang}/{locale}/smells.md` e `techniques.md`
5. Mantenha os nomes das seções consistentes: Problema, Solução, Quando aplicar, Passos, Exemplo, Exemplos negativos, Benefícios
