# SKILL: Detectando e Refatorando Dead Code — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/dead-code

---

## 1. O que é Dead Code

Variáveis, parâmetros, atributos, funções, métodos ou classes inteiras que não são mais usados em lugar nenhum. O dead code polui a base de código, engana futuros leitores fazendo-os pensar que é relevante, e ainda precisa ser lido e compreendido mesmo não fazendo nada.

**Por que isso acontece:**
- Requisitos mudaram mas a implementação antiga não foi removida
- Uma funcionalidade foi desabilitada sem deletar o código de suporte
- Refatoração produziu caminhos de código inacessíveis que nunca foram limpos
- Medo de deletar: "pode ser que precisemos depois"

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] `flake8`, `pyflakes` ou IDE destaca um import ou variável não usado
- [ ] Uma função é prefixada com `_` (privada) e não tem chamadores no módulo
- [ ] Código dentro de um branch `if` nunca pode executar (condição impossível)
- [ ] Uma classe não tem instanciações em lugar nenhum no projeto
- [ ] Blocos de código comentado que estão lá há mais de um sprint
- [ ] Um parâmetro nunca é lido dentro da função

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                           | Técnica recomendada          |
|-----------------------------------------------|------------------------------|
| Função, atributo ou variável não usada        | Delete                       |
| Parâmetro não usado                           | Remove Parameter             |
| Classe ou módulo inteiro não usado            | Delete                       |
| Bloco de código comentado                     | Delete (git tem o histórico) |
| Caminho de código inacessível                 | Delete                       |

**Regra:** o `git` é sua rede de segurança. Delete com confiança — o código não sumiu, apenas foi arquivado.

---

## 4. Exemplo

**ANTES — não aceito:**
```python
import csv           # nunca usado desde a migração para pandas
import json          # nunca usado

class ServiçoCliente:
    def __init__(self):
        self.id_sistema_legado = None  # nunca definido ou lido desde a migração v2

    def buscar_por_id(self, id_cliente: int): ...

    # Busca antiga antes do ElasticSearch ser integrado
    # def _busca_linear(self, nome: str):
    #     return next((c for c in self.todos_clientes if c.nome == nome), None)

    def _sincronizar_com_sistema_legado(self):
        # TODO: remover após conclusão da migração (migração foi feita há 18 meses)
        pass

    def notificar_cliente(self, cliente, formato: str, legado: bool = False):
        if legado:
            # caminho legado — removido na v2, este branch nunca é True
            self._enviar_fax(cliente)
        else:
            self._enviar_email(cliente)
```

**DEPOIS — esperado:**
```python
class ServiçoCliente:
    def buscar_por_id(self, id_cliente: int): ...

    def notificar_cliente(self, cliente) -> None:
        self._enviar_email(cliente)
```

**Por que este padrão:**
- Cada linha que permanece é essencial; leitores podem confiar que nada está órfão
- Classes menores são mais rápidas de entender, testar e modificar

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Comentar o código em vez de deletá-lo**
```python
# Não aceito — código comentado é dead code disfarçado
# def metodo_antigo(self): ...
```

**Erro 2: Manter dead code "só por precaução"**
```python
# Não aceito — git guarda o histórico; o medo de deletar não tem fundamento
def _migrar_para_formato_legado(self): ...  # "pode ser que precisemos depois"
```

**Erro 3: Usar warning de depreciação em vez de deletar**
```python
# Não aceito — mantém o dead code visível e confuso
import warnings
def notificar_antigo(self, cliente):
    warnings.warn("Use notificar_cliente em vez disso", DeprecationWarning)
    ...
```

---

## 6. Benefícios

- **Sinal-para-ruído:** Cada linha restante é relevante — leitores confiam na base de código
- **Velocidade de import:** Menos imports e módulos significa inicialização mais rápida
- **Clareza:** Dead code engana — sua ausência remove trilhas falsas
