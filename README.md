# Refactoring Skills

> [Leia em Português](docs/README.pt-br.md)

A structured knowledge base for identifying code smells and applying refactoring techniques,
organized by programming language and locale. Based on the [refactoring.guru](https://refactoring.guru) catalog.

---

## Repository structure

```
skills/
├── go/
│   ├── smells/
│   │   ├── en/
│   │   │   ├── long-method/SKILL.md
│   │   │   ├── large-class/SKILL.md
│   │   │   └── ...                      # 23 smells total
│   │   └── pt-br/                       # same 23 smells
│   └── techniques/
│       ├── en/
│       │   ├── extract-method/SKILL.md
│       │   ├── move-method/SKILL.md
│       │   └── ...                      # 19 techniques total
│       └── pt-br/                       # same 19 techniques
├── java/                                # same structure + templates/
├── python/                              # same structure + templates/
└── php/                                 # same structure + templates/
```

Each `SKILL.md` follows a consistent format: problem definition, when to apply, step-by-step refactoring,
before/after code examples, negative examples (what NOT to do), and benefits.

---

## How to use

### Option 1 — Clone everything (simplest)

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
```

### Option 2 — Clone only your language (recommended)

Avoids downloading packages you won't use:

```bash
git clone --filter=blob:none --sparse https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
git sparse-checkout set skills/java   # replace with: skills/go, skills/python or skills/php
```

### Option 3 — Clone everything, then remove unused packages

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
rm -rf skills/go skills/python skills/php   # keep only what you need
```

---

## Available content

| Language | Smells | Techniques | Templates | Locales |
|----------|--------|------------|-----------|---------|
| Go       | 23     | 19         | —         | `en`, `pt-br` |
| Java     | 23     | 19         | 2         | `en`, `pt-br` |
| Python   | 23     | 19         | 2         | `en`, `pt-br` |
| PHP      | 23     | 19         | 2         | `en`, `pt-br` |

**Smells (23):** Alternative Classes with Different Interfaces · Comments · Data Class · Data Clumps · Dead Code · Divergent Change · Duplicate Code · Feature Envy · Inappropriate Intimacy · Incomplete Library Class · Large Class · Lazy Class · Long Method · Long Parameter List · Message Chains · Middle Man · Parallel Inheritance Hierarchies · Primitive Obsession · Refused Bequest · Shotgun Surgery · Speculative Generality · Switch Statements · Temporary Field

**Techniques (19):** Decompose Conditional · Extract Class · Extract Method · Extract Variable · Hide Delegate · Inline Class · Inline Method · Inline Temp · Introduce Parameter Object · Move Field · Move Method · Remove Assignments to Parameters · Remove Middle Man · Replace Conditional with Polymorphism · Replace Method with Method Object · Replace Nested Conditional with Guard Clauses · Replace Temp with Query · Split Temporary Variable · Substitute Algorithm

**Templates (Java · Python · PHP only):** Code Review Instruction · Refactoring Task Prompt

> All content is available in English (`en/`) and Brazilian Portuguese (`pt-br/`).

---

## Contributing

### Adding a new language

1. Create the directory structure:
   ```bash
   mkdir -p skills/rust/{smells,techniques,templates}/{en,pt-br}
   ```
2. Add the skill folders following the existing structure of any language as reference
3. Use `skills/go/smells/en/long-method/SKILL.md` as a template for format and sections

### Adding a new skill

1. Create a folder under `{lang}/smells/{locale}/skill-name/` and add `SKILL.md`
2. Create the corresponding folder under `{lang}/techniques/{locale}/skill-name/` and add `SKILL.md`
3. Keep section names consistent: Problem, Solution, When to apply, Steps, Example, Negative examples, Benefits
