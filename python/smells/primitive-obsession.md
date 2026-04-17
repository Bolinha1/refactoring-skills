# SKILL: Detecção e Refatoração de Primitive Obsession — Python

## Fonte
Baseado em: https://refactoring.guru/smells/primitive-obsession

---

## 1. O que é Primitive Obsession

Uso excessivo de tipos básicos (`str`, `int`, `float`, `bool`, `dict`, `list`)
para representar conceitos de domínio que mereceriam classes ou dataclasses próprias.

**Por que isso acontece:**
- Criar uma nova classe parece exagero para algo "simples"
- `dict` é conveniente e rápido de usar
- O código cresce e as chaves do dict viram strings mágicas difíceis de rastrear

---

## 2. Sinais de alerta (gatilhos para acionar este SKILL)

- [ ] `str` representando CPF, telefone, CEP, e-mail, código de produto
- [ ] `int` ou `str` constante simulando tipo (ex: `ROLE_ADMIN = "admin"`)
- [ ] `dict` com chaves mágicas de string para estruturar dados de domínio
- [ ] Múltiplos parâmetros primitivos que sempre aparecem juntos (ex: `valor: float, moeda: str`)
- [ ] Validação do mesmo primitivo repetida em vários lugares

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                  | Técnica indicada               |
|------------------------------------------------------|-------------------------------|
| Primitivo com regras de validação próprias            | Replace Data Value with Object |
| Múltiplos primitivos que andam juntos                | Introduce Parameter Object     |
| Primitivo passado como grupo                         | Preserve Whole Object          |
| `str`/`int` simulando tipo enumerado                 | Replace Type Code with Class   |
| `dict` ou `list` como estrutura de dados             | Replace Array with Object      |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
class Cliente:
    def __init__(self, nome: str, cpf: str, telefone: str, email: str):
        self.nome = nome
        self.cpf = cpf          # string crua, sem validação centralizada
        self.telefone = telefone
        self.email = email

class PedidoService:
    def criar(self, cpf_cliente: str, valor: float, moeda: str, tipo_pagamento: str):
        # tipo_pagamento: "cartao", "boleto", "pix" — string mágica
        if tipo_pagamento == "cartao":
            ...
        # validação de CPF duplicada em vários pontos
        if not cpf_cliente or len(cpf_cliente) != 11:
            raise ValueError("CPF inválido")

    def buscar_endereco(self, dados: dict):
        # chaves mágicas no dict
        rua = dados["rua"]
        numero = dados["numero"]
        cidade = dados["cidade"]
        cep = dados["cep"]
```

**DEPOIS — esperado:**
```python
from dataclasses import dataclass
from enum import Enum
import re


# Value Object para CPF
class Cpf:
    def __init__(self, valor: str):
        if not valor or not re.fullmatch(r"\d{11}", valor):
            raise ValueError("CPF inválido")
        self._valor = valor

    @property
    def valor(self) -> str:
        return self._valor

    def __eq__(self, other):
        return isinstance(other, Cpf) and self._valor == other._valor

    def __hash__(self):
        return hash(self._valor)


# Value Object para Email
class Email:
    def __init__(self, endereco: str):
        if not endereco or "@" not in endereco:
            raise ValueError("E-mail inválido")
        self._endereco = endereco

    @property
    def endereco(self) -> str:
        return self._endereco


# Parameter Object para dados que andam juntos
@dataclass(frozen=True)
class Dinheiro:
    valor: float
    moeda: str


# Enum no lugar de string mágica
class TipoPagamento(Enum):
    CARTAO = "cartao"
    BOLETO = "boleto"
    PIX = "pix"


# Replace Array with Object
@dataclass
class Endereco:
    rua: str
    numero: str
    cidade: str
    cep: str


class Cliente:
    def __init__(self, nome: str, cpf: Cpf, email: Email):
        self.nome = nome
        self.cpf = cpf
        self.email = email


class PedidoService:
    def criar(self, cpf_cliente: Cpf, valor: Dinheiro, tipo_pagamento: TipoPagamento):
        if tipo_pagamento == TipoPagamento.CARTAO:
            ...

    def buscar_endereco(self, endereco: Endereco):
        rua = endereco.rua
        cidade = endereco.cidade
```

**Por que esse padrão:**
- A validação do CPF está centralizada em `Cpf` — não se repete
- `TipoPagamento` é autoexplicativo, sem strings mágicas
- `Dinheiro` agrupa valor e moeda com `frozen=True` garantindo imutabilidade
- `Endereco` substitui o dict com campos tipados e com nome

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: `str` para tudo**
```python
# Não aceito
def processar(cpf: str, status: str, tipo_pagamento: str):
    if status == "ativo" and tipo_pagamento == "pix":
        ...
```

**Erro 2: Constantes string no lugar de Enum**
```python
# Não aceito
PAGAMENTO_CARTAO = "cartao"
PAGAMENTO_BOLETO = "boleto"
if tipo == PAGAMENTO_CARTAO:
    ...
```

**Erro 3: `dict` com chaves mágicas**
```python
# Não aceito
endereco = {
    "rua": "Rua A",
    "numero": "123",
    "cidade": "SP",
    "cep": "01310-100",
}
```

---

## 6. Benefícios

- **Flexibilidade:** Regras de negócio ficam encapsuladas no objeto correto
- **Legibilidade:** Parâmetros e campos expressam intenção de domínio
- **Manutenção:** Validação centralizada — muda em um lugar, vale em todos
- **Segurança:** Tipagem forte facilita análise estática com `mypy`
