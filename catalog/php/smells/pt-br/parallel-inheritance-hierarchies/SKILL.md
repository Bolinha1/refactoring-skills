# SMELL: Parallel Inheritance Hierarchies — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/parallel-inheritance-hierarchies

---

## 1. O que é?

Toda vez que você adiciona uma subclasse a uma hierarquia de classes, é forçado a adicionar uma subclasse correspondente a outra hierarquia. As duas árvores espelham uma à outra: adicionar `UsuarioPremium` em uma árvore significa adicionar `RelatorioUsuarioPremium` em outra.

---

## 2. Sinais de alerta

- [ ] Nomes de classes em duas hierarquias compartilham um padrão de prefixo ou sufixo comum
- [ ] Adicionar uma subclasse na hierarquia A sempre exige uma subclasse correspondente na hierarquia B
- [ ] Uma fábrica ou switch mapeia tipos da hierarquia A para tipos da hierarquia B
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
```php
// Hierarquia A: formas
abstract class Forma { abstract public function area(): float; }
class Circulo extends Forma { public function area(): float { return M_PI * $this->r ** 2; } }
class Retangulo extends Forma { public function area(): float { return $this->l * $this->a; } }

// Hierarquia B: renderizadores — espelha a hierarquia A exatamente
abstract class RenderizadorForma { abstract public function renderizar(Forma $f): void; }
class RenderizadorCirculo extends RenderizadorForma { public function renderizar(Forma $f): void { /* desenha círculo */ } }
class RenderizadorRetangulo extends RenderizadorForma { public function renderizar(Forma $f): void { /* desenha retângulo */ } }

function renderizadorPara(Forma $f): RenderizadorForma {
    return match(true) {
        $f instanceof Circulo   => new RenderizadorCirculo(),
        $f instanceof Retangulo => new RenderizadorRetangulo(),
        default                 => throw new \InvalidArgumentException('Forma desconhecida'),
    };
}
```

**DEPOIS — esperado:**
```php
// Mescle a renderização na hierarquia de formas
abstract class Forma {
    abstract public function area(): float;
    abstract public function renderizar(): void;
}

class Circulo extends Forma {
    public function area(): float { return M_PI * $this->r ** 2; }
    public function renderizar(): void { /* desenha círculo */ }
}

class Retangulo extends Forma {
    public function area(): float { return $this->l * $this->a; }
    public function renderizar(): void { /* desenha retângulo */ }
}

array_walk($formas, fn(Forma $f) => $f->renderizar());
```

**Por que este padrão:**
- Adicionar `Triangulo` requer apenas uma nova classe, não duas
- O mapeamento da fábrica desaparece — cada `Forma` se renderiza sozinha

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter ambas as hierarquias e adicionar um padrão bridge**
```php
// Não aceito — bridge é um padrão válido mas aqui perpetua as árvores paralelas;
// use bridge apenas quando a variação em duas dimensões é genuinamente independente
```

**Erro 2: Mover apenas parte da hierarquia paralela**
```php
// Não aceito — o colapso parcial deixa um design inconsistente
```

---

## 6. Benefícios

- **Lugar único para mudanças:** Um novo tipo de forma vive inteiramente em uma classe
- **Sem fardo de sincronização:** As duas árvores nunca podem ficar fora de sincronia porque há apenas uma
- **Fábrica/despacho mais simples:** O polimorfismo substitui o mapeamento condicional de tipos
