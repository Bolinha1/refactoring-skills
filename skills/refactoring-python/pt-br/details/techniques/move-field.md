# TÉCNICA: Move Field — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/move-field

---

## 1. Problema

Um atributo é usado mais por outra classe do que pela própria classe — outras classes leem ou escrevem nele com mais frequência do que a classe que o contém.

---

## 2. Solução

Crie um novo atributo na classe alvo. Atualize todas as referências para usar o novo local e então delete o atributo antigo.

---

## 3. Quando aplicar

- Métodos em outra classe referenciam o atributo mais do que a classe que o possui
- Você está fazendo Extract Class e precisa mover atributos com seus métodos
- Um atributo é uma informação que conceitualmente pertence a outra classe
- Após um Move Method, o método movido agora referencia atributos deixados na classe original

---

## 4. Passos de refatoração

1. Verifique se o atributo é usado por métodos em sua própria classe — se sim, considere mover esses métodos também (Move Method)
2. Adicione uma propriedade ou acessor para o atributo se precisar de acesso controlado
3. Adicione o atributo à classe alvo
4. Atualize a classe original para delegar acesso ao atributo à classe alvo (ou mova métodos dependentes)
5. Atualize todas as referências externas para usar a classe alvo diretamente
6. Delete o atributo original
7. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```python
class Conta:
    def __init__(self, tipo_conta: "TipoConta", taxa_juros: float):
        self.tipo_conta = tipo_conta
        self.taxa_juros = taxa_juros  # usada por lógica relacionada ao tipo

    def juros_para_valor(self, valor: float) -> float:
        return self.taxa_juros * valor


class TipoConta:
    # taxa_juros conceitualmente pertence aqui
    pass
```

**DEPOIS — esperado:**
```python
class TipoConta:
    def __init__(self, taxa_juros: float):
        self.taxa_juros = taxa_juros


class Conta:
    def __init__(self, tipo_conta: TipoConta):
        self.tipo_conta = tipo_conta

    def juros_para_valor(self, valor: float) -> float:
        return self.tipo_conta.taxa_juros * valor  # delega para TipoConta
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover o atributo mas manter um duplicado na classe original**
```python
# Não aceito — agora taxa_juros existe em Conta e TipoConta
class Conta:
    def __init__(self):
        self.taxa_juros = 0.0      # antigo — deve ser deletado
        self.tipo_conta = None
```

**Erro 2: Mover o atributo sem mover os métodos que o usam**
```python
# Não aceito — juros_para_valor ainda vive em Conta mas agora precisa chamar de volta
# TipoConta — isso introduz Feature Envy em vez de corrigi-lo
```

**Erro 3: Mover um atributo usado intensivamente pela classe original**
```python
# Não aceito — se Conta usa taxa_juros em 10 métodos e TipoConta usa em 1,
# o atributo deve ficar em Conta
```

---

## 7. Benefícios

- **Coesão:** Dados e o comportamento que os usa vivem na mesma classe
- **Encapsulamento:** A classe proprietária controla todo acesso ao atributo
- **Reduz acoplamento:** Outras classes não precisam mais alcançar a classe original para obter dados
