# SKILL: Detectando e Refatorando Data Clumps — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/data-clumps

---

## 1. O que é Data Clumps

Partes diferentes do código contêm grupos idênticos de variáveis (um "clump") — os mesmos atributos em múltiplas classes, ou os mesmos parâmetros aparecendo juntos em muitas assinaturas de função. Se você removesse um item do grupo e os demais perdessem sentido, você tem um data clump.

**Por que isso acontece:**
- O relacionamento entre os itens de dados nunca foi formalizado como um dataclass ou classe
- Copy-paste propagou o grupo por lugares não relacionados
- O conceito existia informalmente na cabeça dos desenvolvedores, mas não no código

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Três ou mais atributos que aparecem juntos em múltiplas classes
- [ ] Os mesmos 2–3 parâmetros consistentemente passados juntos para funções
- [ ] Variáveis como `data_inicio`/`data_fim`, `latitude`/`longitude`, `rua`/`cidade`/`cep` como primitivos separados
- [ ] Você precisa atualizar o mesmo grupo de atributos em vários lugares para uma única mudança lógica

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                      | Técnica recomendada           |
|----------------------------------------------------------|-------------------------------|
| Clump aparece como atributos em uma classe               | Extract Class                 |
| Clump aparece em listas de parâmetros de funções         | Introduce Parameter Object    |
| Uma função recebe um objeto mas usa apenas parte dele    | Preserve Whole Object         |

**Teste chave:** remova um item do clump. Se os outros perderem significado, o grupo merece seu próprio dataclass.

---

## 4. Exemplo

**ANTES — não aceito:**
```python
class Pedido:
    def __init__(self):
        self.rua = ""
        self.cidade = ""
        self.cep = ""
        self.pais = ""
        self.nome_cliente = ""
        self.email_cliente = ""

class Fatura:
    def __init__(self):
        self.rua = ""    # mesmo clump novamente
        self.cidade = ""
        self.cep = ""
        self.pais = ""

def enviar(rua: str, cidade: str, cep: str, pais: str) -> None: ...
```

**DEPOIS — esperado:**
```python
from dataclasses import dataclass

@dataclass
class Endereco:
    rua: str
    cidade: str
    cep: str
    pais: str

    def formatar(self) -> str:
        return f"{self.rua}, {self.cidade} {self.cep}, {self.pais}"


@dataclass
class Pedido:
    endereco_entrega: Endereco
    nome_cliente: str
    email_cliente: str

@dataclass
class Fatura:
    endereco_cobranca: Endereco

def enviar(destino: Endereco) -> None: ...
```

**Por que este padrão:**
- `Endereco` é um conceito nomeado — sua validação, formatação e comparação agora vivem em um único lugar
- Toda classe que possui um endereço se beneficia de qualquer melhoria em `Endereco`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Agrupar o clump em um dict ou contêiner genérico**
```python
# Não aceito — perde segurança de tipos e intenção
endereco = {"rua": "Rua Principal", "cidade": "São Paulo"}
```

**Erro 2: Criar o dataclass mas manter os atributos separados antigos também**
```python
# Não aceito — agora há duas representações dos mesmos dados
self.endereco_entrega = Endereco(...)
self.rua = ""    # duplicado
self.cidade = "" # duplicado
```

**Erro 3: Agrupar dados sem coesão só porque aparecem juntos**
```python
# Não aceito — PreferenciasCliente agrupa campos não relacionados em uma classe
# só porque apareceram juntos na mesma assinatura de função
```

---

## 6. Benefícios

- **Fonte única da verdade:** A lógica do clump (validação, formatação) é centralizada
- **Legibilidade:** `pedido.endereco_entrega` é mais expressivo do que quatro atributos separados
- **Extensibilidade:** Adicionar um novo campo de endereço requer mudança apenas no dataclass `Endereco`
