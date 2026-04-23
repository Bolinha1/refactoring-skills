# SMELL: Incomplete Library Class — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/incomplete-library-class

---

## 1. O que é?

Uma biblioteca ou classe de framework não possui um método que você precisa e, como não é possível modificar a biblioteca, você acaba colocando o comportamento ausente em uma classe utilitária, um helper estático ou uma subclasse. A lógica que pertence à biblioteca vaza para seu próprio código.

---

## 2. Sinais de alerta

- [ ] Uma classe utilitária contém métodos que existem apenas porque uma classe de biblioteca não os possui
- [ ] Você cria uma subclasse de uma classe de biblioteca apenas para adicionar um ou dois métodos ausentes
- [ ] Métodos helper estáticos recebem um objeto de biblioteca como seu único parâmetro
- [ ] As mesmas operações "ausentes" são reimplementadas em múltiplos lugares no código
- [ ] Comentários como "deveria estar na biblioteca X" aparecem próximos a métodos helper

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Introduce Foreign Method** | Adicione a operação ausente como um método em sua própria classe, passando o objeto de biblioteca como parâmetro |
| **Introduce Local Extension** | Crie uma subclasse ou wrapper da classe de biblioteca que adiciona todos os métodos ausentes em um único lugar |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
import java.time.LocalDate;

// Helper espalhado como método estático
public class UtilData {
    // Deveria estar em LocalDate, mas não podemos modificá-la
    public static boolean isFimDeSemana(LocalDate data) {
        return data.getDayOfWeek().getValue() >= 6;
    }

    public static LocalDate proximoDiaUtil(LocalDate data) {
        LocalDate proximo = data.plusDays(1);
        while (isFimDeSemana(proximo)) {
            proximo = proximo.plusDays(1);
        }
        return proximo;
    }
}

// Uso é estranho — sempre precisa lembrar de chamar UtilData em vez da data
LocalDate proximo = UtilData.proximoDiaUtil(LocalDate.now());
```

**DEPOIS — esperado:**
```java
import java.time.LocalDate;

// Introduce Local Extension: envolva LocalDate com o comportamento ausente
public class DataUtil {
    private final LocalDate data;

    public DataUtil(LocalDate data) { this.data = data; }

    public static DataUtil de(LocalDate data) { return new DataUtil(data); }

    public boolean isFimDeSemana() {
        return data.getDayOfWeek().getValue() >= 6;
    }

    public DataUtil proximoDiaUtil() {
        DataUtil proximo = new DataUtil(data.plusDays(1));
        while (proximo.isFimDeSemana()) {
            proximo = new DataUtil(proximo.data.plusDays(1));
        }
        return proximo;
    }

    public LocalDate toLocalDate() { return data; }
}

// Uso lê naturalmente
DataUtil proximo = DataUtil.de(LocalDate.now()).proximoDiaUtil();
```

**Por que este padrão:**
- Toda a lógica de data de negócios vive em `DataUtil` — sem helpers estáticos espalhados
- Adicionar mais comportamento ausente tem um lugar óbvio

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Duplicar o método ausente em múltiplas classes utilitárias**
```java
// Não aceito — UtilPedido.isFimDeSemana(), UtilRelatorio.isFimDeSemana(), etc.
// cria divergência quando a regra precisa mudar
```

**Erro 2: Criar subclasse de uma classe de biblioteca final**
```java
// Não aceito — muitas classes de biblioteca são final; use composição/wrapping em vez disso
class LocalDateEstendido extends LocalDate { ... } // erro de compilação se LocalDate é final
```

---

## 6. Benefícios

- **Única fonte da verdade:** Comportamento ausente vive em uma classe de extensão
- **Pontos de chamada legíveis:** A extensão lê como métodos nativos da classe original
- **Isolado de mudanças da biblioteca:** Wrapping é mais seguro que subclassing quando a biblioteca evolui
