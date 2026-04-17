# TÉCNICA: Move Method — Java

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

- O método acessa dados de outra classe mais do que os dados da própria classe
- O método seria mais útil na classe que o consome
- Mover o método reduziria ou eliminaria dependências entre classes
- O smell **Feature Envy** está presente (método "inveja" os dados de outra classe)

---

## 4. Passos de refatoração

1. Analise as dependências do método dentro da classe atual
2. Verifique se o método existe em superclasses ou subclasses (evite quebrar polimorfismo)
3. Declare o método na classe de destino com nome contextualmente adequado
4. Obtenha uma referência à classe de destino (via campo, parâmetro ou criação local)
5. Transforme o método original em uma delegação — ou delete-o se não houver chamadores externos
6. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public class Conta {
    private double saldo;
    private TipoContrato contrato;

    public double calcularEncargo() {
        // esse método usa quase só dados de TipoContrato — está no lugar errado
        if (contrato.getTipo() == TipoContrato.ESPECIAL) {
            return contrato.getTaxaEspecial() * saldo * 30;
        }
        return contrato.getTaxaPadrao() * saldo * 30;
    }
}

public class TipoContrato {
    private String tipo;
    private double taxaPadrao;
    private double taxaEspecial;

    // getters...
}
```

**DEPOIS — esperado:**
```java
public class Conta {
    private double saldo;
    private TipoContrato contrato;

    public double calcularEncargo() {
        // delega para quem realmente tem os dados
        return contrato.calcularEncargo(saldo);
    }
}

public class TipoContrato {
    private String tipo;
    private double taxaPadrao;
    private double taxaEspecial;

    public double calcularEncargo(double saldo) {
        if (tipo.equals("ESPECIAL")) {
            return taxaEspecial * saldo * 30;
        }
        return taxaPadrao * saldo * 30;
    }
}
```

**Por que esse padrão:**
- `calcularEncargo` usava `contrato.getTipo()`, `contrato.getTaxaEspecial()` e `contrato.getTaxaPadrao()`
- O método pertence a `TipoContrato` — é lá que estão os dados necessários
- `Conta` agora apenas coordena, sem precisar conhecer os detalhes de `TipoContrato`

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover e deixar a classe original com referência circular**
```java
// Não aceito — cria dependência mútua desnecessária
public class TipoContrato {
    public double calcularEncargo(Conta conta) {
        return taxa * conta.getSaldo() * conta.getDias(); // agora depende de Conta
    }
}
```

**Erro 2: Mover método que pertence à hierarquia sem considerar polimorfismo**
```java
// Não aceito — se calcularEncargo() está em ContaBase e é sobrescrito em subclasses,
// mover sem cuidado quebra o contrato polimórfico
```

**Erro 3: Mover e não remover a versão original, criando duplicação**
```java
// Não aceito — dois métodos com o mesmo comportamento em classes diferentes
```

---

## 7. Benefícios

- **Coesão:** Cada classe contém os métodos que pertencem aos seus dados
- **Acoplamento reduzido:** Elimina dependências desnecessárias entre classes
- **Manutenção:** Mudanças na lógica afetam apenas a classe correta
