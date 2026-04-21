# SMELL: Speculative Generality — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/speculative-generality

---

## 1. O que é?

Código escrito para lidar com requisitos que ainda não existem — e talvez nunca existam. Classes abstratas com apenas uma subclasse concreta, hooks que nunca são chamados, parâmetros que são sempre passados como `null` e interfaces com uma única implementação são todos sinais de que alguém construiu flexibilidade "por precaução".

---

## 2. Sinais de alerta

- [ ] Uma classe abstrata ou interface tem apenas uma implementação concreta
- [ ] Um método tem parâmetros que são sempre `null` ou a mesma constante em todos os pontos de chamada
- [ ] Uma classe ou método existe apenas para suportar requisitos futuros mencionados em comentários
- [ ] Um hook ou ponto de extensão nunca foi usado desde que foi introduzido
- [ ] Testes são os únicos chamadores de certos métodos

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Collapse Hierarchy** | Quando uma classe abstrata tem apenas uma subclasse concreta e elas podem ser mescladas |
| **Inline Class** | Quando uma classe existe puramente como ponto de extensão futuro sem consumidores atuais |
| **Remove Parameter** | Quando um parâmetro é sempre passado com o mesmo valor em todos os pontos de chamada |
| **Rename Method** | Quando nomes abstratos foram escolhidos para acomodar uma variedade que nunca se materializou |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
// NotificadorAbstrato existe apenas porque alguém assumiu que outros notificadores viriam
public abstract class NotificadorAbstrato {
    public abstract void enviar(String mensagem, String canal);
}

public class NotificadorEmail extends NotificadorAbstrato {
    @Override
    public void enviar(String mensagem, String canal) {
        // canal é sempre "email" em todos os pontos de chamada
        clienteEmail.enviar(mensagem);
    }
}

// Todo ponto de chamada passa "email" — o parâmetro não agrega valor
notificador.enviar("Olá", "email");
```

**DEPOIS — esperado:**
```java
public class NotificadorEmail {
    public void enviar(String mensagem) {
        clienteEmail.enviar(mensagem);
    }
}

notificador.enviar("Olá");
```

**Por que este padrão:**
- A classe abstrata e o parâmetro não utilizado eram complexidade especulativa
- Se um segundo tipo de notificador for necessário, a abstração pode ser introduzida naquele momento com requisitos reais para guiá-la

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Remover abstrações que estão genuinamente abertas para extensão**
```java
// Cuidado — se o roadmap do produto já especifica notificações por SMS e push,
// a abstração não é especulativa; mantenha-a
```

**Erro 2: Remover uma interface apenas porque tem uma implementação hoje**
```java
// Cuidado — interfaces usadas para injeção de dependência ou testes (mockability) servem
// a um propósito atual real; verifique antes de remover
```

---

## 6. Benefícios

- **Complexidade reduzida:** O código contém apenas código que atende às necessidades atuais
- **Integração mais fácil:** Desenvolvedores não precisam entender hooks e abstrações que ainda não fazem nada
- **Refatoração mais simples:** Quando requisitos reais chegam, você adiciona abstrações informadas por casos de uso reais
