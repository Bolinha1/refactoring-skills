# TÉCNICA: Move Field — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/move-field

---

## 1. Problema

Um campo é usado mais por outra classe do que pela própria classe — outras classes leem ou escrevem nele com mais frequência do que a classe que o contém.

---

## 2. Solução

Crie um novo campo na classe alvo. Atualize todas as referências para usar o novo local e então delete o campo antigo.

---

## 3. Quando aplicar

- Métodos em outra classe referenciam o campo mais do que a classe que o possui
- Você está fazendo Extract Class e precisa mover campos com seus métodos
- Um campo é uma informação que conceitualmente pertence a outra classe
- Após um Move Method, o método movido agora referencia campos deixados na classe original

---

## 4. Passos de refatoração

1. Verifique se o campo é usado por métodos em sua própria classe — se sim, considere mover esses métodos também (Move Method)
2. Crie um getter e setter para o campo em sua classe atual (se ainda não existir)
3. Adicione um novo campo com o mesmo nome à classe alvo
4. Defina o novo campo na classe alvo e remova-o do original (ou redirecione via getter/setter)
5. Atualize todas as referências:
   - Na classe original, substitua acesso direto por delegação à classe alvo
   - Ou mova os métodos dependentes (Move Method) para que acessem o campo nativamente
6. Delete o campo original (e seu getter/setter se não forem mais necessários)
7. Execute os testes

---

## 5. Exemplo

**ANTES — não aceito:**
```java
public class Conta {
    private TipoConta tipo;
    private double taxaJuros; // usada por lógica relacionada ao tipo, não pela Conta diretamente

    public double jurosParaValor(double valor) {
        return taxaJuros * valor;
    }
}

public class TipoConta {
    // taxaJuros conceitualmente pertence aqui
}
```

**DEPOIS — esperado:**
```java
public class TipoConta {
    private double taxaJuros;

    public double getTaxaJuros() { return taxaJuros; }
    public void setTaxaJuros(double taxa) { this.taxaJuros = taxa; }
}

public class Conta {
    private TipoConta tipo;

    public double jurosParaValor(double valor) {
        return tipo.getTaxaJuros() * valor; // delega para TipoConta
    }
}
```

---

## 6. Exemplos negativos — o que NÃO fazer

**Erro 1: Mover o campo mas manter um duplicado na classe original**
```java
// Não aceito — agora taxaJuros existe em Conta e TipoConta
public class Conta {
    private double taxaJuros; // antigo — deve ser deletado
    private TipoConta tipo;
}
```

**Erro 2: Mover o campo sem mover os métodos que o usam**
```java
// Não aceito — jurosParaValor ainda vive em Conta mas agora precisa chamar de volta
// TipoConta — isso introduz Feature Envy em vez de corrigi-lo
```

**Erro 3: Mover um campo usado intensivamente pela classe original**
```java
// Não aceito — se Conta usa taxaJuros em 10 métodos e TipoConta usa em 1,
// o campo deve ficar em Conta
```

---

## 7. Benefícios

- **Coesão:** Dados e o comportamento que os usa vivem na mesma classe
- **Encapsulamento:** A classe proprietária controla todo acesso ao campo
- **Reduz acoplamento:** Outras classes não precisam mais alcançar a classe original para obter dados
