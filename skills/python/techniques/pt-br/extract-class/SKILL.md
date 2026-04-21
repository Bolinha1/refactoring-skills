# TÉCNICA: Extract Class — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/extract-class

---

## 1. Problema

Uma classe faz o trabalho de duas. Um subconjunto de seus atributos e métodos forma um conceito coeso que faria sentido como sua própria classe.

---

## 2. Solução

Crie uma nova classe (ou dataclass) e mova os atributos e métodos que pertencem a esse conceito para ela. Substitua os atributos originais por uma referência à nova classe.

---

## 3. Quando aplicar

- A classe tem atributos que são sempre usados juntos (Data Clumps)
- A classe tem métodos que tocam apenas um subconjunto de seus atributos
- A classe cresceu para lidar com mais de uma responsabilidade distinta (Divergent Change)
- Um subconjunto da classe poderia ser reutilizado independentemente em outro contexto
- O nome da classe não consegue descrever o que ela faz sem usar "e"

---

## 4. Passos de refatoração

1. Identifique o grupo de atributos e métodos a extrair — eles devem formar um conceito coeso
2. Crie uma nova classe com um nome que expresse esse conceito
3. Crie uma instância da nova classe como atributo na classe original
4. Mova os atributos identificados para a nova classe
5. Mova os métodos identificados para a nova classe
6. Decida sobre a exposição:
   - Exporte a classe do módulo se pode ser usada independentemente ou reutilizada
   - Mantenha com prefixo `_` (privada ao módulo) se é um detalhe interno
7. Execute os testes após cada movimentação

---

## 5. Exemplo

**ANTES — não aceito:**
```python
class Pessoa:
    def __init__(self, nome: str, codigo_area_escritorio: str, numero_escritorio: str):
        self.nome = nome
        self.codigo_area_escritorio = codigo_area_escritorio
        self.numero_escritorio = numero_escritorio

    def get_numero_telefone(self) -> str:
        return f"({self.codigo_area_escritorio}) {self.numero_escritorio}"
```

**DEPOIS — esperado:**
```python
from dataclasses import dataclass

@dataclass
class NumeroTelefone:
    codigo_area: str
    numero: str

    def formatar(self) -> str:
        return f"({self.codigo_area}) {self.numero}"


class Pessoa:
    def __init__(self, nome: str, telefone_escritorio: NumeroTelefone):
        self.nome = nome
        self.telefone_escritorio = telefone_escritorio

    def get_numero_telefone(self) -> str:
        return self.telefone_escritorio.formatar()
```

**Variante — extraindo um data clump:**
```python
# ANTES — atributos de endereço espalhados em Pessoa e Fatura
class Pessoa:
    def __init__(self):
        self.rua = ""
        self.cidade = ""
        self.cep = ""

# DEPOIS — Endereco é um conceito independente e reutilizável
@dataclass
class Endereco:
    rua: str
    cidade: str
    cep: str

    def formatar(self) -> str:
        return f"{self.rua}, {self.cidade} {self.cep}"

class Pessoa:
    def __init__(self, endereco_residencial: Endereco):
        self.endereco_residencial = endereco_residencial
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair uma classe sem coesão**
```python
# Não aceito — DadosPessoa agrupa atributos sem relacionamento natural
@dataclass
class DadosPessoa:
    nome: str
    codigo_escritorio: str
    salario: float
    data_contratacao: date
```

**Erro 2: Mover métodos sem mover os atributos que eles precisam**
```python
# Não aceito — NumeroTelefone.formatar() ainda recebe campos brutos de Pessoa
class NumeroTelefone:
    def formatar(self, codigo_area: str, numero: str) -> str: ...  # ainda acoplado
```

**Erro 3: Extrair mas deixar os atributos originais no lugar**
```python
# Não aceito — agora há duas fontes da verdade para dados de telefone
class Pessoa:
    def __init__(self):
        self.telefone = NumeroTelefone(...)      # novo
        self.codigo_area_escritorio = ""         # antigo — deve ser deletado
        self.numero_escritorio = ""             # antigo — deve ser deletado
```

---

## 7. Benefícios

- **Responsabilidade Única:** Cada classe tem um conceito claro para representar
- **Reutilização:** A classe extraída pode ser usada por outras classes independentemente
- **Encapsulamento:** Lógica de validação e formatação do conceito vive em um único lugar
