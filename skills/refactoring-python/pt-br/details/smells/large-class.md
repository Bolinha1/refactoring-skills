# SKILL: Detecção e Refatoração de Large Class — Python

## Fonte
Baseado em: https://refactoring.guru/smells/large-class

---

## 1. O que é Large Class

Uma classe que contém muitos campos, métodos ou linhas de código.
Quando uma classe tenta fazer coisas demais, ela acumula responsabilidades
que deveriam estar distribuídas.

**Por que isso acontece:**
- Classes começam pequenas e crescem conforme o programa evolui
- É mentalmente mais fácil adicionar funcionalidade a uma classe existente do que criar uma nova
- O acúmulo é gradual — ninguém percebe até que a classe já virou um monolito

---

## 2. Sinais de alerta (gatilhos para acionar este SKILL)

- [ ] Classe com mais de 200 linhas
- [ ] Classe com mais de 10 métodos públicos
- [ ] Classe com mais de 5 atributos de instância no `__init__`
- [ ] Classe com responsabilidades claramente distintas (violação de SRP)
- [ ] Classe que é modificada por razões diferentes
- [ ] Muitos imports no topo do arquivo que não se relacionam
- [ ] Nome da classe é genérico demais (Manager, Processor, Handler, Utils)
- [ ] Atributos que são usados apenas por um subconjunto dos métodos

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                               | Técnica indicada        |
|----------------------------------------------------|-------------------------|
| Comportamentos agrupáveis em unidade autônoma      | Extract Class           |
| Funcionalidade especializada ou raramente usada    | Extract Subclass        |
| Clientes usam apenas parte da interface            | Extract Interface       |
| Classe GUI com dados e lógica misturados           | Duplicate Observed Data |

**Regra de ouro:** Se você consegue descrever a classe com a conjunção "e"
(ex: "essa classe valida pedidos **e** calcula frete **e** envia email"),
ela tem responsabilidades demais.

---

## 4. Exemplo

**ANTES — não aceito:**
```python
class PedidoService:
    def __init__(self, repository, email_service, estoque_service, nf_service):
        self.repository = repository
        self.email_service = email_service
        self.estoque_service = estoque_service
        self.nf_service = nf_service

    def criar(self, pedido):
        if not pedido.itens:
            raise ValueError("Sem itens")
        if pedido.cliente is None:
            raise ValueError("Sem cliente")

        total = sum(i.preco * i.quantidade for i in pedido.itens)
        pedido.total = total

        for item in pedido.itens:
            self.estoque_service.reservar(item.produto_id, item.quantidade)

        self.repository.save(pedido)

        nf = self.nf_service.gerar(pedido)
        pedido.nota_fiscal = nf

        self.email_service.enviar_confirmacao(pedido.cliente.email, pedido)

    def cancelar(self, pedido): ...
    def recalcular_frete(self, pedido): ...
    def aplicar_cupom(self, pedido, cupom): ...
    def buscar_por_cliente(self, cliente_id): ...
    def buscar_por_periodo(self, inicio, fim): ...
    def gerar_relatorio(self, pedidos): ...
```

**DEPOIS — esperado (Extract Class):**
```python
class PedidoService:
    def __init__(self, repository, validator, calculadora, reserva_estoque, notificacao):
        self.repository = repository
        self.validator = validator
        self.calculadora = calculadora
        self.reserva_estoque = reserva_estoque
        self.notificacao = notificacao

    def criar(self, pedido):
        self.validator.validar(pedido)
        pedido.total = self.calculadora.calcular(pedido)
        self.reserva_estoque.reservar(pedido)
        self.repository.save(pedido)
        self.notificacao.enviar_confirmacao(pedido)


class PedidoValidator:
    def validar(self, pedido):
        if not pedido.itens:
            raise ValueError("Sem itens")
        if pedido.cliente is None:
            raise ValueError("Sem cliente")


class CalculadoraDeTotal:
    def calcular(self, pedido):
        return sum(i.preco * i.quantidade for i in pedido.itens)


class ReservaDeEstoque:
    def __init__(self, estoque_service):
        self.estoque_service = estoque_service

    def reservar(self, pedido):
        for item in pedido.itens:
            self.estoque_service.reservar(item.produto_id, item.quantidade)


class NotificacaoDePedido:
    def __init__(self, email_service, nf_service):
        self.email_service = email_service
        self.nf_service = nf_service

    def enviar_confirmacao(self, pedido):
        nf = self.nf_service.gerar(pedido)
        pedido.nota_fiscal = nf
        self.email_service.enviar_confirmacao(pedido.cliente.email, pedido)
```

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Extrair classe sem coesão**
```python
# Não aceito — a classe extraída ainda mistura responsabilidades
class PedidoHelper:
    def validar(self, pedido): ...
    def calcular_frete(self, pedido): ...
    def gerar_relatorio(self, pedidos): ...
```

**Erro 2: Criar classes anêmicas**
```python
# Não aceito — apenas delega sem comportamento real
class PedidoValidator:
    def validate(self, pedido):
        return pedido.is_valid()
```

**Erro 3: Nomes genéricos nas classes extraídas**
```python
# Não aceito
class PedidoUtils: ...
class PedidoManager: ...
class PedidoHelper2: ...
```

---

## 6. Benefícios

- **Carga cognitiva:** Desenvolvedores não precisam memorizar atributos e métodos excessivos
- **Eliminação de duplicação:** Dividir classes grandes frequentemente elimina código redundante
- **Manutenção:** Cada classe com responsabilidade única é mais fácil de testar e modificar
- **Reutilização:** Classes menores e focadas são mais fáceis de reutilizar em outros contextos
