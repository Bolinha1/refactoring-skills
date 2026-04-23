# SMELL: Alternative Classes with Different Interfaces — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/alternative-classes-with-different-interfaces

---

## 1. O que é?

Duas ou mais classes executam funções similares ou idênticas, mas possuem nomes e assinaturas de métodos diferentes. Como as interfaces diferem, o código cliente não pode tratá-las de forma intercambiável e precisa saber com qual classe concreta está lidando.

---

## 2. Sinais de alerta

- [ ] Duas classes fazem coisas similares mas têm nomes de métodos diferentes para operações equivalentes
- [ ] Trocar de uma implementação para outra exige alterar todos os pontos de chamada
- [ ] Um condicional seleciona entre duas classes que poderiam ser unificadas
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
```php
// Dois loggers com interfaces completamente diferentes para a mesma operação
class LoggerArquivo {
    public function escreverLog(string $mensagem): void {
        // escreve em arquivo
    }
}

class LoggerBancoDados {
    public function persistirEntrada(string $texto, string $nivel): void {
        // escreve no banco de dados
    }
}

// Cliente precisa bifurcar no tipo concreto
class ServicoDeOrdem {
    private LoggerArquivo $loggerArquivo;
    private LoggerBancoDados $loggerBD;
    private bool $usarBD;

    public function processarOrdem(Ordem $ordem): void {
        if ($this->usarBD) {
            $this->loggerBD->persistirEntrada((string) $ordem, 'INFO');
        } else {
            $this->loggerArquivo->escreverLog((string) $ordem);
        }
    }
}
```

**DEPOIS — esperado:**
```php
// Interface compartilhada
interface Logger {
    public function registrar(string $mensagem, string $nivel): void;
}

class LoggerArquivo implements Logger {
    public function registrar(string $mensagem, string $nivel): void {
        // escreve em arquivo
    }
}

class LoggerBancoDados implements Logger {
    public function registrar(string $mensagem, string $nivel): void {
        // escreve no banco de dados
    }
}

// Cliente depende apenas da interface
class ServicoDeOrdem {
    public function __construct(private readonly Logger $logger) {}

    public function processarOrdem(Ordem $ordem): void {
        $this->logger->registrar((string) $ordem, 'INFO');
    }
}
```

**Por que este padrão:**
- O cliente não precisa mais saber qual logger possui
- Adicionar um terceiro logger não requer nenhuma alteração em `ServicoDeOrdem`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Envolver uma classe em um adaptador apenas para evitar renomeação**
```php
// Não aceito — adiciona indireção sem corrigir o problema raiz
class AdaptadorLoggerBD extends LoggerArquivo {
    private LoggerBancoDados $interno;
    public function escreverLog(string $mensagem): void {
        $this->interno->persistirEntrada($mensagem, 'INFO'); // delegação oculta
    }
}
```

**Erro 2: Manter o condicional e adicionar mais ramificações**
```php
// Não aceito — cada novo tipo de logger adiciona mais uma ramificação
if ($tipo === 'arquivo') { $loggerArquivo->escreverLog($msg); }
elseif ($tipo === 'bd') { $loggerBD->persistirEntrada($msg, 'INFO'); }
elseif ($tipo === 'nuvem') { $loggerNuvem->enviar($msg); }
```

---

## 6. Benefícios

- **Intercambiabilidade:** Classes com a mesma interface podem ser trocadas sem alterar os chamadores
- **Redução de duplicação:** Comportamento compartilhado pode ser movido para um supertipo comum
- **Extensibilidade:** Novas implementações precisam apenas satisfazer o contrato compartilhado
