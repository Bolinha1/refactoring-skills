# SKILL: Detectando e Refatorando Divergent Change — Java

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
- [ ] A classe tem imports/dependências de muitos subsistemas não relacionados
- [ ] Descrever o que a classe faz requer a palavra "e" mais de uma vez

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                      | Técnica recomendada     |
|----------------------------------------------------------|-------------------------|
| Classe lida com acesso a dados E lógica de negócio       | Extract Class           |
| Classe lida com múltiplos conceitos de negócio não rel.  | Extract Class           |
| Métodos que formam um cluster podem viver em outro lugar | Move Method + Move Field |
| Todos os métodos se relacionam mas a classe está grande  | Extract Subclass        |

**Regra de ouro:** cada classe deve ter exatamente uma razão para mudar.

---

## 4. Exemplo

**ANTES — não aceito:**
```java
public class ServiçoPedido {
    // Razão 1: muda quando o schema do BD muda
    public Pedido buscarPorId(long id) { /* consulta JDBC */ }
    public void salvar(Pedido pedido) { /* insert/update JDBC */ }

    // Razão 2: muda quando as regras de negócio mudam
    public void aplicarDesconto(Pedido pedido) { /* lógica de desconto */ }
    public double calcularImposto(Pedido pedido) { /* lógica de imposto */ }

    // Razão 3: muda quando o formato do relatório muda
    public String gerarFaturaPdf(Pedido pedido) { /* geração de PDF */ }
    public String gerarExportacaoCsv(Pedido pedido) { /* exportação CSV */ }
}
```

**DEPOIS — esperado:**
```java
public class RepositorioPedido {
    public Pedido buscarPorId(long id) { /* consulta JDBC */ }
    public void salvar(Pedido pedido) { /* insert/update JDBC */ }
}

public class ServiçoPreçoPedido {
    public void aplicarDesconto(Pedido pedido) { /* lógica de desconto */ }
    public double calcularImposto(Pedido pedido) { /* lógica de imposto */ }
}

public class ServiçoRelatorioPedido {
    public String gerarFaturaPdf(Pedido pedido) { /* geração de PDF */ }
    public String gerarExportacaoCsv(Pedido pedido) { /* exportação CSV */ }
}
```

**Por que este padrão:**
- Cada classe tem exatamente uma razão para mudar
- Um novo formato de relatório só toca `ServiçoRelatorioPedido`; uma mudança de regra de imposto só toca `ServiçoPreçoPedido`

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Dividir por camada em vez de por responsabilidade**
```java
// Não aceito — CamadaDadosPedido ainda mistura schema do BD e validação de negócio
public class CamadaDadosPedido { /* BD + validação */ }
public class ApresentacaoPedido { /* todos os formatos de saída */ }
```

**Erro 2: Criar muitas micro-classes com um único método cada**
```java
// Não aceito — fragmentação excessiva que aumenta overhead de navegação
public class BuscadorPedido { Pedido buscar(long id); }
public class SalvadorPedido { void salvar(Pedido p); }
public class AplicadorDescountoPedido { void aplicar(Pedido p); }
```

**Erro 3: Mover código sem mover os dados com que opera**
```java
// Não aceito — ServiçoPreçoPedido precisa chamar de volta ServiçoPedido para obter dados
// porque os campos necessários não foram movidos junto com os métodos
```

---

## 6. Benefícios

- **Responsabilidade Única:** Cada classe tem uma razão para mudar
- **Isolamento:** Mudanças em uma preocupação não arriscam quebrar outra
- **Descobribilidade:** Desenvolvedores sabem exatamente qual classe abrir para cada preocupação
