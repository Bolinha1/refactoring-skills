# SKILL: Detectando e Refatorando Dead Code — PHP

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

- [ ] IDE ou análise estática (PHPStan, Psalm) destaca um import, propriedade ou método não usado
- [ ] Um método é `private` e não tem chamadores na classe
- [ ] Código dentro de um branch `if` nunca pode executar (condição impossível)
- [ ] Uma classe não tem instanciações ou usos em lugar nenhum no projeto
- [ ] Blocos de código comentado que estão lá há mais de um sprint
- [ ] Um parâmetro nunca é lido dentro do método

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada                           | Técnica recomendada          |
|-----------------------------------------------|------------------------------|
| Método, propriedade ou variável não usado     | Delete                       |
| Parâmetro não usado                           | Remove Parameter             |
| Classe inteira não usada                      | Delete                       |
| Bloco de código comentado                     | Delete (git tem o histórico) |
| Caminho de código inacessível                 | Delete                       |

**Regra:** o controle de versão é sua rede de segurança. Delete com confiança — o código não sumiu, apenas foi arquivado.

---

## 4. Exemplo

**ANTES — não aceito:**
```php
use App\Legacy\ClienteLegado; // nunca usado desde a migração

class ServiçoCliente
{
    private ?string $idSistemaLegado = null; // nunca definido ou lido desde a migração v2

    public function buscarPorId(int $id): Cliente { ... }

    // Busca antiga antes do ElasticSearch ser integrado
    // private function buscaLinear(string $nome): ?Cliente
    // {
    //     foreach ($this->todosClientes as $cliente) {
    //         if ($cliente->getNome() === $nome) return $cliente;
    //     }
    //     return null;
    // }

    private function sincronizarComSistemaLegado(): void
    {
        // TODO: remover após conclusão da migração (migração foi feita há 18 meses)
    }

    public function notificarCliente(Cliente $cliente, string $formato, bool $legado = false): void
    {
        if ($legado) {
            // caminho legado — removido na v2, este branch nunca é verdadeiro
            $this->enviarFax($cliente);
        } else {
            $this->enviarEmail($cliente);
        }
    }
}
```

**DEPOIS — esperado:**
```php
class ServiçoCliente
{
    public function buscarPorId(int $id): Cliente { ... }

    public function notificarCliente(Cliente $cliente): void
    {
        $this->enviarEmail($cliente);
    }
}
```

**Por que este padrão:**
- Cada linha que permanece é essencial; leitores podem confiar que nada está órfão
- Classes menores são mais rápidas de entender, testar e modificar

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Comentar o código em vez de deletá-lo**
```php
// Não aceito — código comentado é dead code disfarçado
// public function metodoAntigo(): void { ... }
```

**Erro 2: Manter dead code "só por precaução"**
```php
// Não aceito — git guarda o histórico; o medo de deletar não tem fundamento
private function migrarParaFormatoLegado(): void { ... } // "pode ser que precisemos depois"
```

**Erro 3: Usar @deprecated em vez de deletar**
```php
// Não aceito — mantém o dead code visível e confuso
/** @deprecated Use notificarCliente() em vez disso */
public function notificarAntigo(Cliente $cliente): void { ... }
```

---

## 6. Benefícios

- **Sinal-para-ruído:** Cada linha restante é relevante — leitores confiam na base de código
- **Velocidade do autoload:** Menos classes significa resolução mais rápida do autoloader
- **Clareza:** Dead code engana — sua ausência remove trilhas falsas
