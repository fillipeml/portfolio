# Case studies

Write-ups of systems I built that are not going to be open-sourced, and why each one could
not be.

Most of my work lives in repositories you can clone and run — they are listed at the bottom
of this page. What is here is the remainder: three systems where publishing the code was the
wrong answer. One belongs to the firm that commissioned it. One is a live product of my own.
One cannot be separated from the records of the several hundred people it processes.

In two of the three, the confidential part is not the source at all. It is the configuration —
a firm's own operating procedure, modelled as data, which is the thing that made the system
worth building — and the documentation, which carries far more identifiers than any source
file does. Replacing all of that with fiction is possible, and what survives is a generic
schema that demonstrates nothing about the part that was interesting.

So a write-up was the honest artefact rather than the consolation prize. A reader deciding
whether I can design a system gets more from four pages about the decisions than from a
directory listing they will not open.

So each case study here is organised around the decisions: what the constraint was, what I
chose, what it cost, and what I would do differently now. The seven core sections are the same
in each one, each write-up adds what its subject needs, and the last section always answers the
question the repository's absence raises.

## The case studies

**[A bespoke CRM built as a multi-tenant product with one tenant](case-studies/law-firm-crm.md)**
A forty-lawyer firm's commercial process modelled as data, with tenancy enforced by the
database rather than by the application, and an action engine that documents its own new
failure mode. 44,600 lines; 29 of 30 tables under forced row-level security; eight critical
findings from an adversarial review of my own work, all closed. Not a repository because the
configuration *is* the confidential part — the seed files encode a real firm's operating
procedure — and the documentation carries more identifiers than the code does.

**[A multi-tenant SaaS over a certificate-only government API](case-studies/vehicle-stock-saas.md)**
My own product, in production: a usable surface over a mandatory federal vehicle-stock
register whose only credential is a digital certificate and whose documentation is not
published. Six brute-force probe scripts, profile detection by asking the government what a
certificate is allowed to be, and no local copy of the data. Not a repository because it is a
live product, and because the working tree holds real buyers' data in dead code.

**[A bulk legal-notice pipeline that refused to send 116 of 284 letters](case-studies/bulk-legal-notice-pipeline.md)**
Extrajudicial debt notices for an education company's defaulted students, where the headline
result is the refusals: every letter whose stated contract arithmetic did not reconcile was
blocked rather than signed. Idempotency keyed so that nobody can receive two parallel
deadlines, and a consumer-protection rule enforced by a test. Not a repository because the
records of 284 identified people are inseparable from it — the generated filenames alone
encode their tax IDs.

## What was extracted instead

Where a system contained something genuinely reusable, I pulled that part out, rewrote it
against synthetic data, and published it on its own. Those are real repositories with tests
and CI, and they are the closest thing to the code behind two of the case studies here:

- [whatsapp-cloud-api-client](https://github.com/fillipeml/whatsapp-cloud-api-client) — the
  Groups API client from the CRM, as a standalone library.
- [mtls-pkcs12-agent](https://github.com/fillipeml/mtls-pkcs12-agent) — the mutual-TLS
  certificate handling from the vehicle-stock SaaS, as a standalone library.
- [postgres-rls-multitenant-starter](https://github.com/fillipeml/postgres-rls-multitenant-starter)
  — the tenancy model from the CRM, with the four guarantees proven and the four ways to get it
  wrong reproduced.

## The rest of the work

Systems I built that are published in full, each running offline on fictional data:

| Repository | What it does |
| --- | --- |
| [legal-intake-triage](https://github.com/fillipeml/legal-intake-triage) | The front door of a law firm's practice areas: e-mail and form intake, a model-prepared decision package, a human approval that creates the task and the reply |
| [judgment-summary-pipeline](https://github.com/fillipeml/judgment-summary-pipeline) | A backlog of court decisions turned into one house-style summary per case, behind two gates that can only refuse |
| [court-debt-calculator](https://github.com/fillipeml/court-debt-calculator) | A deterministic engine for updating court-ordered debts, validated to the cent against public court calculators |
| [settlement-reminder-pipeline](https://github.com/fillipeml/settlement-reminder-pipeline) | Daily reminders and collection for court settlements, settled from receipts read by a model and matched by amount |
| [case-diagnostics](https://github.com/fillipeml/case-diagnostics) | Strategic diagnosis of a case file, where deterministic rules verify every thesis the model proposes |
| [tax-settlement-analytics](https://github.com/fillipeml/tax-settlement-analytics) | Analytics over 1,134 public tax-settlement terms, plus a simulator that checks a proposal against what the treasury has accepted |
| [court-deadline-triage](https://github.com/fillipeml/court-deadline-triage) | Daily triage of court gazette publications, with the due date computed deterministically over versioned court calendars |
| [court-notice-monitor](https://github.com/fillipeml/court-notice-monitor) | A read-only sweep of the electronic judicial domicile, built so that it cannot acknowledge service |
| [eu-job-pipeline](https://github.com/fillipeml/eu-job-pipeline) | Multi-source job ingestion with rule-based and model-based fit scoring, and a golden-set evaluation |
| [legal-llm-evals](https://github.com/fillipeml/legal-llm-evals) | An evaluation harness that refuses to print a rate without its confidence interval, or a model grader's score without that grader's calibration against human labels |

## On names and numbers

No client, employer, counterparty or individual is named anywhere in this repository, and no
identifier appears that could be used to find one. Systems are described by what they do and
who they serve — "a law firm's controllership team", "a vehicle dealer" — which is the whole
reason these write-ups can exist at all.

Every number here was measured rather than estimated. Where I do not have a figure, there is
no figure, rather than a round one.

## Licence

[CC BY 4.0](LICENSE) — these are written works, so they carry a content licence. The code
repositories linked above are MIT.
