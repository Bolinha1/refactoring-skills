# SMELL: Alternative Classes with Different Interfaces — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/alternative-classes-with-different-interfaces

---

## 1. O que é?

Duas ou mais classes executam funções similares ou idênticas, mas possuem nomes e assinaturas de métodos diferentes. Como as interfaces diferem, o código cliente não pode tratá-las de forma intercambiável e precisa saber com qual classe concreta está lidando.

---

## 2. Sinais de alerta

- [ ] Duas classes fazem coisas similares mas têm nomes de métodos diferentes para operações equivalentes
- [ ] Trocar de uma implementação para outra exige alterar todos os pontos de chamada
- [ ] Um condicional (`if`/`instanceof`) seleciona entre duas classes que poderiam ser unificadas
- [ ] Uma classe foi criada como substituta de outra mas nunca compartilhou sua interface
- [ ] Lógica duplicada aparece em ambas as classes porque elas não podem compartilhar uma abstração comum

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Rename Method** | Alinhe os nomes dos métodos para que ambas as classes exponham a mesma interface |
| **Move Method** | Mova os métodos ausentes para a classe que não os possui até que ambas sejam equivalentes |
| **Extract Superclass** | Uma vez que as interfaces correspondam, puxe o contrato compartilhado para uma classe abstrata ou interface |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
// Dois loggers com interfaces completamente diferentes para a mesma operação
public class LoggerArquivo {
    public void escreverLog(String mensagem) {
        // escreve em arquivo
    }
}

public class LoggerBancoDados {
    public void persistirEntrada(String texto, String nivel) {
        // escreve no banco de dados
    }
}

// Cliente precisa bifurcar no tipo concreto
public class ServicoDeOrdem {
    private LoggerArquivo loggerArquivo;
    private LoggerBancoDados loggerBD;
    private boolean usarBD;

    public void processarOrdem(Ordem ordem) {
        if (usarBD) {
            loggerBD.persistirEntrada(ordem.toString(), "INFO");
        } else {
            loggerArquivo.escreverLog(ordem.toString());
        }
    }
}
```

**DEPOIS — esperado:**
```java
// Interface compartilhada
public interface Logger {
    void registrar(String mensagem, String nivel);
}

public class LoggerArquivo implements Logger {
    @Override
    public void registrar(String mensagem, String nivel) {
        // escreve em arquivo
    }
}

public class LoggerBancoDados implements Logger {
    @Override
    public void registrar(String mensagem, String nivel) {
        // escreve no banco de dados
    }
}

// Cliente depende apenas da interface
public class ServicoDeOrdem {
    private final Logger logger;

    public ServicoDeOrdem(Logger logger) {
        this.logger = logger;
    }

    public void processarOrdem(Ordem ordem) {
        logger.registrar(ordem.toString(), "INFO");
    }
}
```

**Por que este padrão:**
- O cliente não precisa mais saber qual logger possui
- Adicionar um terceiro logger não requer nenhuma alteração em `ServicoDeOrdem`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Envolver uma classe em um adaptador apenas para evitar renomeação**
```java
// Não aceito — adiciona indireção sem corrigir o problema raiz
public class AdaptadorLoggerBD extends LoggerArquivo {
    private LoggerBancoDados interno;
    @Override
    public void escreverLog(String mensagem) {
        interno.persistirEntrada(mensagem, "INFO"); // delegação oculta
    }
}
```

**Erro 2: Manter o condicional e adicionar mais ramificações**
```java
// Não aceito — cada novo tipo de logger adiciona mais uma ramificação
if (tipo.equals("arquivo")) { loggerArquivo.escreverLog(msg); }
else if (tipo.equals("bd")) { loggerBD.persistirEntrada(msg, "INFO"); }
else if (tipo.equals("nuvem")) { loggerNuvem.enviar(msg); }
```

---

## 6. Benefícios

- **Intercambiabilidade:** Classes com a mesma interface podem ser trocadas sem alterar os chamadores
- **Redução de duplicação:** Comportamento compartilhado pode ser movido para um supertipo comum
- **Extensibilidade:** Novas implementações precisam apenas satisfazer o contrato compartilhado
