# SKILL: Detectando e Refatorando Long Parameter List — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/long-parameter-list

---

## 1. O que é Long Parameter List

Um método tem parâmetros demais — tipicamente mais de três ou quatro. Listas longas de parâmetros são difíceis de entender, fáceis de confundir e dolorosas de chamar. Elas frequentemente indicam que dados relacionados deveriam ser agrupados em um value object ou DTO.

**Por que isso acontece:**
- Métodos foram mesclados sem criar uma abstração adequada para os dados combinados
- Algoritmos foram ficando mais complexos ao longo do tempo, exigindo mais flags de controle
- Dados que naturalmente pertencem juntos são passados como primitivos individuais

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer um dos itens abaixo:

- [ ] Assinatura de método com 4 ou mais parâmetros
- [ ] Vários parâmetros do mesmo tipo em sequência (fácil de trocar acidentalmente)
- [ ] Parâmetros que sempre aparecem juntos em vários locais de chamada
- [ ] Parâmetros booleanos "flag" que mudam o comportamento do método
- [ ] O chamador precisa construir muitas variáveis locais só para chamar o método

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                                      | Técnica recomendada                |
|----------------------------------------------------------|------------------------------------|
| Parâmetros são todos campos de um objeto existente       | Preserve Whole Object              |
| Parâmetros representam um conceito novo não no modelo    | Introduce Parameter Object         |
| Um parâmetro pode ser computado a partir de outros       | Replace Parameter with Method Call |
| Um flag booleano muda o comportamento do método          | Dividir em dois métodos explícitos |

---

## 4. Exemplo

**ANTES — não aceito:**
```php
class ServiçoPedido
{
    public function criarPedido(
        string $idCliente,
        string $nomeCliente,
        string $emailCliente,
        string $rua,
        string $cidade,
        string $cep,
        string $pais,
        array $idsProdutos,
        ?string $codigoCupom = null
    ): Recibo {
        // ... lógica longa
    }
}
```

**DEPOIS — esperado:**
```php
class Endereco
{
    public function __construct(
        public readonly string $rua,
        public readonly string $cidade,
        public readonly string $cep,
        public readonly string $pais,
    ) {}
}

class RequisicaoPedido
{
    public function __construct(
        public readonly string $idCliente,
        public readonly string $nomeCliente,
        public readonly string $emailCliente,
        public readonly Endereco $enderecoEntrega,
        public readonly array $idsProdutos,
        public readonly ?string $codigoCupom = null,
    ) {}
}

class ServiçoPedido
{
    public function criarPedido(RequisicaoPedido $requisicao): Recibo
    {
        // lógica usa $requisicao->idCliente, $requisicao->enderecoEntrega, etc.
    }
}
```

**Por que este padrão:**
- `RequisicaoPedido` e `Endereco` são conceitos de primeira classe — podem ser validados, reutilizados e testados de forma independente
- Os chamadores constroem um objeto significativo, não uma lista de argumentos posicionais

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Agrupar parâmetros não relacionados em uma classe só para reduzir a contagem**
```php
// Não aceito — DataHolder não tem significado de domínio
public function criarPedido(DataHolder $holder): Recibo { ... }
```

**Erro 2: Usar um array como "objeto de parâmetro"**
```php
// Não aceito — perde segurança de tipos, suporte de IDE e descobribilidade
public function criarPedido(array $params): Recibo { ... }
```

**Erro 3: Manter a lista longa mas adicionar sobrecargas com valores padrão**
```php
// Não aceito — multiplica o problema sem abordar a causa
public function criarPedido(string $id, string $nome): Recibo { ... }
public function criarPedidoComEmail(string $id, string $nome, string $email): Recibo { ... }
```

---

## 6. Benefícios

- **Legibilidade:** Objetos de parâmetro nomeados tornam os locais de chamada autodocumentados
- **Segurança:** A confusão de argumentos posicionais é eliminada
- **Extensibilidade:** Adicionar um campo ao objeto de parâmetro não muda cada local de chamada
