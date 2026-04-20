# Refactoring Skills

A structured knowledge base for identifying code smells and applying refactoring techniques,
organized by programming language and locale. Based on the [refactoring.guru](https://refactoring.guru) catalog.

---

## Repository structure

```
skills/
├── java/
│   ├── smells/
│   │   ├── en/
│   │   │   ├── long-method/SKILL.md
│   │   │   ├── large-class/SKILL.md
│   │   │   └── primitive-obsession/SKILL.md
│   │   └── pt-br/
│   │       ├── long-method/SKILL.md
│   │       ├── large-class/SKILL.md
│   │       └── primitive-obsession/SKILL.md
│   ├── techniques/
│   │   ├── en/
│   │   │   ├── extract-method/SKILL.md
│   │   │   ├── move-method/SKILL.md
│   │   │   └── replace-conditional-with-polymorphism/SKILL.md
│   │   └── pt-br/
│   │       ├── extract-method/SKILL.md
│   │       ├── move-method/SKILL.md
│   │       └── replace-conditional-with-polymorphism/SKILL.md
│   └── templates/
│       ├── en/
│       │   ├── code-review-instruction/SKILL.md
│       │   └── refactoring-task-prompt/SKILL.md
│       └── pt-br/
│           ├── code-review-instruction/SKILL.md
│           └── refactoring-task-prompt/SKILL.md
├── python/              # same structure
└── php/                 # same structure
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
git sparse-checkout set skills/java   # replace with: skills/python or skills/php
```

### Option 3 — Clone everything, then remove unused packages

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
rm -rf skills/python skills/php       # keep only what you need
```

---

## Available content

| Language | Smells | Techniques | Templates | Locales |
|----------|--------|------------|-----------|---------|
| Java     | 3      | 3          | 2         | `en`, `pt-br` |
| Python   | 3      | 3          | 2         | `en`, `pt-br` |
| PHP      | 3      | 3          | 2         | `en`, `pt-br` |

**Smells:** Long Method · Large Class · Primitive Obsession

**Techniques:** Extract Method · Move Method · Replace Conditional with Polymorphism

**Templates:** Code Review Instruction · Refactoring Task Prompt

> All content is available in English (`en/`) and Brazilian Portuguese (`pt-br/`).

---

## Contributing

### Adding a new language

1. Create the directory structure:
   ```bash
   mkdir -p skills/go/{smells,techniques,templates}/{en,pt-br}
   ```
2. Add the skill folders following the existing structure of any language as reference
3. Use `skills/java/smells/en/long-method/SKILL.md` as a template for format and sections

### Adding a new skill

1. Create a folder under `{lang}/smells/{locale}/skill-name/` and add `SKILL.md`
2. Create the corresponding folder under `{lang}/techniques/{locale}/skill-name/` and add `SKILL.md`
3. Keep section names consistent: Problem, Solution, When to apply, Steps, Example, Negative examples, Benefits
