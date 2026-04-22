---
name: refactoring-templates
description: >
  Ready-to-use prompt templates for AI-assisted code reviews and refactoring tasks.
  Includes a code review instruction checklist and a refactoring task prompt. Available
  in English and Brazilian Portuguese.
---

# Refactoring Templates

## When to use
When the user wants a ready-made prompt or instruction for conducting a code review
focused on refactoring, or for guiding a refactoring task.

## Locale
- If the user writes in Portuguese → read files from `pt-br/`
- Default → read files from `en/`

## Available templates
- **Code Review Instruction** → `{locale}/code-review-instruction.md`
  System instruction for AI-assisted code reviews. Contains a checklist of smells
  (Long Method, Large Class, Primitive Obsession, Feature Envy, Complex Conditionals)
  and expected response format.

- **Refactoring Task Prompt** → `{locale}/refactoring-task-prompt.md`
  Prompt template to guide a refactoring session on a specific piece of code.
