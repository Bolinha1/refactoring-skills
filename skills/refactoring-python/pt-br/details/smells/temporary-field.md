# SMELL: Temporary Field — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/temporary-field

---

## 1. O que é?

Uma classe contém um atributo que só é definido e usado em certas situações — durante um algoritmo ou chamada de método. No restante do tempo ele é `None`, vazio ou sem significado. Outros códigos que leem o atributo não conseguem saber quando ele tem um valor válido e quando não tem.

---

## 2. Sinais de alerta

- [ ] Um atributo é `None` ou não inicializado na maior parte do tempo
- [ ] Um atributo é definido em um método e consumido em outro, mas nunca mantido durante o tempo de vida do objeto
- [ ] Um atributo existe apenas porque um algoritmo longo precisava de um lugar para colocar estado intermediário
- [ ] A classe tem um padrão "prepare então execute" onde atributos são preenchidos logo antes do uso
- [ ] Verificações de `None` para atributos de instância aparecem em toda a classe

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Extract Class** | Mova os atributos temporários (e os métodos que os usam) para uma classe dedicada |
| **Introduce Null Object** | Substitua o estado None/vazio por um Null Object adequado para que chamadores não precisem verificar |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
# CalculadoraRota mantém atributos que só são válidos durante calcular()
class CalculadoraRota:
    def __init__(self):
        self.paradas = None               # só definido durante o cálculo
        self.distancia_total = 0          # só válido após calcular()
        self.caminho_mais_rapido = None   # None a menos que calcular() tenha executado

    def set_paradas(self, paradas: list[str]) -> None:
        self.paradas = paradas

    def calcular(self) -> None:
        self.distancia_total = self._calcular_distancia(self.paradas)
        self.caminho_mais_rapido = self._encontrar_caminho_mais_rapido(self.paradas)

    def get_caminho_mais_rapido(self) -> str | None:
        return self.caminho_mais_rapido  # None se calcular() não foi chamado primeiro
```

**DEPOIS — esperado:**
```python
from dataclasses import dataclass

# Extraia o estado temporário para um objeto de valor dedicado
@dataclass(frozen=True)
class ResultadoRota:
    distancia_total: int
    caminho_mais_rapido: str


class CalculadoraRota:
    def calcular(self, paradas: list[str]) -> ResultadoRota:
        distancia = self._calcular_distancia(paradas)
        caminho = self._encontrar_caminho_mais_rapido(paradas)
        return ResultadoRota(distancia_total=distancia, caminho_mais_rapido=caminho)
```

**Por que este padrão:**
- `ResultadoRota` só existe quando contém dados válidos — não há estado None
- `CalculadoraRota` é sem estado: fácil de reutilizar e testar em paralelo

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Adicionar uma flag `pronto` em vez de corrigir o design**
```python
# Não aceito — disciplina de flag é frágil; Extract Class remove a necessidade dela
self._calculado = False

def get_caminho_mais_rapido(self) -> str:
    if not self._calculado:
        raise RuntimeError("Chame calcular() primeiro")
    return self.caminho_mais_rapido
```

**Erro 2: Mover o atributo temporário para uma subclasse**
```python
# Não aceito — o atributo ainda viaja com o objeto; o smell permanece
class CalculadoraRotaCalculando(CalculadoraRota):
    def __init__(self):
        super().__init__()
        self.caminho_mais_rapido = None  # mesmo problema, apenas renomeado
```

---

## 6. Benefícios

- **Correção:** Sem mais estado None se passando por valor válido
- **Clareza:** Os atributos da classe são sempre significativos independente de quando você os observa
- **Testabilidade:** A classe extraída pode ser testada com dados válidos e totalmente inicializados
