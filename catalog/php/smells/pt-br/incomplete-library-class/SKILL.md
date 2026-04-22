# SMELL: Incomplete Library Class — PHP

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/incomplete-library-class

---

## 1. O que é?

Uma biblioteca ou classe de framework não possui um método que você precisa e, como não é possível modificar a biblioteca, você acaba colocando o comportamento ausente em uma classe utilitária, um helper estático ou uma subclasse. A lógica que pertence à biblioteca vaza para seu próprio código.

---

## 2. Sinais de alerta

- [ ] Uma classe utilitária contém métodos que existem apenas porque uma classe de biblioteca não os possui
- [ ] Você cria uma subclasse de uma classe de biblioteca apenas para adicionar um ou dois métodos ausentes
- [ ] Métodos helper estáticos recebem um objeto de biblioteca como seu único parâmetro
- [ ] As mesmas operações "ausentes" são reimplementadas em múltiplos lugares no código
- [ ] Comentários como "deveria estar na biblioteca X" aparecem próximos a métodos helper

---

## 3. Técnicas de tratamento

| Técnica | Quando usar |
|---|---|
| **Introduce Foreign Method** | Adicione a operação ausente como um método em sua própria classe, passando o objeto de biblioteca como parâmetro |
| **Introduce Local Extension** | Crie uma subclasse ou wrapper da classe de biblioteca que adiciona todos os métodos ausentes em um único lugar |

---

## 4. Exemplo

**ANTES — não aceito:**
```php
use Carbon\Carbon;

// Helper espalhado como método estático
class UtilData {
    public static function isWeekend(Carbon $data): bool {
        return $data->isWeekend();
    }

    public static function proximoDiaUtil(Carbon $data): Carbon {
        $proximo = $data->copy()->addDay();
        while ($proximo->isWeekend()) {
            $proximo->addDay();
        }
        return $proximo;
    }
}

// Uso é estranho
$proximo = UtilData::proximoDiaUtil(Carbon::now());
```

**DEPOIS — esperado:**
```php
use Carbon\Carbon;

// Introduce Local Extension: envolva Carbon com o comportamento ausente
class DataUtil {
    private Carbon $data;

    public function __construct(Carbon $data) {
        $this->data = $data->copy();
    }

    public static function agora(): self {
        return new self(Carbon::now());
    }

    public function isWeekend(): bool {
        return $this->data->isWeekend();
    }

    public function proximoDiaUtil(): self {
        $proximo = new self($this->data->copy()->addDay());
        while ($proximo->isWeekend()) {
            $proximo = new self($proximo->data->copy()->addDay());
        }
        return $proximo;
    }

    public function toCarbon(): Carbon {
        return $this->data->copy();
    }
}

// Uso lê naturalmente
$proximo = DataUtil::agora()->proximoDiaUtil();
```

**Por que este padrão:**
- Toda a lógica de data de negócios vive em `DataUtil` — sem helpers estáticos espalhados
- Adicionar mais comportamento ausente tem um lugar óbvio

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Duplicar o método ausente em múltiplas classes utilitárias**
```php
// Não aceito — UtilPedido::isWeekend(), UtilRelatorio::isWeekend(), etc. criam divergência
```

**Erro 2: Fazer monkey-patch na classe de biblioteca**
```php
// Não aceito — modificar código de vendor diretamente quebra em atualizações da biblioteca
```

---

## 6. Benefícios

- **Única fonte da verdade:** Comportamento ausente vive em uma classe de extensão
- **Pontos de chamada legíveis:** A extensão lê como métodos nativos da classe original
- **Isolado de mudanças da biblioteca:** Wrapping é mais seguro que monkey-patching quando a biblioteca evolui
