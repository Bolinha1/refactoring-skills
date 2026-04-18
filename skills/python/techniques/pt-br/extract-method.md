# TÉCNICA: Extract Method — Python

## Fonte
Baseado em: https://refactoring.guru/extract-method

---

## 1. Problema

Você tem um fragmento de código que pode ser agrupado em uma unidade com sentido próprio,
mas ele está embutido em um método maior junto com outras responsabilidades.

---

## 2. Solução

Mova esse fragmento para um novo método (privado com prefixo `_`) com nome que descreva
a intenção. Substitua o trecho original por uma chamada ao novo método.

---

## 3. Quando aplicar

- O trecho de código mereceria um comentário explicativo
- O mesmo bloco de lógica aparece em mais de um lugar
- O método atual faz mais de uma coisa claramente distinta
- Existe dificuldade em dar um nome curto e preciso ao método atual

---

## 4. Passos de refatoração

1. Crie um novo método com nome que expresse a intenção do trecho
2. Copie o fragmento de código para o novo método
3. Identifique variáveis locais usadas pelo trecho:
   - Usadas apenas dentro do trecho → tornam-se variáveis locais do novo método
   - Declaradas antes e lidas dentro → tornam-se parâmetros
   - Modificadas dentro e usadas depois → o novo método deve retorná-las
4. Substitua o trecho original pela chamada ao novo método
5. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
def imprimir_fatura(self, fatura):
    # imprimir cabeçalho
    print("***********************")
    print(f"***   FATURA #{fatura.numero}   ***")
    print("***********************")

    # imprimir itens
    for item in fatura.itens:
        print(f"{item.nome}\t{item.quantidade}\t{item.preco}")

    # imprimir total
    total = sum(i.quantidade * i.preco for i in fatura.itens)
    print(f"TOTAL: R$ {total:.2f}")
```

**DEPOIS — esperado:**
```python
def imprimir_fatura(self, fatura):
    self._imprimir_cabecalho(fatura)
    self._imprimir_itens(fatura)
    self._imprimir_total(fatura)

def _imprimir_cabecalho(self, fatura):
    print("***********************")
    print(f"***   FATURA #{fatura.numero}   ***")
    print("***********************")

def _imprimir_itens(self, fatura):
    for item in fatura.itens:
        print(f"{item.nome}\t{item.quantidade}\t{item.preco}")

def _imprimir_total(self, fatura):
    total = sum(i.quantidade * i.preco for i in fatura.itens)
    print(f"TOTAL: R$ {total:.2f}")
```

**Variante — método com retorno:**
```python
# ANTES — variável temporária calculada e usada depois
def calcular_desconto(self, pedido):
    subtotal = 0
    for item in pedido.itens:
        subtotal += item.preco * item.quantidade
    # ... outras coisas ...
    return subtotal * pedido.taxa_desconto

# DEPOIS — extração com retorno
def calcular_desconto(self, pedido):
    subtotal = self._calcular_subtotal(pedido)
    return subtotal * pedido.taxa_desconto

def _calcular_subtotal(self, pedido):
    return sum(i.preco * i.quantidade for i in pedido.itens)
```

**Variante — função standalone (sem estado de classe):**
```python
# Quando o trecho extraído não depende de self, pode ser função de módulo
def _calcular_subtotal(itens):
    return sum(i.preco * i.quantidade for i in itens)

class PedidoService:
    def calcular_desconto(self, pedido):
        subtotal = _calcular_subtotal(pedido.itens)
        return subtotal * pedido.taxa_desconto
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Nome vago que não expressa intenção**
```python
# Não aceito
def _processar_parte1(self, fatura): ...
def _helper(self, fatura): ...
```

**Erro 2: Extrair trecho muito pequeno sem ganho de legibilidade**
```python
# Não aceito — uma linha não precisa virar método
def _imprimir_nova_linha(self):
    print()
```

**Erro 3: Deixar o método original ainda grande após extração**
```python
# Não aceito — extraiu um trecho mas o método ainda tem 50 linhas
def imprimir_fatura(self, fatura):
    self._imprimir_cabecalho(fatura)
    # ... 40 linhas restantes sem extração ...
```

---

## 7. Benefícios

- **Legibilidade:** O método principal vira uma narrativa de alto nível
- **Reuso:** O método extraído pode ser reutilizado em outros contextos
- **Testabilidade:** Métodos menores são mais fáceis de testar isoladamente
- **Isolamento de erros:** Mudanças afetam apenas o método que as contém
