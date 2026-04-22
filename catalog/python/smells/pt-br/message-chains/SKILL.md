# SMELL: Message Chains — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/message-chains

---

## 1. O que é?

Um cliente pede a um objeto outro objeto, que pede a outro objeto ainda, formando uma cadeia: `a.get_b().get_c().get_d().fazer_algo()`. O cliente está acoplado a toda a cadeia de intermediários. Se qualquer passo mudar sua estrutura, o cliente quebra.

---

## 2. Sinais de alerta

- [ ] Chamadas de método são encadeadas três ou mais níveis de profundidade: `pedido.get_cliente().get_endereco().get_cidade()`
- [ ] A mesma cadeia aparece em múltiplos lugares no código
- [ ] É necessário navegar por uma cadeia de objetos apenas para obter um único valor
- [ ] Uma cadeia aparece em um loop ou condicional, amplificando o acoplamento
- [ ] Adicionar uma camada entre dois objetos na cadeia requer atualizar cada ponto de chamada

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Hide Delegate** | Adicione um método de atalho no primeiro objeto da cadeia para que clientes não precisem navegar pelos intermediários |
| **Extract Method** | Extraia a cadeia para um método nomeado que oculte a navegação |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
# Cliente navega por todo o grafo de objetos apenas para obter um nome de cidade
class ServicoFatura:
    def get_cidade_entrega(self, pedido) -> str:
        return pedido.get_cliente().get_endereco().get_cidade()

    def is_entrega_local(self, pedido) -> bool:
        return pedido.get_cliente().get_endereco().get_cidade() == "São Paulo"
```

**DEPOIS — esperado:**
```python
# Adicionar um atalho em Pedido (Hide Delegate)
class Pedido:
    def get_cidade_entrega(self) -> str:
        return self._cliente.get_endereco().get_cidade()


# Cliente não conhece mais Cliente nem Endereco
class ServicoFatura:
    def get_cidade_entrega(self, pedido) -> str:
        return pedido.get_cidade_entrega()

    def is_entrega_local(self, pedido) -> bool:
        return pedido.get_cidade_entrega() == "São Paulo"
```

**Por que este padrão:**
- `ServicoFatura` está desacoplado de `Cliente` e `Endereco`
- Renomear `Endereco.get_cidade()` requer apenas uma mudança em vez de muitas

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Atribuir a cadeia a uma variável local sem quebrar a dependência**
```python
# Não aceito — o acoplamento ainda está lá; apenas a sintaxe mudou
endereco = pedido.get_cliente().get_endereco()
cidade = endereco.get_cidade()
```

**Erro 2: Adicionar um método que retorna o objeto intermediário**
```python
# Não aceito — agora os chamadores encadeiam no objeto retornado; mesmo problema
def get_cliente(self): return self._cliente
```

---

## 6. Benefícios

- **Menor acoplamento:** Clientes dependem apenas de seu colaborador direto
- **Refatoração mais fácil:** A navegação interna pode ser alterada sem atualizar cada ponto de chamada
- **Mais legível:** `pedido.get_cidade_entrega()` é mais claro que `pedido.get_cliente().get_endereco().get_cidade()`
