# SKILL: Detectando e Refatorando Dead Code — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/dead-code

---

## 1. O que é Dead Code

Variáveis, parâmetros, campos, métodos ou classes inteiras que não são mais usados em lugar nenhum. O dead code polui a base de código, engana futuros leitores fazendo-os pensar que é relevante, e ainda precisa ser lido e compreendido mesmo não fazendo nada.

**Por que isso acontece:**
- Requisitos mudaram mas a implementação antiga não foi removida
- Uma funcionalidade foi desabilitada sem deletar o código de suporte
- Refatoração produziu caminhos de código inacessíveis que nunca foram limpos
- Medo de deletar: "pode ser que precisemos depois"

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] IDE destaca um import, campo, variável ou método não usado
- [ ] Um método é `private` e não tem chamadores na classe
- [ ] Código dentro de um branch `if` nunca pode executar (condição impossível)
- [ ] Uma classe não tem instanciações ou usos em lugar nenhum no projeto
- [ ] Blocos de código comentado que estão lá há mais de um sprint
- [ ] Um parâmetro nunca é lido dentro do método

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                           | Técnica recomendada          |
|-----------------------------------------------|------------------------------|
| Método, campo ou variável não usado           | Delete                       |
| Parâmetro não usado                           | Remove Parameter             |
| Classe inteira não usada                      | Delete                       |
| Bloco de código comentado                     | Delete (o controle de versão tem o histórico) |
| Caminho de código inacessível                 | Delete                       |

**Regra:** o controle de versão é sua rede de segurança. Delete com confiança — o código não sumiu, apenas foi arquivado.

---

## 4. Exemplo

**ANTES — não aceito:**
```java
public class ServiçoCliente {

    // Import antigo não mais necessário
    import java.util.LinkedList;

    private String idSistemaLegado;  // nunca definido ou lido desde a migração v2

    public Cliente buscarPorId(long id) { ... }

    // Esta era a busca antiga antes do ElasticSearch ser integrado
    // private Cliente buscaLinear(String nome) {
    //     for (Cliente c : todosClientes) {
    //         if (c.getNome().equals(nome)) return c;
    //     }
    //     return null;
    // }

    private void sincronizarComSistemaLegado() {
        // TODO: remover após conclusão da migração (migração foi feita há 18 meses)
    }

    public void notificarCliente(Cliente cliente, String formato, boolean legado) {
        if (legado) {
            // caminho legado — removido na v2, este branch nunca é verdadeiro
            enviarFax(cliente);
        } else {
            enviarEmail(cliente);
        }
    }
}
```

**DEPOIS — esperado:**
```java
public class ServiçoCliente {

    public Cliente buscarPorId(long id) { ... }

    public void notificarCliente(Cliente cliente) {
        enviarEmail(cliente);
    }
}
```

**Por que este padrão:**
- Cada linha que permanece é essencial; leitores podem confiar que nada está órfão
- Classes menores são mais rápidas de entender, testar e modificar

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Comentar o código em vez de deletá-lo**
```java
// Não aceito — código comentado é apenas dead code disfarçado
// public void metodoAntigo() { ... }
```

**Erro 2: Manter dead code "só por precaução"**
```java
// Não aceito — o controle de versão guarda o histórico; o medo de deletar não tem fundamento
private void migrarParaFormatoLegado() { ... } // "pode ser que precisemos depois"
```

**Erro 3: Adicionar anotação de depreciação em vez de deletar**
```java
// Não aceito — @Deprecated mantém o código visível e confuso
// Delete se não há chamadores externos
@Deprecated
public void notificarAntigo(Cliente cliente) { ... }
```

---

## 6. Benefícios

- **Sinal-para-ruído:** Cada linha restante é relevante — leitores confiam na base de código
- **Velocidade de build:** Menos classes significa ciclos mais rápidos de compilação e teste
- **Clareza:** Dead code engana — sua ausência remove trilhas falsas
