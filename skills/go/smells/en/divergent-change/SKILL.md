# SKILL: Detecting and Refactoring Divergent Change — Go

## Source
Based on: https://refactoring.guru/smells/divergent-change

---

## 1. What is Divergent Change

A single struct or package that must be modified for many different, unrelated reasons. Every time a new feature is added — payment logic, report format, third-party integration — the same file is touched. The type has absorbed multiple responsibilities.

**Why this happens:**
- A "service" type grew to coordinate everything without delegating
- Responsibilities were never separated because the system started small
- Fear of creating too many files led to stuffing unrelated logic into one place

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] The same struct is modified when database schema changes, when API responses change, and when business rules change
- [ ] A file's git log shows commits from payment, reporting, and integration teams
- [ ] You cannot describe what the type does without using "and" multiple times
- [ ] The type imports packages from several unrelated domains (database, HTTP, email, PDF)
- [ ] Methods on the struct fall into clearly distinct groups with no shared state

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Responsibilities can be separated | Extract Type into focused structs |
| Shared package mixes multiple domains | Split into sub-packages |
| A group of methods shares no state with the rest | Extract and move to a new package |
| Orchestration logic mixed with domain logic | Separate orchestrator from domain object |

---

## 4. Example

**BEFORE — not accepted:**
```go
package app

// OrderManager changes when: DB schema changes, email template changes,
// report format changes, AND payment provider changes.
type OrderManager struct {
	db      *sql.DB
	mailer  *smtp.Client
	payment PaymentGateway
}

func (m *OrderManager) Save(o *Order) error            { /* DB logic */ return nil }
func (m *OrderManager) SendConfirmation(o *Order) error { /* email template */ return nil }
func (m *OrderManager) GenerateInvoicePDF(o *Order) []byte { /* PDF logic */ return nil }
func (m *OrderManager) ChargeCard(o *Order) error       { /* payment logic */ return nil }
func (m *OrderManager) ExportCSV(orders []Order) []byte { /* report logic */ return nil }
```

**AFTER — expected:**
```go
package order

type Repository struct{ db *sql.DB }
func (r *Repository) Save(o *Order) error { return nil }

package notification

type Service struct{ mailer *smtp.Client }
func (s *Service) SendConfirmation(o *order.Order) error { return nil }

package invoice

type Generator struct{}
func (g *Generator) GeneratePDF(o *order.Order) []byte { return nil }

package payment

type Processor struct{ gateway PaymentGateway }
func (p *Processor) Charge(o *order.Order) error { return nil }

package report

type Exporter struct{}
func (e *Exporter) ExportCSV(orders []order.Order) []byte { return nil }
```

**Why this pattern:**
- Each type changes for exactly one reason
- Teams can work on their respective packages without merge conflicts on a shared file
- Packages can be tested and deployed independently

---

## 5. Negative examples — what NOT to do

**Mistake 1: Splitting into files but keeping one package**
```go
// Not accepted — same package still changes for all reasons; files are cosmetic
// order_db.go, order_email.go, order_payment.go — all in package app
```

**Mistake 2: Creating a god orchestrator that pulls all extracted types back together**
```go
// Not accepted — the orchestrator becomes the new Divergent Change smell
type AppService struct {
	repo    *order.Repository
	notif   *notification.Service
	invoice *invoice.Generator
	payment *payment.Processor
	report  *report.Exporter
}
func (s *AppService) ProcessOrder(o *order.Order) error { /* all steps */ return nil }
```

---

## 6. Benefits

- **Single Responsibility:** Each type has one reason to change
- **Team independence:** Separate packages reduce cross-team merge conflicts
- **Testability:** Focused types are easier to unit-test in isolation
