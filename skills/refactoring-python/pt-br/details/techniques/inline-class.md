# TÉCNICA: Inline Class — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/inline-class

---

## 1. Problema

Uma classe não faz mais o suficiente para justificar sua existência. Ela tem responsabilidades demais poucas e é apenas uma indireção desnecessária.

---

## 2. Solução

Mova todas as funcionalidades da classe para outra classe e então delete-a.

---

## 3. Quando aplicar

- Após uma refatoração, uma classe ficou com apenas um ou dois métodos triviais
- Uma classe é usada em apenas um lugar e não adiciona abstração real
- Uma classe que foi dividida de forma muito agressiva precisa ser consolidada
- A classe é um wrapper fino que apenas delega tudo para outra classe

---

## 4. Passos de refatoração

1. Identifique a classe a fazer inline e a classe alvo (para onde as funcionalidades irão)
2. Para cada funcionalidade pública na classe a fazer inline:
   - Copie-a para a classe alvo
   - Atualize a classe alvo para delegar à classe inline (transição segura)
3. Atualize todos os chamadores para usar a classe alvo diretamente
4. Remova a delegação da classe alvo (métodos agora funcionam nativamente)
5. Delete a classe que foi inlined
6. Execute os testes após cada passo

---

## 5. Exemplo

**ANTES — não aceito:**
```python
class NumeroTelefone:
    def __init__(self, numero: str):
        self.numero = numero

    def get_numero(self) -> str:
        return self.numero


class Pessoa:
    def __init__(self, nome: str, telefone: NumeroTelefone):
        self.nome = nome
        self._telefone = telefone

    def get_numero_telefone(self) -> str:
        return self._telefone.get_numero()  # NumeroTelefone é apenas um wrapper
```

**DEPOIS — esperado:**
```python
class Pessoa:
    def __init__(self, nome: str, numero_telefone: str):
        self.nome = nome
        self.numero_telefone = numero_telefone  # inlined diretamente

    def get_numero_telefone(self) -> str:
        return self.numero_telefone
```

**Caso de uso: consolidando Shotgun Surgery**
```python
# ANTES — quatro classes de serviço minúsculas divididas de forma muito agressiva
class BuscadorPedido:
    def buscar(self, id_pedido: int): ...

class SalvadorPedido:
    def salvar(self, pedido): ...

class AtualizadorPedido:
    def atualizar(self, pedido): ...

class DeletadorPedido:
    def deletar(self, id_pedido: int): ...

# DEPOIS — fazer inline de todos em um único repositório coeso
class RepositorioPedido:
    def buscar(self, id_pedido: int): ...
    def salvar(self, pedido): ...
    def atualizar(self, pedido): ...
    def deletar(self, id_pedido: int): ...
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Fazer inline de uma classe que ainda tem um conceito significativo**
```python
# Não aceito — Dinheiro tem comportamento (aritmética, formatação) que pertence junto
# Não faça inline de Dinheiro em Pedido só porque tem poucos atributos
```

**Erro 2: Fazer inline sem remover a classe original**
```python
# Não aceito — tanto Pessoa.numero_telefone quanto NumeroTelefone existem simultaneamente
class Pessoa:
    def __init__(self):
        self.numero_telefone = ""         # cópia inlined
        self._telefone = NumeroTelefone(...)  # ainda aqui — cria confusão
```

**Erro 3: Fazer inline de uma classe usada em muitos lugares**
```python
# Não aceito — se NumeroTelefone é usado por Pessoa, Funcionario e Contato,
# fazer inline exigiria duplicar a lógica em três lugares
```

---

## 7. Benefícios

- **Simplicidade:** Remove indireção desnecessária da base de código
- **Legibilidade:** Menos classes para navegar quando a abstração não adiciona valor
- **Preparação:** Frequentemente precede Extract Class — fazer inline primeiro, depois re-extrair corretamente
