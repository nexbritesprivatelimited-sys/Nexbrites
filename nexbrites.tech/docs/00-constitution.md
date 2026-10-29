# 00-Constitution & Architecture Decisions (ADR)

## 4. Constitution (Rules That Do Not Change)
1. **Never store card data.** Card, UPI, and net banking details are entered only on the gateway's hosted page or secure SDK. This keeps us out of PCI-DSS scope.
2. **Payment truth comes from the gateway webhook, not the browser.** Access unlocks only after a verified signed webhook or a server-side verification call.
3. **Money is stored as integers in paise** (₹1,000 = 100000). No floats.
4. **All payment operations are idempotent.** A retried webhook never creates a second enrollment.
5. **Secrets live only in environment variables.** Nothing in git, nothing in documents.
6. **Test mode first.** Live keys only after gateway approval.
7. **Every feature has a spec with acceptance criteria** before code.
8. **TypeScript strict mode**, validation on every API route, tests for all pricing and payment logic.

---

## 5. Technology Decisions (ADR Summary)

| ID | Decision | Choice | Reason | Revisit when |
|---|---|---|---|---|
| **ADR-01** | Framework | Next.js (App Router) + TypeScript | Full stack in one repo, strong ecosystem | Never for MVP |
| **ADR-02** | UI | Tailwind CSS + shadcn/ui | Fast, consistent, accessible components | Never for MVP |
| **ADR-03** | Database | PostgreSQL (Docker on the VPS) | Relational integrity for orders and money | Traffic growth: move to managed DB |
| **ADR-04** | ORM | Prisma | Typed schema and migrations | If performance requires raw SQL |
| **ADR-05** | Auth | Auth.js with email/password, Google optional | Standard sessions, OAuth ready | Enterprise SSO needs |
| **ADR-06** | Payments | Razorpay test mode behind a `PaymentProvider` adapter | Supports UPI, cards, net banking, wallets; adapter makes swap cheap | Client confirms another gateway |
| **ADR-07** | Email | Resend via queue | Simple API, queue protects payment flow | Volume growth |
| **ADR-08** | Hosting | Hostinger KVM 2 VPS (2 vCPU, 8 GB RAM, 100 GB NVMe), Docker, Nginx | Full control, public HTTPS URL for webhooks, best learning value | Single point of failure becomes unacceptable |
| **ADR-09** | Testing | Vitest, Playwright, k6, axe | Covers unit, integration, end-to-end, load, accessibility | n/a |

### Additional Architecture Notes:
* **Hosting note:** Next.js needs a Node runtime, so the decision is the KVM 2 VPS (already provisioned).
* **Payment adapter contract:** `createOrder`, `verifyWebhook`, `refund`. Razorpay is the first implementation. Cashfree, PayU, or Adyen would each be one new file.