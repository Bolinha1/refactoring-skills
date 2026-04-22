# SKILL: Detectando e Refatorando Shotgun Surgery — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/shotgun-surgery

---

## 1. O que é Shotgun Surgery

Uma única mudança lógica requer modificar muitos módulos ou classes diferentes ao mesmo tempo. Fazer uma mudança conceitual "dispara" edições pela base de código como um tiro de espingarda — tocando muitos arquivos para o que deveria ser uma correção localizada.

O inverso do Divergent Change: muitas classes, uma razão para mudar.

**Por que isso acontece:**
- Uma responsabilidade foi fragmentada em muitos módulos em vez de viver em um único lugar
- Lógica que deveria ser centralizada foi duplicada ou distribuída por "flexibilidade"
- Preocupações transversais (logging, validação, notificações) foram tratadas inline em todo lugar

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Adicionar uma nova funcionalidade força você a editar 5+ módulos/classes não relacionados
- [ ] Renomear um conceito requer tocar dezenas de arquivos
- [ ] Uma única regra de negócio é aplicada em múltiplos lugares
- [ ] Você sempre precisa fazer a mesma mudança em vários lugares simultaneamente
- [ ] Falhas de teste em cascata por muitos arquivos de teste para uma única mudança conceitual

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                            | Técnica recomendada     |
|----------------------------------------------------------------|-------------------------|
| Comportamento espalhado todo relacionado a um conceito         | Move Method             |
| Múltiplas classes pequenas contribuindo para um conceito       | Inline Class            |
| Preocupação transversal repetida em todo lugar                 | Extract Class           |
| Regra de negócio duplicada aplicada em muitos lugares          | Move Method para um dono |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
# Adicionar o conceito de "moeda" requer mudar TODAS essas classes:

class Produto:
    preco: float  # deve adicionar campo moeda aqui

class Pedido:
    def get_total(self) -> float: ...  # deve formatar com moeda

class Fatura:
    def formatar_valor(self, valor: float) -> str:
        return f"R${valor:.2f}"  # deve usar moeda

class Recibo:
    def imprimir_total(self, valor: float) -> None:
        print(valor)  # deve usar moeda
```

**DEPOIS — esperado:**
```python
from decimal import Decimal
from dataclasses import dataclass

@dataclass(frozen=True)
class Dinheiro:
    valor: Decimal
    moeda: str

    def __add__(self, outro: "Dinheiro") -> "Dinheiro":
        if self.moeda != outro.moeda:
            raise ValueError("Moedas diferentes")
        return Dinheiro(self.valor + outro.valor, self.moeda)

    def formatar(self) -> str:
        simbolos = {"BRL": "R$", "USD": "$", "EUR": "€"}
        simbolo = simbolos.get(self.moeda, self.moeda)
        return f"{simbolo}{self.valor:.2f}"


# Agora Produto, Pedido, Fatura, Recibo usam Dinheiro — um conceito, um lugar
@dataclass
class Produto:
    preco: Dinheiro
```

**Por que este padrão:**
- `Dinheiro` centraliza todo comportamento de moeda — adicionar um novo formato de moeda toca uma classe
- Todos os chamadores se beneficiam automaticamente

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Centralizar os dados mas não o comportamento**
```python
# Não aceito — DinheiroDto agrupa os campos mas formatação/aritmética ficam espalhadas
@dataclass
class DinheiroDto:
    valor: float
    codigo_moeda: str
```

**Erro 2: Usar um módulo utilitário como ponto único**
```python
# Não aceito — utils_dinheiro é um smell escondido; lógica ainda fragmentada entre chamadores
def formatar_dinheiro(valor: float, moeda: str) -> str: ...
```

**Erro 3: Fazer inline de classes agressivamente, criando uma nova god class**
```python
# Não aceito — fazer inline de todas as classes pequenas em uma megaclasse troca
# Shotgun Surgery por Divergent Change
```

---

## 6. Benefícios

- **Localidade:** Um conceito de negócio = uma classe para mudar
- **Segurança:** Mudanças são mais fáceis de raciocinar quando o blast radius é pequeno
- **Descobribilidade:** Todo comportamento de um conceito está em um lugar óbvio
