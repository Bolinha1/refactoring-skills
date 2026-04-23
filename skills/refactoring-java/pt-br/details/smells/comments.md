# SKILL: Detectando e Refatorando Comments — Java

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/comments

---

## 1. O que é Comments (como code smell)

Um método está cheio de comentários explicativos porque o código em si não é claro o suficiente para ser compreendido sem eles. O comentário é um sintoma: ele marca um lugar onde o código deveria falar por si mesmo, mas não consegue.

**Importante:** Nem todo comentário é um smell. Comentários que explicam *por que* uma decisão não óbvia foi tomada são valiosos. Comentários que explicam *o que* o código faz são o smell — eles deveriam ser substituídos por nomes melhores e métodos menores.

**Por que isso acontece:**
- O código foi escrito para funcionar, não para comunicar
- Lógica complexa foi deixada inline em vez de ser extraída para métodos nomeados
- Nomes de variáveis e métodos não expressam intenção

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Um comentário descreve *o que* um bloco de código faz (não *por que*)
- [ ] Um comentário precede um bloco que poderia ser nomeado e extraído
- [ ] Uma variável precisa de um comentário para explicar o que ela contém
- [ ] Um corpo de método tem divisores de seção (`// --- etapa 1 ---`)
- [ ] Um comentário reformula o nome do método em prosa

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                    | Técnica recomendada       |
|--------------------------------------------------------|---------------------------|
| Comentário descreve um bloco de código                 | Extract Method            |
| Comentário explica o que uma variável contém           | Renomear variável ou Extract Variable |
| Comentário explica uma condição complexa               | Extract Method ou Introduce Assertion |
| Comentário marca um TODO que deveria ser código        | Introduce Assertion        |
| Comentário explica POR QUÊ (restrição não óbvia)       | Manter o comentário — ele adiciona valor |

---

## 4. Exemplo

**ANTES — não aceito:**
```java
public void processarPagamento(Pagamento pagamento) {
    // validar que o pagamento não expirou
    if (pagamento.getDataExpiracao().isBefore(LocalDate.now())) {
        throw new PagamentoExpiradoException("Pagamento expirado");
    }

    // calcular valor líquido após taxas
    double taxa = pagamento.getValor() * 0.025;
    double valorLiquido = pagamento.getValor() - taxa;

    // enviar para o gateway de pagamento
    gateway.enviar(pagamento.getNumeroCartao(), valorLiquido);

    // notificar cliente
    emailService.enviarConfirmacaoPagamento(pagamento.getEmailCliente(), valorLiquido);
}
```

**DEPOIS — esperado:**
```java
public void processarPagamento(Pagamento pagamento) {
    validarPagamentoNaoExpirado(pagamento);
    double valorLiquido = calcularValorLiquido(pagamento);
    gateway.enviar(pagamento.getNumeroCartao(), valorLiquido);
    emailService.enviarConfirmacaoPagamento(pagamento.getEmailCliente(), valorLiquido);
}

private void validarPagamentoNaoExpirado(Pagamento pagamento) {
    if (pagamento.getDataExpiracao().isBefore(LocalDate.now())) {
        throw new PagamentoExpiradoException("Pagamento expirado");
    }
}

private double calcularValorLiquido(Pagamento pagamento) {
    double taxa = pagamento.getValor() * TAXA_PROCESSAMENTO;
    return pagamento.getValor() - taxa;
}
```

**Por que este padrão:**
- Nomes de métodos substituem comentários — `validarPagamentoNaoExpirado` é mais preciso que `// validar`
- O método principal lê como uma narrativa de negócio, não como detalhes de implementação

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Manter o comentário após extrair**
```java
// Não aceito — o comentário e o nome do método dizem a mesma coisa
// validar pagamento
validarPagamento(pagamento);
```

**Erro 2: Extrair com um nome vago**
```java
// Não aceito — o nome do método não substitui o comentário
private void fazerValidacao(Pagamento pagamento) { ... }
```

**Erro 3: Remover comentários explicativos de POR QUÊ**
```java
// Não aceito — este comentário explica uma restrição não óbvia; deve ser mantido
// Usando floor() intencionalmente — o banco trunca frações, nunca arredonda
double taxa = Math.floor(pagamento.getValor() * TAXA * 100) / 100;
```

---

## 6. Benefícios

- **Autodocumentação:** Código que lê como prosa não precisa de comentários
- **Manutenção:** Nomes de métodos não podem ficar obsoletos como os comentários
- **Descoberta:** Métodos extraídos se tornam blocos de construção reutilizáveis
