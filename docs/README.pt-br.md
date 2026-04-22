# Refactoring Skills

Uma base de conhecimento estruturada para identificar code smells e aplicar técnicas de refatoração,
organizada por linguagem de programação e idioma. Baseada no catálogo do [refactoring.guru](https://refactoring.guru/pt-br/refactoring).

---

## Estrutura do repositório

```text
skills/
├── go/
│   ├── smells/
│   │   ├── en/
│   │   │   ├── long-method/SKILL.md
│   │   │   ├── large-class/SKILL.md
│   │   │   └── ...                      # 23 smells no total
│   │   └── pt-br/                       # mesmos 23 smells
│   └── techniques/
│       ├── en/
│       │   ├── extract-method/SKILL.md
│       │   ├── move-method/SKILL.md
│       │   └── ...                      # 19 técnicas no total
│       └── pt-br/                       # mesmas 19 técnicas
├── java/                                # mesma estrutura + templates/
├── python/                              # mesma estrutura + templates/
└── php/                                 # mesma estrutura + templates/
```

Cada `SKILL.md` segue um formato consistente: definição do problema, quando aplicar, passo a passo da refatoração,
exemplos de código antes/depois, exemplos negativos (o que NÃO fazer) e benefícios.

---

## Como usar

### Opção 1 — Clonar tudo (mais simples)

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
```

### Opção 2 — Clonar apenas a sua linguagem (recomendado)

Evita baixar pacotes que você não vai usar:

```bash
git clone --filter=blob:none --sparse https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
git sparse-checkout set skills/java   # substitua por: skills/go, skills/python ou skills/php
```

### Opção 3 — Clonar tudo e remover o que não precisa

```bash
git clone https://github.com/Bolinha1/refactoring-skills.git
cd refactoring-skills
rm -rf skills/go skills/python skills/php   # mantenha apenas o que precisar
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
   mkdir -p skills/rust/{smells,techniques,templates}/{en,pt-br}
   ```

2. Adicione as pastas de skills seguindo a estrutura existente de qualquer linguagem como referência
3. Use `skills/go/smells/en/long-method/SKILL.md` como modelo para o formato e as seções

### Adicionando uma nova skill

1. Crie uma pasta em `{lang}/smells/{locale}/nome-do-smell/` e adicione `SKILL.md`
2. Crie a pasta correspondente em `{lang}/techniques/{locale}/nome-da-tecnica/` e adicione `SKILL.md`
3. Mantenha os nomes das seções consistentes: Problema, Solução, Quando aplicar, Passos, Exemplo, Exemplos negativos, Benefícios
