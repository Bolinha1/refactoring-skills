# SKILL: Detectando e Refatorando Middle Man — Go

## Fonte
Baseado em: https://refactoring.guru/pt-br/smells/middle-man

---

## 1. O que é Middle Man

Uma struct ou função que existe apenas para delegar chamadas a outra struct sem
adicionar nenhum valor. Se a maior parte dos métodos de uma struct apenas repassam
chamadas para outra, ela é um intermediário desnecessário.

**Por que isso acontece:**
- Resultado excessivo de Hide Delegate: a delegação foi aplicada além do necessário
- Uma abstração foi criada antecipadamente e nunca ganhou comportamento próprio
- Refatorações anteriores deixaram uma camada vazia após mover a lógica real

---

## 2. Sinais de alerta (gatilhos para esta SKILL)

Ative esta SKILL quando identificar qualquer item abaixo:

- [ ] Mais da metade dos métodos de uma struct apenas delega para outra struct
- [ ] A struct não adiciona nenhuma lógica própria — é pura delegação
- [ ] Remover a struct e chamar o objeto subjacente diretamente não mudaria o comportamento
- [ ] A struct existe apenas como "wrapper" sem transformar dados ou encapsular regras

---

## 3. Técnicas de tratamento (em ordem de preferência)

| Situação encontrada | Técnica indicada |
|---|---|
| Struct que só delega | Remove Middle Man |
| Delegação tem algum valor (logging, auth) | Manter — não é smell nesse caso |
| Struct criada por Hide Delegate excessivo | Desfazer parcialmente o Hide Delegate |

---

## 4. Exemplo

**ANTES — não aceito:**
```go
type GerenciadorPessoa struct {
	pessoa *Pessoa
}

// Todos os métodos apenas delegam — não há lógica própria
func (g *GerenciadorPessoa) GetNome() string       { return g.pessoa.GetNome() }
func (g *GerenciadorPessoa) GetDepartamento() string { return g.pessoa.GetDepartamento() }
func (g *GerenciadorPessoa) GetDataAdmissao() time.Time { return g.pessoa.GetDataAdmissao() }
```

**DEPOIS — esperado:**
```go
// Usar Pessoa diretamente
type Pessoa struct {
	Nome        string
	Departamento string
	DataAdmissao time.Time
}

// Chamador acessa Pessoa diretamente
nome := pessoa.Nome
dept := pessoa.Departamento
```

**Por que esse padrão:**
- `GerenciadorPessoa` não adiciona nenhum valor — é ruído puro
- Remover o intermediário simplifica o código sem perder funcionalidade

---

## 5. Exemplos negativos — o que NÃO fazer

**Erro 1: Remover um intermediário que tem valor real**
```go
// Não aceito — se o intermediário faz logging, auth ou cache, ele tem valor
type ServicoPessoa struct {
	repo *RepositorioPessoa
}

func (s *ServicoPessoa) BuscarPorID(id int) (*Pessoa, error) {
	s.logger.Info("buscando pessoa", "id", id) // valor real: logging
	return s.repo.BuscarPorID(id)
}
```

**Erro 2: Eliminar todas as camadas de serviço por serem "delegação"**
```go
// Não aceito — camadas de serviço geralmente têm propósito;
// verifique antes de remover se não há lógica futura ou testes dependendo dela
```

---

## 6. Benefícios

- **Simplicidade:** Menos camadas para navegar ao ler o código
- **Manutenção:** Menos arquivos para atualizar quando a implementação muda
- **Clareza:** O código chamador vê diretamente com quem está interagindo
