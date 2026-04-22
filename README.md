# Refactoring Skills

> [Leia em Português](docs/README.pt-br.md)

A structured knowledge base for identifying code smells and applying refactoring techniques,
organized by programming language and locale. Based on the [refactoring.guru](https://refactoring.guru) catalog.

---

## Install via skills CLI

Use with any AI agent that supports [skills.sh](https://skills.sh):

```bash
# Install all languages
npx skills add Bolinha1/refactoring-skills

# Install a specific language
npx skills add Bolinha1/refactoring-skills --skill refactoring-java
npx skills add Bolinha1/refactoring-skills --skill refactoring-go
npx skills add Bolinha1/refactoring-skills --skill refactoring-python
npx skills add Bolinha1/refactoring-skills --skill refactoring-php

# Install prompt templates only
npx skills add Bolinha1/refactoring-skills --skill refactoring-templates
```

Once installed, the agent automatically identifies smells and applies the right refactoring technique when reviewing code.

---

## Repository structure

```
skills/                              ← skills.sh-compatible entry points
├── refactoring-go/
│   ├── SKILL.md                     ← entry point (locale-aware)
│   ├── en/
│   │   ├── smells.md                ← index of 23 smells
│   │   ├── techniques.md            ← index of 19 techniques
│   │   └── details/
│   │       ├── smells/              ← full content per smell
│   │       └── techniques/          ← full content per technique
│   └── pt-br/                       ← same structure in Portuguese
├── refactoring-java/                ← same structure
├── refactoring-python/              ← same structure
├── refactoring-php/                 ← same structure
└── refactoring-templates/           ← code review + refactoring prompts

catalog/                             ← granular reference (original structure)
├── go/smells/{en,pt-br}/*/SKILL.md
├── java/...
├── python/...
└── php/...
```

Each `SKILL.md` follows a consistent format: problem definition, when to apply, step-by-step refactoring,
before/after code examples, negative examples (what NOT to do), and benefits.

---

## Clone the full catalog

### Option 1 — Clone everything (simplest)

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
```

### Option 2 — Clone only your language (recommended)

Avoids downloading packages you won't use:

```bash
git clone --filter=blob:none --sparse https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
git sparse-checkout set catalog/java   # replace with: catalog/go, catalog/python or catalog/php
```

### Option 3 — Clone everything, then remove unused packages

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
rm -rf catalog/go catalog/python catalog/php   # keep only what you need
```

---

## Available content

| Language | Smells | Techniques | Templates | Locales |
|----------|--------|------------|-----------|---------|
| Go       | 23     | 19         | 2         | `en`, `pt-br` |
| Java     | 23     | 19         | 2         | `en`, `pt-br` |
| Python   | 23     | 19         | 2         | `en`, `pt-br` |
| PHP      | 23     | 19         | 2         | `en`, `pt-br` |

**Smells (23):** Alternative Classes with Different Interfaces · Comments · Data Class · Data Clumps · Dead Code · Divergent Change · Duplicate Code · Feature Envy · Inappropriate Intimacy · Incomplete Library Class · Large Class · Lazy Class · Long Method · Long Parameter List · Message Chains · Middle Man · Parallel Inheritance Hierarchies · Primitive Obsession · Refused Bequest · Shotgun Surgery · Speculative Generality · Switch Statements · Temporary Field

**Techniques (19):** Decompose Conditional · Extract Class · Extract Method · Extract Variable · Hide Delegate · Inline Class · Inline Method · Inline Temp · Introduce Parameter Object · Move Field · Move Method · Remove Assignments to Parameters · Remove Middle Man · Replace Conditional with Polymorphism · Replace Method with Method Object · Replace Nested Conditional with Guard Clauses · Replace Temp with Query · Split Temporary Variable · Substitute Algorithm

**Templates:** Code Review Instruction · Refactoring Task Prompt

> All content is available in English (`en/`) and Brazilian Portuguese (`pt-br/`).

---

## Contributing

### Adding a new language

1. Create the directory structure:
   ```bash
   mkdir -p catalog/rust/{smells,techniques,templates}/{en,pt-br}
   ```
2. Add the skill folders following the existing structure of any language as reference
3. Use `catalog/go/smells/en/long-method/SKILL.md` as a template for format and sections
4. Add a corresponding entry in `skills/refactoring-rust/` following the pattern of any existing language skill

### Adding a new skill

1. Create a folder under `catalog/{lang}/smells/{locale}/skill-name/` and add `SKILL.md`
2. Create the corresponding folder under `catalog/{lang}/techniques/{locale}/skill-name/` and add `SKILL.md`
3. Copy the detail files to `skills/refactoring-{lang}/{locale}/details/`
4. Update the index files `skills/refactoring-{lang}/{locale}/smells.md` and `techniques.md`
5. Keep section names consistent: Problem, Solution, When to apply, Steps, Example, Negative examples, Benefits
