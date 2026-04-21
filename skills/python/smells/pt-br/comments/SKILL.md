# SKILL: Detectando e Refatorando Comments — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/comments

---

## 1. O que é Comments (como code smell)

Uma função ou método está cheia de comentários explicativos porque o código em si não é claro o suficiente para ser compreendido sem eles. O comentário é um sintoma: ele marca um lugar onde o código deveria falar por si mesmo, mas não consegue.

**Importante:** Nem todo comentário é um smell. Comentários que explicam *por que* uma decisão não óbvia foi tomada são valiosos. Comentários que explicam *o que* o código faz são o smell — eles deveriam ser substituídos por nomes melhores e funções menores.

**Por que isso acontece:**
- O código foi escrito para funcionar, não para comunicar
- Lógica complexa foi deixada inline em vez de ser extraída para funções nomeadas
- Nomes de variáveis e funções não expressam intenção

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Um comentário descreve *o que* um bloco de código faz (não *por que*)
- [ ] Um comentário precede um bloco que poderia ser nomeado e extraído
- [ ] Uma variável precisa de um comentário para explicar o que ela contém
- [ ] Um corpo de função tem divisores de seção (`# --- etapa 1 ---`)
- [ ] Um comentário reformula o nome da função em prosa
- [ ] Uma docstring de várias linhas descreve o algoritmo linha por linha

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                    | Técnica recomendada       |
|--------------------------------------------------------|---------------------------|
| Comentário descreve um bloco de código                 | Extract Method            |
| Comentário explica o que uma variável contém           | Renomear variável ou Extract Variable |
| Comentário explica uma condição complexa               | Extract Method com nome descritivo |
| Comentário marca um TODO que deveria ser código        | Introduce Assertion        |
| Comentário explica POR QUÊ (restrição não óbvia)       | Manter o comentário — ele adiciona valor |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
def processar_pagamento(pagamento) -> None:
    # validar que o pagamento não expirou
    if pagamento.data_expiracao < date.today():
        raise PagamentoExpiradoError("Pagamento expirado")

    # calcular valor líquido após taxas
    taxa = pagamento.valor * 0.025
    valor_liquido = pagamento.valor - taxa

    # enviar para o gateway de pagamento
    gateway.enviar(pagamento.numero_cartao, valor_liquido)

    # notificar cliente
    email_service.enviar_confirmacao_pagamento(pagamento.email_cliente, valor_liquido)
```

**DEPOIS — esperado:**
```python
def processar_pagamento(pagamento) -> None:
    _validar_nao_expirado(pagamento)
    valor_liquido = _calcular_valor_liquido(pagamento)
    gateway.enviar(pagamento.numero_cartao, valor_liquido)
    email_service.enviar_confirmacao_pagamento(pagamento.email_cliente, valor_liquido)


def _validar_nao_expirado(pagamento) -> None:
    if pagamento.data_expiracao < date.today():
        raise PagamentoExpiradoError("Pagamento expirado")


def _calcular_valor_liquido(pagamento) -> float:
    taxa = pagamento.valor * TAXA_PROCESSAMENTO
    return pagamento.valor - taxa
```

**Por que este padrão:**
- Nomes de funções substituem comentários — `_validar_nao_expirado` é mais preciso que `# validar`
- A função principal lê como uma narrativa de negócio, não como detalhes de implementação

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter o comentário após extrair**
```python
# Não aceito — o comentário e o nome da função dizem a mesma coisa
# validar pagamento
_validar_pagamento(pagamento)
```

**Erro 2: Extrair com um nome vago**
```python
# Não aceito — o nome da função não substitui o comentário
def _fazer_validacao(pagamento): ...
```

**Erro 3: Remover comentários explicativos de POR QUÊ**
```python
# Não aceito — este comentário explica uma restrição não óbvia; deve ser mantido
# Usando floor() intencionalmente — o banco trunca frações, nunca arredonda
taxa = math.floor(pagamento.valor * TAXA * 100) / 100
```

---

## 6. Benefícios

- **Autodocumentação:** Código que lê como prosa não precisa de comentários
- **Manutenção:** Nomes de funções não podem ficar obsoletos como os comentários
- **Descoberta:** Funções extraídas se tornam blocos de construção reutilizáveis
