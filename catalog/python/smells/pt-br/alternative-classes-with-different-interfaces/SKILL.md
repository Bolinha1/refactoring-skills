# SMELL: Alternative Classes with Different Interfaces — Python

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
| **Extract Superclass** | Uma vez que as interfaces correspondam, puxe o contrato compartilhado para uma classe base abstrata ou Protocol |

---

## 4. Exemplo

**ANTES — não aceito:**
```python
# Dois loggers com interfaces completamente diferentes para a mesma operação
class LoggerArquivo:
    def escrever_log(self, mensagem: str) -> None:
        # escreve em arquivo
        pass

class LoggerBancoDados:
    def persistir_entrada(self, texto: str, nivel: str) -> None:
        # escreve no banco de dados
        pass

# Cliente precisa bifurcar no tipo concreto
class ServicoDeOrdem:
    def __init__(self, usar_bd: bool):
        self._logger_arquivo = LoggerArquivo()
        self._logger_bd = LoggerBancoDados()
        self._usar_bd = usar_bd

    def processar_ordem(self, ordem) -> None:
        if self._usar_bd:
            self._logger_bd.persistir_entrada(str(ordem), "INFO")
        else:
            self._logger_arquivo.escrever_log(str(ordem))
```

**DEPOIS — esperado:**
```python
from abc import ABC, abstractmethod

# Interface compartilhada usando ABC ou Protocol
class Logger(ABC):
    @abstractmethod
    def registrar(self, mensagem: str, nivel: str) -> None: ...


class LoggerArquivo(Logger):
    def registrar(self, mensagem: str, nivel: str) -> None:
        # escreve em arquivo
        pass


class LoggerBancoDados(Logger):
    def registrar(self, mensagem: str, nivel: str) -> None:
        # escreve no banco de dados
        pass


# Cliente depende apenas da interface
class ServicoDeOrdem:
    def __init__(self, logger: Logger):
        self._logger = logger

    def processar_ordem(self, ordem) -> None:
        self._logger.registrar(str(ordem), "INFO")
```

**Por que este padrão:**
- O cliente não precisa mais saber qual logger possui
- Adicionar um terceiro logger não requer nenhuma alteração em `ServicoDeOrdem`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Envolver uma classe em um adaptador apenas para evitar renomeação**
```python
# Não aceito — adiciona indireção sem corrigir o problema raiz
class AdaptadorLoggerBD(LoggerArquivo):
    def __init__(self):
        self._interno = LoggerBancoDados()
    def escrever_log(self, mensagem: str) -> None:
        self._interno.persistir_entrada(mensagem, "INFO")  # delegação oculta
```

**Erro 2: Manter o condicional e adicionar mais ramificações**
```python
# Não aceito — cada novo tipo de logger adiciona mais uma ramificação
if tipo_log == "arquivo":
    logger_arquivo.escrever_log(msg)
elif tipo_log == "bd":
    logger_bd.persistir_entrada(msg, "INFO")
elif tipo_log == "nuvem":
    logger_nuvem.enviar(msg)
```

---

## 6. Benefícios

- **Intercambiabilidade:** Classes com a mesma interface podem ser trocadas sem alterar os chamadores
- **Redução de duplicação:** Comportamento compartilhado pode ser movido para um supertipo comum
- **Extensibilidade:** Novas implementações precisam apenas satisfazer o contrato compartilhado
