# TÉCNICA: Remove Middle Man — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/remove-middle-man

---

## 1. Problema

Uma classe tem muitas propriedades ou métodos de delegação simples que não fazem nada além de encaminhar chamadas para outro objeto. A classe é um Middle Man — existe apenas para passar mensagens adiante.

---

## 2. Solução

Remova os métodos/propriedades de delegação. Deixe o cliente acessar o delegado diretamente.

---

## 3. Quando aplicar

- A classe servidor cresceu com um grande número de propriedades ou métodos delegantes de uma linha
- A classe servidor delega tanto que não tem comportamento real próprio
- Adicionar um novo recurso ao delegado requer adicionar um novo pass-through no servidor
- O smell Middle Man foi identificado na classe servidor

**Nota:** Este é o inverso do Hide Delegate. Aplique Hide Delegate quando quiser encapsulamento; aplique Remove Middle Man quando o encapsulamento se tornou um obstáculo.

---

## 4. Passos de refatoração

1. Identifique todos os métodos/propriedades de delegação na classe middle man
2. Para cada delegação, encontre seus chamadores
3. Atualize cada chamador para acessar o delegado diretamente
4. Remova cada propriedade/método de delegação do middle man
5. Se o middle man agora não tem comportamento restante, delete-o ou mescle-o com o chamador
6. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
class Pessoa:
    def __init__(self, departamento: "Departamento"):
        self._departamento = departamento

    # Propriedades pass-through puras — a classe é um middle man
    @property
    def gerente(self): return self._departamento.gerente
    @property
    def funcionarios(self): return self._departamento.funcionarios
    @property
    def nome_departamento(self): return self._departamento.nome
    @property
    def orcamento_departamento(self): return self._departamento.orcamento
```

**DEPOIS — esperado:**
```python
class Pessoa:
    def __init__(self, departamento: "Departamento"):
        self.departamento = departamento  # expõe o delegado diretamente
    # Todas as propriedades pass-through removidas


# Chamadores acessam Departamento diretamente onde precisam
gerente = pessoa.departamento.gerente
equipe = pessoa.departamento.funcionarios
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Remover o middle man mas expor objetos internos que deveriam ser escondidos**
```python
# Não aceito — se Departamento é um detalhe interno, expô-lo quebra o encapsulamento
# Remova o middle man apenas se acesso direto for apropriado para a arquitetura
```

**Erro 2: Remover propriedades de delegação que têm lógica real nelas**
```python
# Não aceito — isso não é delegação pura; tem uma guarda
@property
def gerente(self):
    if self._departamento is None:  # lógica real — não remova
        return None
    return self._departamento.gerente
```

**Erro 3: Aplicar Remove Middle Man quando Hide Delegate é na verdade o correto**
```python
# Não aceito — se clientes não deveriam conhecer Departamento de forma alguma,
# Hide Delegate é a escolha certa; Remove Middle Man vai na direção oposta
```

---

## 7. Benefícios

- **Simplicidade:** Remove indireção desnecessária
- **Transparência:** Chamadores têm um caminho direto para o que precisam
- **Manutenibilidade:** Adicionar recursos ao delegado não requer mais tocar o middle man
