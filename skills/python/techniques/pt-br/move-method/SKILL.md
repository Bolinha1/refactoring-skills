# TÉCNICA: Move Method — Python

## Fonte
Baseado em: https://refactoring.guru/move-method

---

## 1. Problema

Um método é mais usado por outra classe do que pela própria classe onde está definido.
Isso cria acoplamento desnecessário e viola o princípio de coesão.

---

## 2. Solução

Declare o método na classe que o usa com mais frequência. Mova o código original para lá.
No lugar original, substitua o corpo por uma delegação ao novo método — ou remova-o completamente
se não for mais necessário.

---

## 3. Quando aplicar

- O método acessa atributos de outra classe mais do que os da própria classe
- O método seria mais útil na classe que o consome
- Mover o método reduziria ou eliminaria dependências entre classes
- O smell **Feature Envy** está presente (método "inveja" os dados de outra classe)

---

## 4. Passos de refatoração

1. Analise as dependências do método dentro da classe atual
2. Verifique se o método é sobrescrito em subclasses (evite quebrar polimorfismo)
3. Declare o método na classe de destino com nome contextualmente adequado
4. Obtenha uma referência à classe de destino (via atributo, parâmetro ou instância local)
5. Transforme o método original em uma delegação — ou delete-o se não houver chamadores externos
6. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
class Conta:
    def __init__(self, saldo: float, contrato: "TipoContrato"):
        self.saldo = saldo
        self.contrato = contrato

    def calcular_encargo(self) -> float:
        # esse método usa quase só dados de TipoContrato — está no lugar errado
        if self.contrato.tipo == "especial":
            return self.contrato.taxa_especial * self.saldo * 30
        return self.contrato.taxa_padrao * self.saldo * 30


class TipoContrato:
    def __init__(self, tipo: str, taxa_padrao: float, taxa_especial: float):
        self.tipo = tipo
        self.taxa_padrao = taxa_padrao
        self.taxa_especial = taxa_especial
```

**DEPOIS — esperado:**
```python
class Conta:
    def __init__(self, saldo: float, contrato: "TipoContrato"):
        self.saldo = saldo
        self.contrato = contrato

    def calcular_encargo(self) -> float:
        # delega para quem realmente tem os dados
        return self.contrato.calcular_encargo(self.saldo)


class TipoContrato:
    def __init__(self, tipo: str, taxa_padrao: float, taxa_especial: float):
        self.tipo = tipo
        self.taxa_padrao = taxa_padrao
        self.taxa_especial = taxa_especial

    def calcular_encargo(self, saldo: float) -> float:
        if self.tipo == "especial":
            return self.taxa_especial * saldo * 30
        return self.taxa_padrao * saldo * 30
```

**Por que esse padrão:**
- `calcular_encargo` usava `contrato.tipo`, `contrato.taxa_especial` e `contrato.taxa_padrao`
- O método pertence a `TipoContrato` — é lá que estão os dados necessários
- `Conta` agora apenas coordena, sem precisar conhecer os detalhes de `TipoContrato`

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover e criar dependência mútua entre as classes**
```python
# Não aceito — cria acoplamento circular
class TipoContrato:
    def calcular_encargo(self, conta: Conta) -> float:
        return self.taxa * conta.saldo * conta.dias  # agora depende de Conta
```

**Erro 2: Mover método sobrescrito em subclasses sem avaliar o impacto**
```python
# Não aceito — se calcular_encargo() é sobrescrito em ContaEspecial,
# mover sem cuidado quebra o contrato polimórfico
```

**Erro 3: Mover e manter a versão original com lógica duplicada**
```python
# Não aceito — dois métodos com o mesmo comportamento em classes diferentes
```

---

## 7. Benefícios

- **Coesão:** Cada classe contém os métodos que pertencem aos seus dados
- **Acoplamento reduzido:** Elimina dependências desnecessárias entre classes
- **Manutenção:** Mudanças na lógica afetam apenas a classe correta
