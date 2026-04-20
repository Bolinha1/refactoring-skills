# TÉCNICA: Replace Conditional with Polymorphism — Java

## Fonte
Baseado em: https://refactoring.guru/replace-conditional-with-polymorphism

---

## 1. Problema

Você tem um `switch` ou cadeia de `if/else` que executa comportamentos diferentes
dependendo do tipo do objeto ou de uma propriedade que simula um tipo.

---

## 2. Solução

Crie subclasses correspondentes a cada ramo do condicional. Em cada subclasse,
implemente um método que contém o comportamento daquele ramo. Substitua o condicional
por uma chamada polimórfica ao método.

---

## 3. Quando aplicar

- O condicional verifica o tipo do objeto (`instanceof`, campo tipo, constante)
- O mesmo condicional aparece em múltiplos lugares
- Adicionar um novo tipo exigiria modificar todos os condicionais existentes (violação do Open/Closed Principle)
- Cada ramo do condicional representa um comportamento coeso e distinto

---

## 4. Passos de refatoração

1. Se o condicional está dentro de um método com outras responsabilidades, aplique **Extract Method** primeiro
2. Crie uma classe base (ou interface) com o método a ser polimorfizado
3. Para cada ramo do condicional, crie uma subclasse que sobrescreve o método com o comportamento daquele ramo
4. Mova a lógica do ramo para o método da subclasse correspondente
5. Remova o ramo do condicional original
6. Repita até o condicional estar vazio, então declare o método como `abstract` na classe base
7. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public class Funcionario {
    private String tipo;   // "ENGENHEIRO", "GERENTE", "VENDEDOR"
    private double salarioBase;
    private double vendas;

    public double calcularBonus() {
        switch (tipo) {
            case "ENGENHEIRO":
                return salarioBase * 0.10;
            case "GERENTE":
                return salarioBase * 0.20 + 1000;
            case "VENDEDOR":
                return vendas * 0.05;
            default:
                throw new IllegalStateException("Tipo desconhecido: " + tipo);
        }
    }

    public String gerarRelatorio() {
        switch (tipo) {
            case "ENGENHEIRO":
                return "Engenheiro: projetos entregues";
            case "GERENTE":
                return "Gerente: equipes lideradas";
            case "VENDEDOR":
                return "Vendedor: R$ " + vendas + " em vendas";
            default:
                throw new IllegalStateException("Tipo desconhecido: " + tipo);
        }
    }
}
```

**DEPOIS — esperado:**
```java
public abstract class Funcionario {
    protected double salarioBase;

    public abstract double calcularBonus();
    public abstract String gerarRelatorio();
}

public class Engenheiro extends Funcionario {
    @Override
    public double calcularBonus() {
        return salarioBase * 0.10;
    }

    @Override
    public String gerarRelatorio() {
        return "Engenheiro: projetos entregues";
    }
}

public class Gerente extends Funcionario {
    @Override
    public double calcularBonus() {
        return salarioBase * 0.20 + 1000;
    }

    @Override
    public String gerarRelatorio() {
        return "Gerente: equipes lideradas";
    }
}

public class Vendedor extends Funcionario {
    private double vendas;

    @Override
    public double calcularBonus() {
        return vendas * 0.05;
    }

    @Override
    public String gerarRelatorio() {
        return "Vendedor: R$ " + vendas + " em vendas";
    }
}
```

**Por que esse padrão:**
- Adicionar um novo tipo de funcionário → criar nova subclasse, sem tocar nas existentes
- Sem `switch` duplicado em `calcularBonus` e `gerarRelatorio`
- Cada classe tem responsabilidade única sobre seu próprio comportamento

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover o switch para dentro de um Factory e achar que resolveu**
```java
// Não aceito — o switch continua existindo, só mudou de lugar
public class FuncionarioFactory {
    public static Funcionario criar(String tipo) {
        switch (tipo) {
            case "ENGENHEIRO": return new Funcionario("ENGENHEIRO"); // mesma classe
            // ...
        }
    }
}
```

**Erro 2: Criar subclasses mas manter condicional nos métodos**
```java
// Não aceito — criou herança mas não polimorfizou o comportamento
public class Engenheiro extends Funcionario {
    @Override
    public double calcularBonus() {
        if (tipo.equals("ENGENHEIRO")) {    // condicional desnecessário
            return salarioBase * 0.10;
        }
        return 0;
    }
}
```

**Erro 3: Usar polimorfismo para condicionais de negócio simples**
```java
// Não aceito — criar hierarquia para um if trivial é over-engineering
// Ex: if (ativo) { ... } else { ... }
```

---

## 7. Benefícios

- **Open/Closed Principle:** Novos tipos não exigem modificação do código existente
- **Eliminação de duplicação:** O mesmo condicional não precisa ser repetido
- **Legibilidade:** Cada classe expressa claramente seu comportamento
- **Testabilidade:** Cada subclasse pode ser testada isoladamente
