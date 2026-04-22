# SKILL: Detecting and Refactoring Refused Bequest — Go

## Source
Based on: https://refactoring.guru/smells/refused-bequest

---

## 1. What is it?

A struct embeds another struct (or implements an interface) but does not use most of
the promoted methods — or overrides them with panics. In Go, inheritance is replaced
by embedding and interfaces, so this smell manifests as embedding abuse: a type
"inheriting" behaviour it doesn't actually support.

---

## 2. Warning signs

- [ ] An embedded struct's promoted methods are overridden to `panic` or do nothing
- [ ] Only a small fraction of an embedded type's API is actually used
- [ ] A type implements an interface but several methods return stubs or errors
- [ ] Tests for the embedding type fail when exercising the promoted behaviour
- [ ] The relationship "is-a" is true only conceptually, not behaviourally

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Replace Inheritance with Delegation** | The struct wants some behaviour but not all — use a field instead of embedding |
| **Extract Interface** | Pull only the shared behaviour into a minimal interface; each type implements what it actually supports |
| **Push Down Method** | Move unwanted promoted behaviour into the types that actually use it |

---

## 4. Example

**BEFORE — not accepted:**
```go
type Bird struct {
	name string
}

func (b *Bird) Name() string { return b.name }
func (b *Bird) Fly()         { fmt.Println("flap flap") }
func (b *Bird) Land()        { fmt.Println("touchdown") }

// Penguin embeds Bird but cannot fly
type Penguin struct {
	Bird
}

func (p *Penguin) Fly() {
	panic("penguins cannot fly") // refused bequest
}

// Caller breaks at runtime
var b Bird = Penguin{} // type system can't catch this
b.Fly()               // panics
```

**AFTER — expected:**
```go
type Bird interface {
	Name() string
}

type Flyer interface {
	Bird
	Fly()
	Land()
}

type Sparrow struct{ name string }

func (s *Sparrow) Name() string { return s.name }
func (s *Sparrow) Fly()        { fmt.Println("flap flap") }
func (s *Sparrow) Land()       { fmt.Println("touchdown") }

type Penguin struct{ name string }

func (p *Penguin) Name() string { return p.name }
func (p *Penguin) Swim()       { fmt.Println("splash") }
```

**Why this pattern:**
- `Bird` contains only behaviour shared by all birds
- `Flyer` is an opt-in interface — penguins simply don't implement it
- Code that needs flying holds a `Flyer`, not just a `Bird`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping the panic and documenting it**
```go
// Not accepted — LSP is still violated; callers cannot trust the contract
func (p *Penguin) Fly() {
	// documented but still breaks callers
	panic("penguins cannot fly")
}
```

**Mistake 2: Returning a no-op instead of restructuring**
```go
// Not accepted — silent failures hide bugs
func (p *Penguin) Fly() {} // does nothing silently
```

---

## 6. Benefits

- **Interface safety:** Every type honours the full contract it advertises
- **Cleaner embedding:** Embedding is used for genuine code reuse, not fake inheritance
- **LSP compliance:** Code iterating a `[]Bird` won't panic unexpectedly
