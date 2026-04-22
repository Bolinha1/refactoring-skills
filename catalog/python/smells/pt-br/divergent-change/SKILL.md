# SKILL: Detectando e Refatorando Divergent Change — Python

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/divergent-change

---

## 1. O que é Divergent Change

Uma única classe é alterada por muitas razões diferentes. Quando você se encontra modificando a mesma classe toda vez que uma nova regra de negócio, uma nova fonte de dados ou uma nova funcionalidade de preocupação não relacionada é adicionada, essa classe tem divergent change.

O inverso do Shotgun Surgery: uma classe, muitas razões para mudar.

**Por que isso acontece:**
- Responsabilidades não relacionadas foram agrupadas em uma classe por conveniência
- Uma classe começou pequena e cresceu por acumulação sem ser dividida
- Tendências de "God class": uma classe que sabe tudo e faz tudo

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Você modifica a mesma classe toda vez que uma nova tabela de banco de dados é adicionada
- [ ] Você modifica a mesma classe toda vez que um novo formato de relatório é necessário
- [ ] A classe tem métodos que se enquadram em grupos conceituais claramente distintos
- [ ] A classe tem imports de muitos módulos/subsistemas não relacionados
- [ ] Descrever o que a classe faz requer a palavra "e" mais de uma vez

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                      | Técnica recomendada     |
|----------------------------------------------------------|-------------------------|
| Classe lida com acesso a dados E lógica de negócio       | Extract Class           |
| Classe lida com múltiplos conceitos de negócio não rel.  | Extract Class           |
| Métodos que formam um cluster podem viver em outro lugar | Move Method             |
| Todos os métodos se relacionam mas a classe está grande  | Extract Subclass        |

**Regra de ouro:** cada classe deve ter exatamente uma razão para mudar.

---

## 4. Exemplo

**ANTES — não aceito:**
```python
class ServiçoPedido:
    # Razão 1: muda quando o schema do BD muda
    def buscar_por_id(self, id_pedido: int): ...   # consulta BD
    def salvar(self, pedido): ...                   # insert/update BD

    # Razão 2: muda quando as regras de negócio mudam
    def aplicar_desconto(self, pedido): ...         # lógica de desconto
    def calcular_imposto(self, pedido) -> float: ...  # lógica de imposto

    # Razão 3: muda quando o formato do relatório muda
    def gerar_fatura_pdf(self, pedido) -> bytes: ...  # geração de PDF
    def gerar_exportacao_csv(self, pedido) -> str: ...  # exportação CSV
```

**DEPOIS — esperado:**
```python
class RepositorioPedido:
    def buscar_por_id(self, id_pedido: int): ...
    def salvar(self, pedido): ...


class ServiçoPreçoPedido:
    def aplicar_desconto(self, pedido): ...
    def calcular_imposto(self, pedido) -> float: ...


class ServiçoRelatorioPedido:
    def gerar_fatura_pdf(self, pedido) -> bytes: ...
    def gerar_exportacao_csv(self, pedido) -> str: ...
```

**Por que este padrão:**
- Cada classe tem exatamente uma razão para mudar
- Um novo formato de relatório só toca `ServiçoRelatorioPedido`; uma mudança de regra de imposto só toca `ServiçoPreçoPedido`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Dividir por camada em vez de por responsabilidade**
```python
# Não aceito — CamadaDadosPedido ainda mistura schema do BD e validação de negócio
class CamadaDadosPedido: ...    # BD + validação
class ApresentacaoPedido: ...   # todos os formatos de saída
```

**Erro 2: Criar muitas micro-classes com um único método cada**
```python
# Não aceito — fragmentação excessiva que aumenta overhead de navegação
class BuscadorPedido: ...
class SalvadorPedido: ...
class AplicadorDesconto: ...
```

**Erro 3: Mover código sem mover os dados com que opera**
```python
# Não aceito — ServiçoPreçoPedido precisa chamar ServiçoPedido para obter dados
# porque os atributos necessários não foram movidos junto com os métodos
```

---

## 6. Benefícios

- **Responsabilidade Única:** Cada classe tem uma razão para mudar
- **Isolamento:** Mudanças em uma preocupação não arriscam quebrar outra
- **Descobribilidade:** Desenvolvedores sabem exatamente qual classe abrir para cada preocupação
