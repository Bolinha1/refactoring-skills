# SKILL: Detectando e Refatorando Long Parameter List — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/long-parameter-list

---

## 1. O que é Long Parameter List

Uma função ou método tem parâmetros demais — tipicamente mais de três ou quatro. Listas longas de parâmetros são difíceis de entender, fáceis de confundir e dolorosas de chamar. Elas frequentemente indicam que dados relacionados deveriam ser agrupados em um dataclass ou objeto.

**Por que isso acontece:**
- Funções foram mescladas sem criar uma abstração adequada para os dados combinados
- Algoritmos foram ficando mais complexos ao longo do tempo, exigindo mais flags de controle
- Dados que naturalmente pertencem juntos são passados como primitivos individuais

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Assinatura de função/método com 4 ou mais parâmetros (excluindo `self`)
- [ ] Vários parâmetros do mesmo tipo em sequência (fácil de trocar acidentalmente)
- [ ] Parâmetros que sempre aparecem juntos em vários locais de chamada
- [ ] Parâmetros booleanos "flag" que mudam o comportamento da função
- [ ] O chamador precisa construir muitas variáveis locais só para chamar a função

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                      | Técnica recomendada                |
|----------------------------------------------------------|------------------------------------|
| Parâmetros são todos campos de um objeto existente       | Preserve Whole Object              |
| Parâmetros representam um conceito novo não no modelo    | Introduce Parameter Object         |
| Um parâmetro pode ser computado a partir de outros       | Replace Parameter with Method Call |
| Um flag booleano muda o comportamento da função          | Dividir em duas funções explícitas |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
def criar_pedido(
    id_cliente: str,
    nome_cliente: str,
    email_cliente: str,
    rua: str,
    cidade: str,
    cep: str,
    pais: str,
    ids_produtos: list[str],
    codigo_cupom: str | None = None,
) -> Recibo:
    ...
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

@dataclass
class RequisicaoPedido:
    id_cliente: str
    nome_cliente: str
    email_cliente: str
    endereco_entrega: Endereco
    ids_produtos: list[str]
    codigo_cupom: str | None = None


def criar_pedido(requisicao: RequisicaoPedido) -> Recibo:
    # lógica usa requisicao.id_cliente, requisicao.endereco_entrega, etc.
    ...
```

**Por que este padrão:**
- `RequisicaoPedido` e `Endereco` são conceitos de primeira classe — podem ser validados, reutilizados e testados de forma independente
- Os chamadores constroem um objeto significativo, não uma lista de argumentos posicionais

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Agrupar parâmetros não relacionados em um dataclass só para reduzir a contagem**
```python
# Não aceito — DataHolder não tem significado de domínio
@dataclass
class DataHolder:
    a: str
    b: str
    c: str
    ...
```

**Erro 2: Usar um dict como "objeto de parâmetro"**
```python
# Não aceito — perde segurança de tipos, suporte de IDE e descobribilidade
def criar_pedido(params: dict) -> Recibo: ...
```

**Erro 3: Usar *args/**kwargs para esconder a lista longa**
```python
# Não aceito — torna a API invisível e impossível de tipar
def criar_pedido(*args, **kwargs) -> Recibo: ...
```

---

## 6. Benefícios

- **Legibilidade:** Objetos de parâmetro nomeados tornam os locais de chamada autodocumentados
- **Segurança:** A confusão de argumentos posicionais é eliminada
- **Extensibilidade:** Adicionar um campo ao objeto de parâmetro não muda cada local de chamada
