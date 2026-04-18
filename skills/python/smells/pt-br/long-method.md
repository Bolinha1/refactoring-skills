# SKILL: Detecção e Refatoração de Long Method — Python

## Fonte
Baseado em: https://refactoring.guru/smells/long-method

---

## 1. O que é Long Method

Um método que contém linhas de código em excesso.
Como regra geral: qualquer método com mais de 10 linhas já merece atenção.

**Por que isso acontece:**
- Sempre se adiciona lógica ao método, nunca se remove
- É mentalmente mais fácil adicionar duas linhas do que criar um novo método
- O problema cresce silenciosamente até virar código espaguete

---

## 2. Sinais de alerta (gatilhos para acionar este SKILL)

- [ ] Método com mais de 10 linhas
- [ ] Bloco de código que você sentiu vontade de comentar
- [ ] Condicional complexa (if/elif aninhado)
- [ ] Loop com lógica não trivial dentro
- [ ] Variáveis temporárias espalhadas pelo método
- [ ] Método que faz mais de uma coisa claramente distinta

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                        | Técnica indicada                  |
|--------------------------------------------|-----------------------------------|
| Trecho de código comentado ou comentável   | Extract Method                    |
| Variáveis locais impedem a extração        | Replace Temp with Query           |
| Muitos parâmetros no método extraído       | Introduce Parameter Object        |
| Objeto inteiro sendo desmembrado           | Preserve Whole Object             |
| Nenhuma das anteriores resolve             | Replace Method with Method Object |
| Condicional complexa                       | Decompose Conditional             |
| Loop com lógica interna complexa           | Extract Method                    |

**Regra de ouro:** Se você sentiu vontade de escrever um comentário dentro do método,
esse trecho deve virar um método com nome descritivo. O nome substitui o comentário.

---

## 4. Exemplo

**ANTES — não aceito:**
```python
def processar_pedido(self, pedido):
    # validar pedido
    if not pedido.itens:
        raise PedidoInvalidoError("Pedido sem itens")
    if pedido.cliente is None:
        raise PedidoInvalidoError("Cliente não informado")

    # calcular total
    total = 0
    for item in pedido.itens:
        total += item.preco * item.quantidade
    if pedido.tem_desconto:
        total = total * (1 - pedido.desconto)

    # persistir
    self.pedido_repository.save(pedido)
    self.email_service.enviar_confirmacao(pedido.cliente.email)
```

**DEPOIS — esperado:**
```python
def processar_pedido(self, pedido):
    self._validar_pedido(pedido)
    total = self._calcular_total(pedido)
    pedido.total = total
    self._confirmar_pedido(pedido)

def _validar_pedido(self, pedido):
    if not pedido.itens:
        raise PedidoInvalidoError("Pedido sem itens")
    if pedido.cliente is None:
        raise PedidoInvalidoError("Cliente não informado")

def _calcular_total(self, pedido):
    total = sum(item.preco * item.quantidade for item in pedido.itens)
    return total * (1 - pedido.desconto) if pedido.tem_desconto else total

def _confirmar_pedido(self, pedido):
    self.pedido_repository.save(pedido)
    self.email_service.enviar_confirmacao(pedido.cliente.email)
```

**Por que esse padrão:**
- Cada método privado (`_`) tem nome que expressa intenção de negócio
- O método principal virou uma narrativa legível
- Nenhum comentário necessário — o nome substitui

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair sem nomear com intenção**
```python
# Não aceito — nome não diz o que faz
def _faz_alguma_coisa(self, pedido): ...
def _metodo1(self, pedido): ...
def _processar_parte2(self, pedido): ...
```

**Erro 2: Comentar em vez de extrair**
```python
# Não aceito — comentário indica que devia ser um método
# valida os dados do cliente
if cliente is None or cliente.cpf is None:
    ...
```

**Erro 3: Extrair e deixar método ainda longo**
```python
# Não aceito — extraiu um trecho mas o método original
# ainda tem 60 linhas com múltiplas responsabilidades
```

---

## 6. Benefícios

- **Longevidade:** Classes com métodos curtos vivem mais
- **Manutenção:** Reduz a dificuldade de entender e manter o código
- **Qualidade:** Previne código duplicado escondido em métodos longos
- **Performance:** Impacto negativo desprezível; habilita otimizações melhores
- **Clareza:** Código claro e compreensível facilita identificar melhorias reais de performance
