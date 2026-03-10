# Flip Finder — Product Roadmap & Feature Ideas

## Current State
Single-page browser tool that scrapes Finca Raíz and Ciencuadras listings, displays them in a clean UI, and lets you mark them as contactado/descartado. Basic filtering by price, area, rooms, estrato.

---

## Phase 1: Fix the Foundation (Quick Wins)

### 1. Deal Calculator per Listing
Right now you see price and $/m2 but you can't evaluate the *flip potential*. Add fields per listing:
- **Estimated renovation cost** (manual input or per-m2 estimate)
- **Estimated resale price** (based on comparable $/m2 in the same barrio + estrato)
- **Projected ROI** = (Resale - Purchase - Renovation) / (Purchase + Renovation)
- **Traffic light badge**: green (>25% ROI), yellow (15-25%), red (<15%)

This turns Flip Finder from a *search tool* into a *decision tool*.

### 2. Barrio Reference Data
A sidebar/overlay with neighborhood intel:
- Average $/m2 by estrato for key barrios (Chapinero, Suba, Kennedy, Bosa, etc.)
- Valorización trend (up/down/flat) — even if manually maintained
- Safety score, walkability, metro/TransMilenio proximity
- This helps you instantly spot underpriced listings

### 3. Notes & Photos per Listing
After visiting a property, you need to capture:
- Free-text notes ("needs full kitchen reno, good bones, noisy street")
- Upload photos from your visit
- Estimated renovation checklist (kitchen, bathrooms, floors, painting, etc.)
- Contact info of the seller/agent

### 4. Comparison View
Select 2-3 listings and see them side by side:
- Price, area, $/m2, estrato, barrio, ROI estimate
- Makes the final decision much easier

---

## Phase 2: Pipeline Management (The Flip Workflow)

### 5. Kanban Pipeline Board
Move listings through stages instead of just contactado/descartado:
- **Prospecto** → **Contactado** → **Visita Programada** → **Oferta Enviada** → **En Negociación** → **Comprado** → **En Renovación** → **En Venta** → **Vendido**
- Each stage shows count and total investment
- Drag and drop between columns

### 6. Financial Dashboard
Once you move a listing to "Comprado":
- Track purchase price, notaría costs, renovation budget
- Log actual expenses by category (plomería, electricidad, cocina, pisos, pintura)
- Budget vs. actual tracking with burn-down chart
- Calculate real ROI once sold

### 7. Timeline & Reminders
- Set follow-up dates ("call agent Tuesday", "visit Saturday 10am")
- Track days-on-market for your active flips
- Renovation timeline with milestones

---

## Phase 3: Intelligence & Automation

### 8. Price Alert / Auto-Search
- Run searches on a schedule (daily/weekly)
- Highlight NEW listings since last search (first-mover advantage is everything)
- Push notification or email when a listing matches your exact criteria
- "Hot deal" flag when $/m2 is significantly below barrio average

### 9. Comparable Sales Analysis
- For any listing, auto-find similar properties in the same barrio
- Show what renovated apartments in that zone sell for
- Calculate the "renovation premium" — how much value does a flip add in that neighborhood
- This is the #1 thing experienced flippers do manually

### 10. Renovation Cost Estimator
- Templates by apartment size and condition:
  - "Cosmetic refresh" (paint + floors): ~$800K-1.2M/m2
  - "Kitchen & bath reno": ~$1.5M-2.5M/m2
  - "Full gut renovation": ~$3M-5M/m2
- Adjust by estrato (higher estrato = higher finish expectations = higher cost)
- Generate a preliminary budget before even visiting

### 11. Contractor Directory
- Save your trusted contractors with specialties and rates
- Assign contractors to specific renovation tasks
- Track contractor availability and performance

---

## Phase 4: Sell-Side Tools

### 12. Listing Creator
Once renovation is done, generate:
- Professional listing description (AI-assisted, in Spanish)
- Photo gallery with before/after
- Floor plan annotation
- Export ready for Finca Raíz / Metrocuadrado / Ciencuadras posting

### 13. Staging & Marketing Checklist
- Pre-listing checklist (professional photos, deep clean, staging)
- Marketing channel tracker (which sites, social media, WhatsApp groups)
- Lead tracking for potential buyers

### 14. Portfolio Analytics
- Total flips completed, average ROI, average days to flip
- Best-performing barrios and estratos
- Capital deployed vs. returned over time
- YTD profit/loss

---

## Phase 5: Scale & Team

### 15. Multi-User Support
- Share pipeline with a partner or team
- Role-based access (viewer, editor, admin)
- Activity log ("Juan moved Apt Chapinero to En Renovación")

### 16. Document Vault
- Store promesas de compraventa, escrituras, renovation contracts
- Link documents to specific properties
- Checklist of required documents per stage

### 17. Financing Tracker
- Track loan terms, interest rates, disbursement schedules
- Calculate carrying costs (interest + admin while renovating)
- Factor financing costs into ROI calculation

---

## Prioritization Matrix

| Feature | Impact | Effort | Priority |
|---------|--------|--------|----------|
| Deal Calculator | 🔥 High | Low | **P0 — Do Now** |
| Kanban Pipeline | 🔥 High | Medium | **P0 — Do Now** |
| Notes & Photos | 🔥 High | Low | **P1 — Next** |
| Barrio Reference | High | Medium | **P1 — Next** |
| Comparison View | Medium | Low | **P1 — Next** |
| Financial Dashboard | 🔥 High | Medium | **P2 — Soon** |
| Timeline & Reminders | Medium | Low | **P2 — Soon** |
| Price Alerts | High | High | **P2 — Soon** |
| Renovation Estimator | High | Medium | **P3 — Later** |
| Comparable Sales | 🔥 High | High | **P3 — Later** |
| Listing Creator | Medium | Medium | **P3 — Later** |
| Portfolio Analytics | Medium | Medium | **P3 — Later** |
| Contractor Directory | Low | Low | **P4 — Backlog** |
| Document Vault | Medium | Medium | **P4 — Backlog** |
| Multi-User | Low | High | **P4 — Backlog** |
| Financing Tracker | Medium | Medium | **P4 — Backlog** |

---

## My Top 3 Recommendations

**If I were building this product, I'd do these first:**

1. **Deal Calculator** — This is the single highest-value feature. Right now Flip Finder helps you *find* apartments. With a deal calculator it helps you *evaluate* them. That's the difference between a search tool and a business tool.

2. **Kanban Pipeline** — The flip workflow is inherently stage-based. The current 3-state system (nuevo/contactado/descartado) doesn't capture "I visited and I'm thinking about it" or "I made an offer." A Kanban board turns Flip Finder into your operating system.

3. **Notes & Photos** — Every experienced flipper keeps notes. If you visit 5 apartments on Saturday, by Monday you can't remember which one had the water damage and which one had the great balcony. Notes tied to listings solve this.

These three features together transform Flip Finder from "a scraper with a nice UI" into "the tool I can't run my flipping business without."
