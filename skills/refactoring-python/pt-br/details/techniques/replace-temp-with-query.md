# TÉCNICA: Replace Temp with Query — Python

## Fonte
Baseado em: https://refactoring.guru/replace-temp-with-query

---

## 1. Problema

Uma variável local armazena o resultado de uma expressão. Esse valor é usado mais adiante no método, mas a variável impede que a expressão seja reutilizada em outros métodos ou sobrescrita em subclasses. O método cresce porque carrega estado que poderia ser movido para um método de consulta reutilizável.

---

## 2. Solução

Extraia a expressão para um método. Substitua cada uso da variável local por uma chamada ao novo método.

---

## 3. Quando aplicar

- A mesma variável local é calculada no início e usada bem mais abaixo, tornando o método longo
- A expressão seria útil em outros métodos da mesma classe
- A variável é somente leitura (nunca reatribuída após a inicialização)
- Uma subclasse pode querer sobrescrever a expressão com lógica diferente

---

## 4. Passos de refatoração

1. Verifique que a expressão não tem efeitos colaterais
2. Extraia a expressão para um método privado com nome claro
3. Substitua cada referência à variável local por uma chamada ao novo método
4. Remova a variável local
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
def preco(self, pedido) -> float:
    preco_base = pedido.quantidade * pedido.preco_unitario
    fator_desconto = 0.95 if preco_base > 1000 else 0.98
    return preco_base * fator_desconto
```

**DEPOIS — esperado:**
```python
def preco(self, pedido) -> float:
    return self._preco_base(pedido) * self._fator_desconto(pedido)

def _preco_base(self, pedido) -> float:
    return pedido.quantidade * pedido.preco_unitario

def _fator_desconto(self, pedido) -> float:
    return 0.95 if self._preco_base(pedido) > 1000 else 0.98
```

**Por que esse padrão:**
- `_preco_base()` e `_fator_desconto()` são métodos de consulta reutilizáveis
- Uma subclasse pode sobrescrever `_fator_desconto()` para aplicar uma regra de precificação diferente

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Aplicar a técnica quando a expressão tem efeitos colaterais**
```python
# Não aceito — um método chamado múltiplas vezes não deve ter efeitos colaterais
def _proximo_id(self) -> int:
    self._contador += 1
    return self._contador  # efeito colateral; chamá-lo duas vezes dá resultados diferentes
```

**Erro 2: Criar um método de consulta que depende inteiramente de variáveis locais**
```python
# Não aceito — se a expressão depende de muitas variáveis locais que todas
# precisariam virar parâmetros, Extract Method é mais apropriado que Replace Temp with Query
```

---

## 7. Benefícios

- **Reuso:** O método extraído pode ser chamado a partir de outros métodos da mesma classe
- **Extensibilidade:** Subclasses podem sobrescrever métodos de consulta individuais
- **Legibilidade:** O método principal lê como uma fórmula de alto nível com subexpressões nomeadas
