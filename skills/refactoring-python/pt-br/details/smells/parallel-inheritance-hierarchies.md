# SMELL: Parallel Inheritance Hierarchies — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/parallel-inheritance-hierarchies

---

## 1. O que é?

Toda vez que você adiciona uma subclasse a uma hierarquia de classes, é forçado a adicionar uma subclasse correspondente a outra hierarquia. As duas árvores espelham uma à outra: adicionar `UsuarioPremium` em uma árvore significa adicionar `RelatorioUsuarioPremium` em outra.

---

## 2. Sinais de alerta

- [ ] Nomes de classes em duas hierarquias compartilham um padrão de prefixo ou sufixo comum
- [ ] Adicionar uma subclasse na hierarquia A sempre exige uma subclasse correspondente na hierarquia B
- [ ] Um dicionário de despacho mapeia tipos da hierarquia A para tipos da hierarquia B
- [ ] Ambas as hierarquias crescem em sincronia a cada nova funcionalidade
- [ ] Uma hierarquia contém "entidades" enquanto a outra contém "comportamento" ou "representação" — mas elas precisam sempre corresponder

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Move Method** | Mova o comportamento da hierarquia paralela para a hierarquia principal, eliminando a necessidade da segunda árvore |
| **Move Field** | Mova os dados que vivem na classe paralela de volta para a classe primária |
| **Inline Class** | Colapse uma classe da hierarquia paralela em sua classe primária correspondente quando a classe paralela é pequena |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
import math

# Hierarquia A: formas
class Forma:
    def area(self) -> float: ...

class Circulo(Forma):
    def area(self) -> float: return math.pi * self.r ** 2

class Retangulo(Forma):
    def area(self) -> float: return self.l * self.a


# Hierarquia B: renderizadores — espelha a hierarquia A exatamente
class RenderizadorForma:
    def renderizar(self, forma: Forma) -> None: ...

class RenderizadorCirculo(RenderizadorForma):
    def renderizar(self, forma: Forma) -> None: pass  # desenha círculo

class RenderizadorRetangulo(RenderizadorForma):
    def renderizar(self, forma: Forma) -> None: pass  # desenha retângulo


_RENDERIZADORES = {Circulo: RenderizadorCirculo, Retangulo: RenderizadorRetangulo}

def renderizador_para(forma: Forma) -> RenderizadorForma:
    return _RENDERIZADORES[type(forma)]()
```

**DEPOIS — esperado:**
```python
import math
from abc import ABC, abstractmethod

# Mescle a renderização na hierarquia de formas
class Forma(ABC):
    @abstractmethod
    def area(self) -> float: ...

    @abstractmethod
    def renderizar(self) -> None: ...  # cada forma se renderiza


class Circulo(Forma):
    def area(self) -> float: return math.pi * self.r ** 2
    def renderizar(self) -> None: pass  # desenha círculo


class Retangulo(Forma):
    def area(self) -> float: return self.l * self.a
    def renderizar(self) -> None: pass  # desenha retângulo


for forma in formas:
    forma.renderizar()
```

**Por que este padrão:**
- Adicionar `Triangulo` requer apenas uma nova classe, não duas
- O dicionário de despacho desaparece — cada `Forma` se renderiza sozinha

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter ambas as hierarquias e usar um dicionário de registro**
```python
# Não aceito — um registro que mapeia Forma → Renderizador é apenas a fábrica disfarçada;
# o problema das hierarquias paralelas permanece
RENDERIZADORES: dict[type, type] = {Circulo: RenderizadorCirculo, Retangulo: RenderizadorRetangulo}
```

**Erro 2: Mover apenas parte da hierarquia paralela**
```python
# Não aceito — o colapso parcial deixa um design inconsistente onde algumas formas
# se renderizam e outras ainda dependem de classes renderizadoras externas
```

---

## 6. Benefícios

- **Lugar único para mudanças:** Um novo tipo de forma vive inteiramente em uma classe
- **Sem fardo de sincronização:** As duas árvores nunca podem ficar fora de sincronia porque há apenas uma
- **Despacho mais simples:** O polimorfismo substitui o mapeamento condicional de tipos
