# TÉCNICA: Replace Conditional with Polymorphism — Python

## Fonte
Baseado em: https://refactoring.guru/replace-conditional-with-polymorphism

---

## 1. Problema

Você tem uma cadeia de `if/elif/else` que executa comportamentos diferentes
dependendo do tipo do objeto ou de uma propriedade que simula um tipo.

---

## 2. Solução

Crie subclasses correspondentes a cada ramo do condicional. Em cada subclasse,
implemente um método que contém o comportamento daquele ramo. Substitua o condicional
por uma chamada polimórfica ao método.

---

## 3. Quando aplicar

- O condicional verifica `isinstance()`, um campo tipo, ou uma string/enum
- O mesmo condicional aparece em múltiplos métodos
- Adicionar um novo tipo exigiria modificar todos os condicionais existentes (violação do Open/Closed Principle)
- Cada ramo do condicional representa um comportamento coeso e distinto

---

## 4. Passos de refatoração

1. Se o condicional está dentro de um método com outras responsabilidades, aplique **Extract Method** primeiro
2. Crie uma classe base com o método a ser polimorfizado (use `ABC` para forçar implementação)
3. Para cada ramo do condicional, crie uma subclasse que sobrescreve o método
4. Mova a lógica do ramo para o método da subclasse correspondente
5. Remova o ramo do condicional original
6. Repita até o condicional estar vazio
7. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
class Funcionario:
    def __init__(self, tipo: str, salario_base: float, vendas: float = 0):
        self.tipo = tipo           # "engenheiro", "gerente", "vendedor"
        self.salario_base = salario_base
        self.vendas = vendas

    def calcular_bonus(self) -> float:
        if self.tipo == "engenheiro":
            return self.salario_base * 0.10
        elif self.tipo == "gerente":
            return self.salario_base * 0.20 + 1000
        elif self.tipo == "vendedor":
            return self.vendas * 0.05
        else:
            raise ValueError(f"Tipo desconhecido: {self.tipo}")

    def gerar_relatorio(self) -> str:
        if self.tipo == "engenheiro":
            return "Engenheiro: projetos entregues"
        elif self.tipo == "gerente":
            return "Gerente: equipes lideradas"
        elif self.tipo == "vendedor":
            return f"Vendedor: R$ {self.vendas:.2f} em vendas"
        else:
            raise ValueError(f"Tipo desconhecido: {self.tipo}")
```

**DEPOIS — esperado:**
```python
from abc import ABC, abstractmethod


class Funcionario(ABC):
    def __init__(self, salario_base: float):
        self.salario_base = salario_base

    @abstractmethod
    def calcular_bonus(self) -> float: ...

    @abstractmethod
    def gerar_relatorio(self) -> str: ...


class Engenheiro(Funcionario):
    def calcular_bonus(self) -> float:
        return self.salario_base * 0.10

    def gerar_relatorio(self) -> str:
        return "Engenheiro: projetos entregues"


class Gerente(Funcionario):
    def calcular_bonus(self) -> float:
        return self.salario_base * 0.20 + 1000

    def gerar_relatorio(self) -> str:
        return "Gerente: equipes lideradas"


class Vendedor(Funcionario):
    def __init__(self, salario_base: float, vendas: float):
        super().__init__(salario_base)
        self.vendas = vendas

    def calcular_bonus(self) -> float:
        return self.vendas * 0.05

    def gerar_relatorio(self) -> str:
        return f"Vendedor: R$ {self.vendas:.2f} em vendas"
```

**Por que esse padrão:**
- Adicionar um novo tipo de funcionário → criar nova subclasse, sem tocar nas existentes
- Sem `if/elif` duplicado em `calcular_bonus` e `gerar_relatorio`
- `ABC` garante em tempo de definição que nenhuma subclasse esqueça de implementar os métodos

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover o if/elif para uma factory e achar que resolveu**
```python
# Não aceito — o condicional continua existindo, só mudou de lugar
def criar_funcionario(tipo: str, salario_base: float) -> Funcionario:
    if tipo == "engenheiro":
        return Funcionario("engenheiro", salario_base)  # mesma classe
    # ...
```

**Erro 2: Criar subclasses mas manter condicional nos métodos**
```python
# Não aceito — criou herança mas não polimorfizou o comportamento
class Engenheiro(Funcionario):
    def calcular_bonus(self) -> float:
        if self.tipo == "engenheiro":  # condicional desnecessário
            return self.salario_base * 0.10
        return 0
```

**Erro 3: Usar polimorfismo para condicionais de negócio triviais**
```python
# Não aceito — over-engineering para um if simples
# Ex: if ativo: ... else: ...
```

---

## 7. Benefícios

- **Open/Closed Principle:** Novos tipos não exigem modificação do código existente
- **Eliminação de duplicação:** O mesmo condicional não precisa ser repetido
- **Legibilidade:** Cada classe expressa claramente seu comportamento
- **Testabilidade:** Cada subclasse pode ser testada isoladamente
